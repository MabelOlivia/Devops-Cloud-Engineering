# Load Balancer Solution With Apache

A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

**Architecture**

<img width="503" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/787d1af9-7f78-424b-9cb6-aa30d8384916">

## Task

Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.
﻿

**Prerequisites**

Ensure that the following servers are installedd and configure already.

- Two RHEL9 Web Servers
- One MySQL DB Server (based on Ubuntu 24.04)
- One RHEL9 NFS Server
  
**Prerequisites Configurations**

- Apache (httpd) is up and running on both Web Servers.
- /var/www directories of both Web Servers are mounted to /mnt/apps of the NFS Server.
- All neccessary TCP/UDP ports are opened on Web, DB and NFS Servers.
- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the Tooling Website (e.g, http://<Public-IP-Address-or-Public-DNS-Name>/index.php)
  
  <img width="789" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7ebff0bc-459f-4e59-8fbd-bba6f587111c">

  <img width="789" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b54cdac6-868b-449a-9454-2853190cc716">


  
## Step 1 - Configure Apache As A Load Balancer

1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb

   <img width="731" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7a23860b-fabd-489e-8d66-5a65f3c16ce9">
   
2. Open TCP port 80 on Project-8-apache-lb by creating an Inbounb Rule in Security Group

3. Install Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.
     
- Update and upgrade Ubuntu
```
sudo apt update && sudo apt upgrade
```

- Install Apache
```
sudo apt install apache2 -y
```
```
sudo apt-get install libxml2-dev
```

- Enable the following modules
```
sudo a2enmod rewrite
sudo a2enmod  proxy
sudo a2enmod  proxy_balancer
sudo a2enmod  proxy_http
sudo a2enmod  headers
sudo a2enmod  lbmethod_bytraffic
```

- Restart Apache2 Service

```  
sudo systemctl restart apache2
sudo systemctl status apache2
```
  <img width="626" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/523687fc-2bfc-4602-821c-fc9b0922430a">

**Configure Load Balancing**

i. Open the file 000-default.conf in sites-available

```
sudo vi /etc/apache2/sites-available/000-default.conf
```

ii. Add this configuration into the section <VirtualHost *:80>  </VirtualHost>

```
<Proxy “balancer://mycluster”>
           BalancerMember http://172.31.36.192:80 loadfactor=5 timeout=1
           BalancerMember http://172.31.35.222:80 loadfactor=5 timeout=1
           ProxySet lbmethod=bytraffic
           # ProxySet lbmethod=byrequests
</Proxy>


ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```
<img width="700" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0c69308f-5c68-425c-a406-d54241babcee">


iii. Restart Apache
```
sudo systemctl restart apache2
```

<img width="607" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/1d0d7710-d687-439a-880a-2982c0583c37">

---

**Bytraffic Balancing Method**

The `bytraffic` balancing method distributes incoming load between the web servers according to the current traffic load. The proportion in which traffic is distributed can be controlled by the `loadfactor` parameter.

Other methods that can also be adopted include:

- **bybusyness**: Balances traffic based on the number of active requests on each server.
- **byrequests**: Distributes requests equally among servers regardless of traffic load.
- **heartbeat**: Uses an external heartbeat mechanism to determine which servers should receive traffic.

---

4. Verify that the configuration works
   
*Access the website using the LB's Public IP address or the Public DNS name from a browser*

<img width="871" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7354433a-348e-4324-a9bb-9275108a7fb7">

**Note:** If in the previous project, /var/log/httpd was mounted from the Web Server to the NFS Server, unmount them and ensure that each Web Servers has its own log directory.

*Unmount the NFS directory*

Check if the Web Server's log directory is mounted to NSF
```
df -h
sudo umount -f /var/log/httpd
```

If the directory is busy, the services using it needs to be stopped first.
```
sudo systemctl stop httpd
```
Check that the directory is unmounted
```
df -h
```

*Open two ssh consoles for both Web Server and run the command:*

```
sudo tail -f /var/log/httpd/access_log
```

<img width="762" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/38ebafc8-e052-4ff5-87f1-22d73ed8a8fc">

<img width="761" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dc1638da-0858-41dd-bcfb-c920e88e9ec7">

*Refresh the browser page several times and ensure both Web Servers receive HTTP and GET requests. New records must apear in each web server log files. The number of request to each servers will be approximately the same since loadfactor is set to the same value for both servers. This means that traffic will be evenly distributed between them.*

<img width="764" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/51dba86c-dd79-4b2c-bf02-442fb6a4c5cb">

<img width="764" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/6f7fa145-d8d6-467f-9259-148d2ce6f9cf">

### Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if there are lots of servers to manage. It is best to configure local domain name resolution. The easiest way is use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well.

Let us Configure the IP address to domain name mapping for our Load Balancer.

- Open the hosts file

```
sudo vi /etc/hosts
```

Add two records into file with Local IP address and arbitrary name for the Web Servers

<img width="395" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7cab74e0-ff2e-4bba-8aeb-9402aa2d144b">

- Update the LB config file with those arbitrary names instead of IP addresses

```
sudo vi /etc/apache2/sites-available/000-default.conf
```
```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

<img width="577" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/98ad1a17-b762-48d8-a7d9-e4dd665021d8">

Let us try to curl the Web Servers from LB locally

```
curl http://Web1
curl http://Web2
```
<img width="608" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/accf60df-4e46-4b61-86bf-a67dd217aa9f">

**Note**, This is only internal configuration and also local to the LB server, these names will neither be 'resolvable' from other servers internally nor from the Internet.

And Finally 🎉
We have implemented a Load Balancing Web Solution










