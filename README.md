# Load-Balancer-Solution-With-Apache

## **LOAD BALANCER**

In this project https://github.com/Umeh-Johnbosco-Ifeanyi/Devops_Tooling_website_Solution, i developed a tooling solution for a Devops team comprising of 3 Webservers, 1 Database(mysql) and an NFS server availble to the web servers. In this project, we will be working on unifying the requests to the webservers whereby clients will be able to access all webservers using a single URL. This will be possible by using a load baalancer(Apache in this case) to distribute traffic to the webservers.

![pere](https://user-images.githubusercontent.com/77943759/225074729-ab32bf65-114f-4038-84ca-3772e09a8644.png)

The requirement for this project are:

> Two RHEL8 Web Servers
> One MySQL DB Server (based on Ubuntu 20.04)
> One RHEL8 NFS server

![perequisite](https://user-images.githubusercontent.com/77943759/225075317-454b4cc1-34b9-43cc-8152-ca79c66a4a18.png)


## **CONFIGURE APACHE AS A LOAD BALANCER**

Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb:

![instance](https://user-images.githubusercontent.com/77943759/225169575-fe254675-bc08-419a-9a16-32f5a3850f6b.png)

Open inbound rule port 80 for the instance so it would be accessible to traffic on this port

Install Apache:

```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev
```

![aptupdate](https://user-images.githubusercontent.com/77943759/225172084-1a41afee-afd7-481f-a8ba-fc34fb4a8f59.png)

![libxml2](https://user-images.githubusercontent.com/77943759/225172195-683ec3c7-2b3a-4318-b180-52fae7b2b751.png)



Enable these modules:

```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```
![aemod](https://user-images.githubusercontent.com/77943759/225172308-c84de5eb-8215-4a55-b45c-8a1751586fae.png)


#Restart apache2 service
`sudo systemctl restart apache2`

Confirm apache2 is up and running

`sudo systemctl status apache2`

![confirmapache](https://user-images.githubusercontent.com/77943759/225172536-0390c441-871c-49fc-9aae-c7f882b892b1.png)


Configure load balancing. Open the following file and populate it with the code below

`sudo vi /etc/apache2/sites-available/000-default.conf`

```
#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```

#Restart apache server

`sudo systemctl restart apache2`

![editsitesavail](https://user-images.githubusercontent.com/77943759/225171070-b2b4ef3e-2ce7-4b8b-b4df-7b77a68ecc59.png)

This is bytraffic balancing method, it distribute incoming load between our Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

Verify that our configuration works. Open web browser and enter:

`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

![indexphplb](https://user-images.githubusercontent.com/77943759/225171966-12d4ba5e-8bdb-42ce-b3bd-1d5c7e9a0254.png)

Open two terminals for the two webservers and Run

`sudo tail -f /var/log/httpd/access_log`

![s2accesslog](https://user-images.githubusercontent.com/77943759/225215660-640b1605-1478-48d3-884a-708d30802f78.png)

![s1accesslog](https://user-images.githubusercontent.com/77943759/225215724-8a2735ad-4f1e-4626-9015-56fbf8077649.png)

These are the logs of our load balancer page (apache), showing the get requests it is senging to our two webserver

Refresh the load balancer page and check the access log on both webserver terminals

![xtralog2](https://user-images.githubusercontent.com/77943759/225215514-575d44d8-5a0f-4bf2-a493-413c29b1be45.png)

![xtralog1](https://user-images.githubusercontent.com/77943759/225215591-a33db8e5-fcde-407f-b6a6-ff5200bc8df3.png)

The loadfactor we configured were set to approximately equal numbers and this has made the requests coming in approximately the same


## **SET UP DNS**

Open and edit load balancer server

`sudo vi /etc/hosts`

Add 2 records into this file with Local IP address and arbitrary name for both of our  Web Servers

```
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```
![etchosts](https://user-images.githubusercontent.com/77943759/225219520-1795e0c6-9dcb-46f2-aaa2-fd726111609c.png)


Update the LB config file with these names instead of IP addresses.

`sudo vi /etc/apache2/sites-available/000-default.conf`

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
![reeditsitesavail](https://user-images.githubusercontent.com/77943759/225219366-beb1051b-172f-4c8f-9495-5cf73d8c2c84.png)


Now, curl the the webservers from the Loadbalancer

```
curl http://Web2
curl http://Web1
```
![curl](https://user-images.githubusercontent.com/77943759/225219079-56b8c92b-ed1f-47a0-a4db-4f53bb51cf6a.png)

Note that this is local to the load balancer and cannot be accessible from either webservers

The diagram below shows exactly how the setup is

![Screenshot from 2023-03-15 06-23-00](https://user-images.githubusercontent.com/77943759/225219787-2d9cfe51-ff16-4598-b1be-53070b72c271.png)


End






