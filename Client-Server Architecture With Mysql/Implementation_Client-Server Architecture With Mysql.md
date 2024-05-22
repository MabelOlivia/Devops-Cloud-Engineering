# Client-Server Architecture With Mysql
 
To demonstrate a basic client-server using MYSQL RDBMS:

## Step 1:

Create and configure two linux-based virtual servers (EC2 instance in AWS)

* mysql server
* mysql client

<img width="728" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/322cdcfd-f200-46e6-8d5f-576bc4f82bf3">

## Step 2:

On mysql server Linux Server, install MySQL Server software

We can connect into the Instance using the AWS CLI

*Update and upgrade Ubuntu*

``` 
sudo apt update && sudo apt upgrade -y
```

*Install MySQL Server software*

``` 
sudo apt install mysql-server -y
```

*Enable mysql server*

``` 
sudo systemctl enable mysql
```

## Step 3:

On mysql client Linux Server install MySQL Client software.

*Update and upgrade Ubuntu*

``` 
sudo apt update && sudo apt upgrade -y
```

*Install MySQL Client software*

``` 
sudo apt install mysql-client -y
```

## Step 4:

Use mysql server's local IP address to connect from mysql client.

By default, both of the EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default so it has to be opened by creating a new entry in inbound rules in mysql server Security Groups. 

For extra security, access to mysql server by all IP addresses should not allowed, only the specific local IP address of mysql client should be allowed.


**Detailed Steps:**

1. **Identify Local IP Addresses:**
   - Go to the AWS EC2 dashboard and select your instances.
   - Note the Private IPv4 address for both the MySQL server and client instances.

2. **Update Security Group Inbound Rules:**
   - Navigate to the Security Groups section in the AWS Management Console.
   - Select the security group attached to your MySQL server instance.
   - Click on "Edit inbound rules".
   - Add a new rule:
     - Type: Custom TCP
     - Protocol: TCP
     - Port Range: 3306
     - Source: Custom (enter the private IP address of the MySQL client)
   - Save the changes.
  
    <img width="859" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9dbcd3a9-44c0-471e-80cf-e647c3e7c4e0">

3. **Configure MySQL Server:**
   - SSH into the MySQL server instance.
   - Edit the MySQL configuration file, typically found at `/etc/mysql/mysql.conf.d/mysqld.cnf` or `/etc/my.cnf`.
   - Find the line with `bind-address` and update it:
     ```ini
     bind-address = 0.0.0.0  # This allows connections from any IP address
     ```
   - Restart the MySQL service to apply changes:
     ```sh
     sudo systemctl restart mysql
     ```

    <img width="723" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/115e6423-ae40-4570-af3b-9f0a9bb1c6c0">

The security script of MySQL was run on mysql server by running the command:

```
sudo mysql_secure_installation
```
We had  not set up a password for your MySQL user during the MySQL server installation or configuration, you can either:

1. **Set up a password for your MySQL user.**
2. **Use the root user if it does not have a password (not recommended for production).**
3. **Connect without a password if your MySQL installation allows it (not recommended for security reasons).**

Here’s how you can set a password for your MySQL user:

### Set Up a Password for MySQL User

1. **Log in to MySQL as the root user:**
   ```sh
   sudo mysql
   ```

2. **Create a new user or set a password for an existing user:**

   **To create a new user:**
   ```sql
   CREATE USER 'client'@'%' IDENTIFIED WITH mysql_native_password BY 'Password';
   ```

   **To set a password for an existing user:**
   ```sql
   ALTER USER 'your_username'@'%' IDENTIFIED BY 'your_password';
   ```

3. **Grant the necessary privileges to the user:**
     ```sql
   GRANT ALL PRIVILEGES ON *.* TO 'client'@'%';
   FLUSH PRIVILEGES;
   ```
     
4. **CREATE DATABASE**
   ```sql
   CREATE DATABASE test_db;
   ```
   
5. **Exit MySQL:**
   ```sql
   EXIT;
   ```

## Step 5

From mysql client Linxus Sever, connect remotely to mysql server Database Engine without using SSH. The mysql utility must be used to perform this action.
 ```
sudo mysql -u client -h 172.31.36.228 -p
 ```

  <img width="610" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/3f644f88-fc19-4c3c-a72a-9bc4d6c02645">

## Step 6 

Check that the connection to the remote MySQL server was successfull and can perform SQL queries.

 ```
show databases;
```

  <img width="382" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0b7057ff-3800-4e9a-b46d-fadb45e35df3">

<br>
<br>

### Create tables, insert records, and select records in our `test_db` database.

*Step-by-Step Instructions*

1. **Switch to `test_db` Database:**

   ```sql
   USE test_db;
   ```

2. **Create a Table:**

   ```sql
   CREATE TABLE employees (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100),
       position VARCHAR(100),
       salary DECIMAL(10, 2)
   );
   ```

3. **Insert Records into the Table:**

   ```sql
   INSERT INTO employees (name, position, salary) VALUES
   ('John Doe', 'Manager', 75000.00),
   ('Jane Smith', 'Developer', 60000.00),
   ('Emily Johnson', 'Designer', 55000.00);
   ```

5. **Select Records from the Table:**

   ```sql
   SELECT * FROM employees;
   ```

<img width="444" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/30e2311a-ad3d-4f2d-913e-ff168d3ab28e"> <br>


<br>

***OUR PROJECT IS NOW COMPLETE 🎉🎉***
