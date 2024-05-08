# What is Web Stack Technology?
Web stack technology, often referred to as a technology stack, is a collection of frameworks and tools specifically chosen to work together in developing a software product. These components are carefully selected to ensure seamless integration and efficient functioning of the software.

Web Stack Examples:

- **LAMP**: Consists of Linux (operating system), Apache (web server), MySQL (database management system), and PHP, Python, or Perl (programming languages).

- **LEMP**: Includes Linux, Nginx (web server), MySQL, and PHP, Python, or Perl.

- **MERN**: Comprises MongoDB (database), ExpressJS (web framework), ReactJS (frontend library), and NodeJS (runtime environment).

- **MEAN**: Features MongoDB, ExpressJS, AngularJS (frontend framework), and NodeJS.

These examples represent common configurations used in web development, each tailored to suit different project requirements and preferences.


## Description
For my LAMP stack assignment, I have implemented a web application using the Linux operating system, Apache web server, MySQL database, and PHP programming language. The purpose of the assignment was to demonstrate my understanding of setting up and configuring a LAMP stack environment to host a dynamic web application.

## Instructions
To run the LAMP stack project locally on an AWS instance, follow these steps:

## Log into your AWS Account & Create an Ubuntu EC2 instance

1. Log in to the AWS Management Console using your AWS account credentials.

2. Navigate to the EC2 dashboard.

3. Click on the "Launch Instance" button to start creating a new instance.

4. Choose a unique name for your instance.

<img width="575" alt="Screenshot 2024-05-06 193332" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d5d35118-0f7d-497f-acf3-054f6b974d27">

5. In the "Choose an Amazon Machine Image (AMI)" section, select "Ubuntu Server 24.04 LTS (HVM), SSD Volume Type" as your instance image.

<img width="539" alt="Ububtu SSD" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c6668dba-3551-4c35-9a20-62cdcca7177e">

6. Ensure that the selected AMI is marked as "free tier eligible."

7. Proceed with the configuration of instance details, such as instance type, network settings, storage, security groups, and key pair.

<img width="548" alt="KeyValue" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/efa9a714-ea29-4623-b8ab-9493e956fa0b">

8. Review your configuration and click on "Launch" to create the Ubuntu EC2 instance.

<img width="763" alt="Running Instance" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b2de5809-222b-416e-97a5-e2703544760f">

## Connect to the Instance via SSH

To connect to the AWS EC2 instance I used MobaXterm with the following steps:

1. **Open MobaXterm**: Launch MobaXterm on your local machine.

2. **Open SSH Session**: In the MobaXterm interface, click on the "Session" button located in the upper left corner.

3. **Configure SSH Connection**: In the session settings window, configure the SSH connection settings:
   - Set the remote host (Public IP or DNS name of your EC2 instance).
   - Specify the username (typically "ubuntu" for Ubuntu instances).
   - Choose the SSH port (default is 22).
   - Select the appropriate SSH key (private key associated with the key pair used during instance creation).

4. **Connect to Instance**: Click on the "OK" button to initiate the SSH connection to your EC2 instance.

 <img width="958" alt="SSH into AWS" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8f2955a0-ade9-41e4-8f52-73417bff8a5c">


### Installing Apache2

Install Apache2:

```
$ *sudo apt install apache2*
```

Confirming that Apache2 is installed.
Verify that Apache was successfully installed and running on our instance. Run the following command: 
```
$ *sudo systemctl status apache2*
```
![Aspose Words 5d56049b-1dab-4f29-b5bd-c8099876c392 001](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/72c9b77f-dd40-4b0c-8847-157d87119e79)

Notes:
Ensure that port 80 is open in the AWS security group to allow HTTP traffic to the instance.

The server is running and can be accessed locally in the ubuntu shell by running the command below:
```
$curl http://localhost:80
```
OR
```
$curl http://127.0.0.1:80
```

<img width="632" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5c95417f-1266-4422-a365-f87442ffd822">

Test via browser

Copy the public IPv4 address and paste it into your browser

<img width="531" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a4376833-25f2-4083-a7b8-40d01cf3a68f">

The result show be the default Apache2 web page being shown 

<img width="392" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/23d722bc-a213-4e02-a084-2c7e0c5ee456">


**Installing MySQL**

MySQL, an open-source relational database management system, enables high availability deployments on AWS instances, catering to both cloud-based and on-premise environments.
```
$ *sudo apt install mysql-server*
```
*Logging into mysql*
```
$ *sudo mysql*
```

![Aspose Words 5d56049b-1dab-4f29-b5bd-c8099876c392 002](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/24b4a9f0-958f-4b5f-a024-111d3d20f857)

Exit the MySQL shell, run the following command:
```
$ *mysql> exit*
```

The security script comes pre-installed with mysql. This script removes some insecure settings and lock down access to the database system.

```
$ *sudo mysql_secure_installation*
```

![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/4acdeae1-026c-4b74-aacb-d1663b33e6e2)


**Installing PHP**

1. php package
2. php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases.
3. libapache2-mod-php, to enable Apache to handle PHP files.

```
$ sudo apt install php libapache2-mod-php php-mysql
```
![Aspose Words 5d56049b-1dab-4f29-b5bd-c8099876c392 003](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/874b3d3e-0bbd-4cbf-8f6d-b9753a4bc6aa)

**Create a Virtual Host for Your Website Using Apache**
create a directory for “projectlamp
```
$ *sudo mkdir /var/www/projectlamp*
```

creating a new configuration file that will reside in the Apache directory using vi/vim
Run the following command:

```
$ *sudo vi /etc/apache2/sites-available/projectlamp.conf*
```

This will result in a blank text file.

To interact with the text editor, first hit the “i” key to be in insert mode

```
*<VirtualHost *:80>*  
    *ServerName* projectlamp  
    *ServerAlias* www.projectlamp   
    *ServerAdmin* webmaster@localhost  
    *DocumentRoot* /var/www/projectlamp  
    *ErrorLog* ${APACHE_LOG_DIR}/error.log  
    *CustomLog* ${APACHE_LOG_DIR}/access.log combined  
*</VirtualHost>*
```

**Enable virtual host**
```
$ *sudo a2ensite projectlamp*
```

**Disable default Apache website**
```
$ *sudo a2dissite 000-default*
```

 **Check configuration file for errors**
 ```
$ *sudo apache2ctl configtest*
```

**Reload Apache server to apply changes**
```
$ *sudo systemctl reload apache2*
```


create a custom index.html

![Aspose Words 5d56049b-1dab-4f29-b5bd-c8099876c392 004](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f29ebe77-dc36-4e51-8c93-e4e43b9170e1)

![Aspose Words 5d56049b-1dab-4f29-b5bd-c8099876c392 005](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/07acbea7-4512-4361-8084-7f05aed19bcd)

**Enabling PHP on the website**

By default, Apache prioritizes the `index.html` file over an `index.php` file. To ensure that `index.php` serves as the landing page for our website, we need to adjust the DirectoryIndex settings in Apache. The command below allows us to modify the file behavior:

```
$ *sudo vim /etc/apache2/mods-enabled/dir.conf*
```
  *DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm*

Reload Apache
Apache is reloaded so the changes takes effect.
```
$ *sudo systemctl reload apache2*
```
Create a new file named index.php
```
$ *vim /var/www/projectlamp/index.php*
```

After Reloading the page 

![Aspose Words 5d56049b-1dab-4f29-b5bd-c8099876c392 006](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f3b683b2-94a7-4e85-ac91-0686f43ee1b8)
