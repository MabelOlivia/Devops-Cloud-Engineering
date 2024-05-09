# LEMP STACK

The LEMP stack, which stands for Linux, Nginx, MySQL, and PHP, is a popular software stack used for hosting dynamic websites and web applications. Here's a summary of each component:

1. **Linux**: The operating system (OS) layer of the stack. Linux provides the foundation for running the other components and managing system resources. It is a stable and secure choice for server environments.

2. **Nginx**: The web server layer of the stack. Nginx is known for its high performance, scalability, and efficient handling of concurrent connections. It is commonly used as a reverse proxy and load balancer, serving static content quickly while also handling dynamic requests.

3. **MySQL**: The database layer of the stack. MySQL is an open-source relational database management system (RDBMS) that provides a robust and scalable solution for storing and managing structured data. It supports SQL queries and is widely used in web development for its reliability and performance.

4. **PHP**: The programming language layer of the stack. PHP is a server-side scripting language commonly used for building dynamic web pages and web applications. It integrates seamlessly with Nginx and MySQL to generate dynamic content, interact with databases, and handle user requests.

Together, the LEMP stack components work in harmony to power web servers and deliver dynamic content to users. It offers a versatile and efficient platform for hosting a wide range of web applications, from blogs and e-commerce sites to enterprise-level systems.


## Implementation 

After creating the EC2 instance on AWS like we did on LAMP Stack we will SSH into it but this time we will use Git Bash

```bash
sh -i TestKey.pem ubuntu@51.20.3.33
```

 
Installing NginX server

We will start by updating our server

```bash
sudo apt update
```

We will then install nginx

```bash
sudo apt install nginx
```

To verify that it was installed correctly, we will run the following command

```bash
sudo systemctl status nginx
```
 
We have just launched our first server in the clouds!! 😊



Let us now open TCP port 80 which is the default port that web browsers use to access web pages on the internet 

We will use the code below

```
curl http://localhost:80

or use 

curl http://127.0.0.1:80
```

 

Let us test to see if our Nginx server can respond to requests on the internet
 

One can accesss their EC2 instance IP using the following command 

