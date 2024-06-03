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
