# DevOps Tooling Website Solution

## Introduction
This project involves the implementation of a solution that consists of the following components:

- **Infrastructure**: AWS
- **Web Server Linux**: Red Hat Enterprise Linux 9
- **Database Server**: Ubuntu Linux + MySQL
- **Storage Server**: Red Hat Enterprise Linux 9 + NFS Server
- **Programming Language**: PHP
- **Code Repository**: GitHub

**Architecture**

We will be implementing a solution that comprises multiple web servers sharing a common database and also accessing the same files using Network File System (NFS) as shared file storage.

<img width="572" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2646ddc9-9b65-40ec-bc9e-3c14cabe15c6">


## Step 1 - Prepare NFS Server

#### 1. Launch an EC2 Instance with RHEL Linux 8 Operating System

1. **Open the AWS Management Console**:
   - Navigate to the EC2 Dashboard.

2. **Launch an EC2 Instance**:
   - Click on "Launch Instance".

3. **Select Amazon Machine Image (AMI)**:
   - Choose **Red Hat Enterprise Linux (RHEL) 8**.

4. **Choose Instance Type**:
   - Select an appropriate instance type (e.g., `t2.medium`).

5. **Configure Instance Details**:
   - Ensure the instance is launched in the same Availability Zone as your other resources.
   - Configure the network and subnet settings as needed.

6. **Add Storage**:
   - Add three new EBS volumes, each of 10 GB:
     - Click "Add New Volume" and set the size to 10 GB.
     - Repeat this step to create a total of three additional volumes.
   - Ensure all volumes are in the same Availability Zone as the EC2 instance.

7. **Configure Security Group**:
   - Set up a security group with the necessary rules to allow NFS traffic.
     - Allow inbound TCP and UDP on port 2049 for NFS.
     - Allow inbound TCP on port 111 for the portmapper service.
     - Add additional rules as necessary for SSH and other services.

8. **Review and Launch**:
   - Review your configuration and launch the instance.
   - Select an existing key pair or create a new one to access your instance.

#### 2. Configure LVM on the Server

##### a. Connect to the EC2 Instance
```bash
ssh -i TestKey.pem ec2-user@13.60.65.197
```

##### b. Identify the New Volumes
```bash
lsblk
```
<img width="300" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/ccf10e6a-8db4-4454-9fe4-bccf3bc2b2c9">


#### Use `gdisk` Utility to Create a Single Partition on Each Disk

1. **Install `gdisk` if not already installed**:
    ```bash
    sudo yum install -y gdisk
    ```

2. **Create partitions on each disk**:
    ```bash
    sudo gdisk /dev/nvme1n1
    ```

    Follow the prompts to create a new partition:
    - `n` to create a new partition
    - `Enter` to accept default partition number
    - `Enter` to accept default first sector
    - `Enter` to accept default last sector
    - `w` to write changes and exit

    Repeat the above steps for `/dev/nvme2n1` and `/dev/nvme3n1`.

3. **Verify the partitions**:
    ```bash
    lsblk
    ```

    You should see new partitions like `/dev/nvme1n1p1`, `/dev/nvme2n1p1`, and `/dev/nvme3n1p1`.

<img width="357" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c53c6033-2051-48a7-be1a-7bede2cf32e7">

### Install LVM2 Package
```bash
sudo yum install -y lvm2
```

##### c. Create Physical Volumes
```bash
sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
```
<img width="516" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/64eaa928-6312-4524-bc05-8cbf5b488232">


##### d. Create a Volume Group
```bash
sudo vgcreate vg_nfs /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
```
<img width="557" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/67775d6a-f1d3-43a9-926a-c6c8a286013b">


##### e. Create Logical Volumes
```bash
sudo lvcreate -L 10G -n lv_apps vg_nfs
sudo lvcreate -L 10G -n lv_logs vg_nfs
sudo lvcreate -L 10G -n lv_opt vg_nfs
```
<img width="494" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f0e78769-2a1d-4d2a-a470-c797d37975ab">

<br>

<img width="335" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7a7e4625-566f-4cda-9b22-9da965d53791">


##### f. Format Logical Volumes as XFS
```bash
sudo mkfs.xfs /dev/vg_nfs/lv_apps
sudo mkfs.xfs /dev/vg_nfs/lv_logs
sudo mkfs.xfs /dev/vg_nfs/lv_opt
```

<img width="475" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7ae4c2dd-806c-4cf0-ab3f-061d5c138f14">