```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
 

Installing MySQL
Just like we did on LAMP stack, we will install MySQL

```
sudo apt install mysql-server
```

To log in into the mysql console

```
sudo mysql
```
 
To exit mysql

```
mysql>exit
```

The security script comes pre-installed with mysql. This script removes some insecure settings and lock down access to the database system.

```
$ *sudo mysql_secure_installation*
```


To Test whether I can log into mysql

```
sudo mysql -p
 ```

We have installed Nginx and MySQL. We will now install PHP just like we did in LAMP stack.

Installing PHP
In Apache, PHP processing is handled internally within each request, whereas Nginx requires an external program, php-fpm, to manage PHP processing. This setup enhances performance for PHP-based websites but requires additional configuration. Additionally, php-mysql is needed to facilitate PHP communication with MySQL databases. Core PHP packages are installed automatically as dependencies.

```
$ sudo apt install php-fpm php-mysql
```

Configuring PHP to use NginX processor
We will create ProjectLemp as an example domain name.
 We will create a directory within var/www for our domain ProjectLemp
Let us create the root web directory for Lempstack

```
sudo mkdir /var/www/projectLEMP
```

We will now assign ownership of the directory with the $USER environment variable which will reference my current system user

```
sudo chown -R $USER:$USER /var/www/projectLEMP/
```

We will open a new configuration file in the Nginx sites available directory 

```
sudo nano /etc/nginx/sites-available/projectLAMP
```

and have the following Bare-bones configuration 


```
#/etc/nginx/sites-available/projectLEMP
server {
listen 80;
server_name projectLEMP www.projectLEMP;
root /var/www/projectLEMP;
index index.html index.htm index.php;
location / {
try_files $uri Şuri/ =404;
}
}
location~ \.php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
}
location ~ /\.ht {
deny all;
}
```

This is what the code block means

1. `server` block:
   - Defines the server configuration.
   - `listen 80`: Specifies that the server listens on port 80.
   - `server_name`: Specifies the domain names associated with this server.
   - `root`: Sets the root directory for the server.
   - `index`: Defines the default files to be served if no file is specified in the URL.
   - `location /`: Handles requests for resources other than PHP files.
      - `try_files $uri $uri/ =404;`: Attempts to serve the requested URI directly, or if it's a directory, tries appending a slash. If neither is successful, returns a 404 error.

2. `location ~ \.php$` block:
   - Handles requests for PHP files.
   - `include snippets/fastcgi-php.conf;`: Includes configuration settings for processing PHP files via FastCGI.
   - `fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;`: Specifies the FastCGI server to which PHP requests should be passed.

3. `location ~ /\.ht` block:
   - Blocks access to .htaccess files, which are typically used in Apache setups but not needed in Nginx.

Overall, this configuration file ensures that Nginx serves the appropriate files and routes PHP requests to the PHP FastCGI Process Manager (php-fpm) for processing. It also prevents access to sensitive .htaccess files.

We will now activate our configuration by linking to the config file.

```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```


Check for any syntax error in the config file

```
sudo nginx -t
```


Let us now disable default nginx host configured to listen on port 80

```
sudo unlink /etc/nginx/sites-enabled/default
```

To apply all these changes we will need to reload nginx

```
sudo systemctl reload nginx
```

We will now create a index.html file to test our new server block

```
sudo sh -c "echo 'Hello LEMP from hostname' \$(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' \$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)" > /var/www/projectLEMP/index.html
```



Testing PHP with NginX

To validate that NginX can correctly handle .php files off to our PHP processor, We will create a test PHP file in our document root called info.php
```
nano /var/www/projectLEMP/info.php
```

PHP code to return info about server

```
<?php
phpinfo();
```


RETRIEVING DATA FROM MYSQL DATABASE WITH PHP
We will create a test database with a  ‘’To do list’’ and configure access to it so Nginx server can query data from it and display it.

•	First, connect to the MySQL console using the root account:
```
sudo mysql
```
•	Create a new database, run the following command from your MySQL console:
```
CREATE DATABASE `example_database`;
```

•	We creates a new user named example_user, using mysql_native_password as default authentication method
```
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Password';
```

•	Now we need to give this user permission over the example_database database:
 ```
 GRANT ALL ON example_database.* TO 'example_user'@'%';
```
•	Now exit the MySQL shell with
              exit

You can test if the new user has the proper permissions by logging in to the MySQL console again
              mysql -u example_user -p
 

•	After logging in to the MySQL console, confirm that you have access to the example_database database
                SHOW DATABASES;

 

Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement

           CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT, content VARCHAR(255), PRIMARY KEY(item_id));
Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:
           INSERT INTO example_database.todo_list (content) VALUES ("My first important item");



To confirm that the data was successfully saved to your table, run:

              SELECT * FROM example_database.todo_list;

Output

 


•	You can exit the MySQL console after confirming valid data in your test table
                 exit


•	Now you can create a PHP script that will connect to MySQL and query for your content
              nano /var/www/projectLEMP/todo_list.php


•	Copy this content into your todo_list.php script:

```
                    <?php
                    $user = "example_user";
                    $password = "password";
                    $database = "example_database";
                    $table = "todo_list";

                    try {
                      $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
                      echo "<h2>TODO</h2><ol>";
                      foreach($db->query("SELECT content FROM $table") as $row) {
                        echo "<li>" . $row['content'] . "</li>";
                      }
                      echo "</ol>";
                    } catch (PDOException $e) {
                        print "Error!: " . $e->getMessage() . "<br/>";
                        die();
                    }
```

This PHP code connects to a MySQL database (example_database) on localhost using the provided username and password. It selects data from the todo_list table and displays it as a list of TODO items.
If there are any errors during the database connection or query execution, it catches the exception and prints the error message.
We can now access the page on our Web Browser
```
http://<Public_domain_or_IP>/todo_list.php
```

 

Our PHP environment is ready to connect and interact with or SQL server 😊😊