##### g. Create Mount Points and Mount Logical Volumes
```bash
sudo mkdir -p /mnt/apps
sudo mkdir -p /mnt/logs
sudo mkdir -p /mnt/opt
```
```
sudo mount /dev/vg_nfs/lv_apps /mnt/apps
sudo mount /dev/vg_nfs/lv_logs /mnt/logs
sudo mount /dev/vg_nfs/lv_opt /mnt/opt
```

##### h. Update `/etc/fstab` for Persistent Mounts
Add the following lines to `/etc/fstab` to ensure the mounts persist after reboot:
```bash
/dev/vg_nfs/lv_apps /mnt/apps xfs defaults 0 0
/dev/vg_nfs/lv_logs /mnt/logs xfs defaults 0 0
/dev/vg_nfs/lv_opt /mnt/opt xfs defaults 0 0
```

#### 3. Install NFS Server, Configure it to Start on Reboot, and Ensure it is Running

##### a. Install NFS Server
```bash
sudo yum -y update
sudo yum install nfs-utils -y
```

##### b. Enable and Start NFS Server
```bash
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

<img width="796" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/412b46d2-d9da-4efa-af07-dab36698008b">


#### 4. Export the Mounts for Web Servers subnet cidr(IPv4 cidr) to connect as clients. For simplicity, all 3 Web Servers are installed in the same subnet but in production set up, each tier should be separated inside its own subnet or higher level of security

##### a. Set Up Permissions for the Mount Points
```bash
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```

##### b. Configure Access to NFS for Clients within the Same Subnet
1. Check your subnet CIDR:
   - Open your EC2 details in the AWS web console.
   - Locate the 'Networking' tab and open the Subnet link.

2. Edit the NFS exports configuration:
```bash
sudo vi /etc/exports
```
Add the following lines, replacing `<Subnet-CIDR>` with your actual subnet CIDR (example: `172.31.32.0/20`):
```bash
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```
```
/mnt/apps 172.31.32.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.32.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.32.0/20(rw,sync,no_all_squash,no_root_squash)
```
Save and exit the editor (`Esc + :wq`).

##### c. Export the NFS Shares
```bash
sudo exportfs -arv
```
<img width="299" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/4fed0fde-4f94-48cd-ac5b-45318ad36a45">

##### d. Restart NFS Server to Apply Changes
```bash
sudo systemctl restart nfs-server.service
```

With these steps, the NFS server should be configured, running, and accessible to your web servers within the same subnet.

#### 5. Check which port is used by NFS and open it using the security group (add new inbound rule)
```
rpcinfo -p | grep nfs
```

<img width="328" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d10e9f6e-177b-4c95-8502-7dcf91d4ae6a">

Note: For NFS Server to be accessible from the client, the following ports must be opened: TCP 111, UDP 111, UDP 2049, NFS 2049. Set the Web Server subnet cidr as the source

<img width="687" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d5c43eb2-4618-4430-9f6a-99e08e3717c9">


## Step 2  Configure the Database Server

### 1. Launch a New EC2 Instance

Launch a new EC2 instance with Ubuntu as the operating system.

### 2. Install MySQL Server

After logging into the Ubuntu instance, update the package index and install the MySQL server package:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install mysql-server
```

### 3. Create a Database

Once MySQL is installed, log in to the MySQL shell as the root user:

```bash
sudo mysql -u root -p
```

Enter the root password you set during installation when prompted. Then, create a new database named `tooling`:

```sql
CREATE DATABASE tooling;
```

### 4. Create a Database User

Next, create a database user named `webaccess`:

```sql
CREATE USER 'webaccess'@'%' IDENTIFIED BY 'Password';
```

### 5. Grant Permissions

Grant the necessary permissions to the `webaccess` user on the `tooling` database, allowing access only from the subnet CIDR of your web servers. Replace `<subnet-CIDR>` with the actual CIDR of your web servers:

```sql
GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'<subnet-CIDR>';
FLUSH PRIVILEGES;
```

For example, if your web servers' subnet CIDR is `172.31.32.0/20`, you would replace `<subnet-CIDR>` with `172.31.32.0/20`.

### 6. Exit MySQL Shell

Exit the MySQL shell:

```sql
exit;
```

Now, our MySQL database server is configured with a database named `tooling`, a user named `webaccess`, and the appropriate permissions for the web servers' subnet CIDR.

<img width="493" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/216d2b72-e890-4ee3-99ac-98a19ec04ed8">

Set Bind Address and restart MySQL
```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

```
sudo systemctl restart mysql
sudo systemctl status mysql
```

<img width="578" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b5551292-b46f-4c48-b68e-391f3a596ce9">

Open MySQL port 3306 on the DB Server EC2.
Access to the DB Server is allowed only from the Subnet Cidr configured as source.

## Step 3 - Prepare the Web Servers

There is need to ensure that the Web Servers can serve the same content from a shared storage solution, in this case - NFS and MySQL database. One DB can be accessed for read and write by multiple clients. For storing shared files that the Web Servers will use, NFS is utilized and previousely created Logical Volume lv-apps is mounted to the folder where Apache stores files to be served to the users (/var/www).

This approach makes the Web server stateless which means they can be replaced when needed and data (in the database and on NFS) integrtity is preserved

In further steps, the following was done:

- Configured NFS (This step was done on all 3 servers)
- Deployed a tooling application to the Web Servers into a shared NFS folder
- Configured the Web Server to work with a single MySQL database

 ### Web Server 1
 
1. Launch a new EC2 instance with RHEL Operating System
2. Install NFS Client

```
sudo yum install nfs-utils nfs4-acl-tools -y
```
3. Mount /var/www/ and target the NFS server's export for apps. NFS Server private IP address = 172.31.43.22
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.43.22:/mnt/apps /var/www
```

<img width="544" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/83e35d65-360e-4df3-8557-c797acf8fc4a">

```
sudo vi /etc/fstab
```

4. Add the following line
(NFS Server private IP address = 172.31.43.22)

```
172.31.43.22:/mnt/apps /var/www nfs defaults 0 0
```

5. Install Remi's repoeitory, Apache and PHP
```
sudo yum install git -y
sudo yum install httpd -y
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
sudo dnf module reset php -y -y
sudo dnf module enable php:remi-7.4 -y
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y -y
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1

```

<img width="482" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/6b75f2ff-a47f-4f89-9ff2-f76a56eec371">

Repeat steps 1–5 for another 2 Web servers.

6. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files, it means NFS is mounted correctly. You can test this by creating a new file from one web server and check if it is accessible from other web servers.

<img width="448" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/ac605b36-fafe-4f51-9c81-8a3571100e46">

7. Locate the log folder for Apache on the Web Server and mount it to NFS server's export for logs. Repeat step 4 to ensure the mount point persists after reboot.
```
sudo vi /etc/fstab
```
Add the following line
```
172.31.43.22:/mnt/logs /var/log/httpd nfs defaults 0 0
```

8. Fork the tooling source code from StegHub GitHub Account

<img width="932" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2f63d10f-4861-4c43-9d08-2c95f764422f">

9. Deploy the tooling Website's code to the Web Server. Ensure that the html folder from the repository is deployed to /var/www/html

<img width="564" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/3c56b8cd-d7f5-4382-bb2c-90d456852ef6">

```
sudo cp -R html/. /var/www/html
```

![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e158a346-4bfd-444a-a807-7f5c9f0696a2)

Note: Acces the website on a browser

- Ensure TCP port 80 is open on the Web Server.
- If 403 Error occur, check permissions to the /var/www/html folder and also disable SELinux

```  
sudo setenforce 0
```

To make the change permanent, open selinux file and set selinux to disable.

```
sudo vi /etc/sysconfig/selinux

SELINUX=disabled

sudo systemctl restart httpd
```

10. Update the website's configuration to connect to the database (in /var/www/html/function.php file). Apply tooling-db.sql command sudo mysql -h <db-private-IP> -u <db-username> -p <db-password < tooling-db.sql

```
sudo vi /var/www/html/functions.php
```

// connect to database
mysqli_connect('172.31.38.185', 'webaccess', 'Password', 'tooling');

<img width="759" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e70ede9c-3651-428c-87f3-0125b9e7d046">

```
sudo mysql -h 172.31.38.185 -u webaccess -p tooling < tooling-db.sql
```

Access the database server from Web Server

```
sudo mysql -h 172.31.38.185 -u webaccess -p
```

<img width="676" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/82545e3f-d009-4ec5-9ad4-0da6f93ae7c7">

11. Create in MySQL a new admin user with username: myuser and password: password
```
INSERT INTO users(id, username, password, email, user_type, status) VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
```

12. Open a browser and access the website using the Web Server public IP address http://<Web-Server-public-IP-address>/index.php. Ensure login into the website with myuser user.

<img width="900" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a5f06637-cd97-4043-8cbf-dd9e25f9f930">

<img width="943" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5f959338-8f3d-4da2-9031-e32a82815b6f">


#### *Conclusion*

We have successfully implemented and deployed a DevOps tooling website solution that makes access to DevOps tools within a corporate infrastructure easily accessible. This comprises multiple web servers sharing a common database and also accessing the same files using Network File System (NFS) as shared file storage.
