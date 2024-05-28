# Web Solution with Wordpress

## Part One

### Step 1: Launch a RedHat EC2 Instance
1. Go to the AWS Management Console and navigate to the EC2 dashboard.
2. Click on the "Launch Instance" button.
3. Choose the Red Hat Enterprise Linux (RHEL) AMI as your instance type.
4. Select an instance type based on your requirements (Free for our case).
5. Review and launch the instance.

<img width="552" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8197e2af-71bc-4590-87b8-9d3e5a217f3d">
   

### Step 2: Create 3 Volumes
1. After launching the instance, go to the EC2 dashboard.
2. Under the "Elastic Block Store" section, click on "Volumes".
3. Click on the "Create Volume" button.
4. Choose the same Availability Zone (AZ) as your web server EC2 instance.
5. Set the volume size to 10GB.
6. Repeat this process to create two more volumes, each with 10GB size.

<img width="738" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0b049e3f-f797-475e-84c2-90e9aa4e9b07">


### Step 3: Attach Volumes to the Web Server EC2 Instance
1. Once the volumes are created, select the first volume from the list.
2. Click on "Actions" and then select "Attach Volume".
3. Choose the web server EC2 instance from the dropdown menu.
4. Specify the device name (e.g., /dev/xvdf).
5. Click on "Attach".
6. Repeat the same process for the remaining two volumes, ensuring to specify different device names (e.g., /dev/xvdg, /dev/xvdh).

### Step 4: Configure the Web Server
1. SSH into your web server EC2 instance using the key pair associated with it.
2. Use commands like `lsblk` to verify that the volumes are attached correctly.

Our EC2 instance is using NVMe block devices for its volumes rather than the older Xen virtualized devices (/dev/xvdX). The NVMe block devices are named differently (/dev/nvmeXnY).

AWS started transitioning to NVMe-based storage for EC2 instances, especially for newer instance types and regions. These NVMe block devices provide better performance and are more optimized for modern workloads.

<img width="294" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c86001ab-c0bb-476c-80e8-9711ddfb16f2">

### Step 4: Use df -h to see all mounts and free space on the server.

```
df -h
```

<img width="328" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9fb64ca6-6100-42eb-ad66-4f65fdcf663f">

### Step 5: Use gdisk utility to create a single partition on each of the 3 disks.

For NVMe volumes, you'll typically use `gdisk`, `parted`, or `fdisk` utilities. Here's how you can create a single partition on each of the 3 NVMe disks using the `gdisk` utility:

```bash
sudo gdisk /dev/nvme1n1
```

Once you're in the `gdisk` interactive interface:

1. Type `n` to create a new partition.
2. Press Enter to accept the default partition number.
3. Press Enter to accept the default starting sector.
4. Press Enter to accept the default ending sector, which will use the entire disk.
5. Type `w` to write the changes to the disk and exit.

Repeat the same process for `/dev/nvme2n1` and `/dev/nvme3n1`.

<img width="491" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2d5d9c8d-e4f8-4040-9739-78ec77f4c137">

### Step 5b: Use lsblk utility to view the newly configured partitions on each of the 3 disks

```
lsblk
```

![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/64ee6365-e1ce-4261-80e7-d3d42ef626c1)

### Step 6: Install lvm package

```
sudo yum install lvm2 -y
```

<img width="942" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e6b985c6-b722-4eca-a9ec-363e0e916ddd">

```
sudo lvmdiskscan
```

![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a5e028e8-13ab-4af1-81bc-ee1efe148079)


### Step 7: Initialize Physical Volumes
Use the `pvcreate` command to mark each of the three disks as physical volumes.

1. Initialize the first disk:
   ```bash
   sudo pvcreate /dev/nvme1n1
   ```
2. Initialize the second disk:
   ```bash
   sudo pvcreate /dev/nvme2n1
   ```
3. Initialize the third disk:
   ```bash
   sudo pvcreate /dev/nvme3n1
   ```

### Step 8: Verify Physical Volumes
Use the `pvs` command to verify that each of the volumes has been successfully initialized as a physical volume.

```bash
sudo pvs
```

<img width="446" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/19d2e5a5-3416-4662-8ddf-04758a6a4e53">

This output indicates that the three disks `/dev/nvme1n1`, `/dev/nvme2n1`, and `/dev/nvme3n1` have been successfully marked as physical volumes and are ready to be used by LVM.

### Step 9: Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg. Verify that the VG has been created successfully

```
sudo vgcreate webdata-vg /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1

sudo vgs
```
<img width="542" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d16f8d39-3fb2-4969-8ac7-8ff41a657ece">

### Step 10: Use lvcreate utility to create 2 logical volume, apps-lv (Use half of the PV size), and logs-lv (Use the remaining space of the PV size). Verify that the logical volumes have been created successfully.

Note: apps-lv is used to store data for the Website while logs-lv is used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg

sudo lvcreate -n logs-lv -L 14G webdata-vg

sudo lvs
```
<img width="548" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/88d63732-9625-45bf-83e6-b2149bbac38c">

**Verify the entire setup**

sudo vgdisplay -v   #view complete setup, VG, PV and LV

<img width="543" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7a77140b-7d2b-4ea0-a5e5-59d321454b6c">

```
lsblk
```

<img width="370" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/35c46f0a-42d6-4968-892c-6f4dc5d0b51b">

### Step 11: Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
sudo mkfs.ext4 /dev/webdata-vg/apps-lv

sudo mkfs.ext4 /dev/webdata-vg/logs-lv
```

### Step 12 

To create the directories `/var/www/html` to store website files and `/home/recovery/logs` to store backup log data, and then mount the `/var/www/html` directory on the `apps-lv` logical volume, you can follow these steps:

#### Step A: Create Directories

1. **Create `/var/www/html` directory**:
   ```bash
   sudo mkdir -p /var/www/html
   ```

2. **Create `/home/recovery/logs` directory**:
   ```bash
   sudo mkdir -p /home/recovery/logs
   ```

#### Step B: Mount `apps-lv` on `/var/www/html`

1. **Mount the `apps-lv` logical volume on `/var/www/html`**:
   ```bash
   sudo mount /dev/webdata-vg/apps-lv /var/www/html
   ```

#### Step C: Verify Mount

After mounting, you can verify that the mount was successful by checking the output of the `df -h` command or inspecting the `/etc/fstab` file.

```bash
# Verify mount
df -h
```

### Output Verification

After running `df -h`, you should see `/dev/webdata-vg/apps-lv` mounted on `/var/www/html`:

<img width="416" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a350771b-5d8e-48d6-b0dd-127e0ac0313b">

### Step 13

Before mounting the filesystem, it's crucial to perform the backup to ensure data integrity. You can use the `rsync` utility to synchronize the contents of the `/var/log` directory to the `/home/recovery/logs` directory.

```bash
sudo rsync -av /var/log/ /home/recovery/logs
```

To mount the `/var/log` directory on the `logs-lv` logical volume, you can use the following command:

```bash
sudo mount /dev/webdata-vg/logs-lv /var/log
```

To restore the backed-up log files from `/home/recovery/logs` back into the `/var/log` directory, you can use the `rsync` command again. Here's the correct command:

```bash
sudo rsync -av /home/recovery/logs/ /var/log
```

Update /etc/fstab file so that the mount configuration will persist after restart of the server

Get the UUID of the device and Update the /etc/fstab file with the format shown inside the file using the UUID. Remember to remove the leading and ending quotes.
```
sudo blkid   # To fetch the UUID

sudo vi /etc/fstab
```
<img width="565" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/ae45b0ef-015e-4dc1-9bc2-e3b3eb3726cc">


### Step 14

To test the configuration, reload the systemd daemon, and verify the setup, follow these steps:

1. **Test the Configuration**:
   ```bash
   sudo mount -a
   ```
   This command mounts all filesystems specified in the `/etc/fstab` file, including the newly added mounts for the `logs-lv` and `apps-lv` logical volumes.

2. **Reload the systemd Daemon**:
   ```bash
   sudo systemctl daemon-reload
   ```
   This command reloads the systemd manager configuration. This is necessary after making changes to unit files or other systemd configuration files.

3. **Verify the Setup**:
   ```bash
   df -h
   ```
   This command displays information about all mounted filesystems. You can use it to verify that the mounts for `/var/log` and `/var/www/html` are present and have the expected sizes.

   <img width="507" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b1382993-a9b4-47a5-8f25-f1ad6ea428c9">

## Part Two

### Prepare the Database Server

Launch a second RedHat EC2 instance that will have a role - DB Server. Repeat the same steps as for the Web Server, but instead of apps-lv, create dv-lv and mount it to /db directory.

Sure, here's a detailed breakdown of the steps with code examples:

1. **Launch EC2 Instance**:
   - Launch a RedHat EC2 instance using the AWS Management Console or AWS CLI. Make sure to choose the appropriate instance type, security groups, and IAM role.

2. **Attach EBS Volumes**:
   - Create EBS volumes using the AWS Management Console or AWS CLI.
   - Attach the created EBS volumes to the DB Server EC2 instance. Adjust the volume size and count based on your requirements.

3. **SSH into the DB Server EC2 Instance**:
   ```bash
   ssh -i your_key.pem ec2-user@your_db_server_ip
   ```

4. **Partition and Format the Disk**:
   - If necessary, partition the attached EBS volume(s) using `gdisk`, `parted`, or `fdisk`.

  To create a single partition on each of the three disks using the `gdisk` utility, follow these steps:

   **Identify the Disks**
   Before partitioning the disks, identify the disks you want to partition. You can list the available disks using the `lsblk` command.

   ```bash
   lsblk
   ```

   Note down the device names of the disks you want to partition, such as `/dev/nvme1n1`, `/dev/nvme2n1`, and `/dev/nvme3n1`.

   **Run gdisk**:
   Start the `gdisk` utility for each disk you want to partition.

   ```bash
   sudo gdisk /dev/nvme1n1
   ```

   Replace `/dev/nvme1n1` with the appropriate device name for each disk.

   **Create a New Partition**:
   Within `gdisk`, create a new partition.

   - Press `n` and then `Enter` to create a new partition.
   - Choose the default partition number (usually 1) and press `Enter`.
   - Choose the default starting sector and press `Enter`.
   - Choose the default ending sector or specify the size of the partition as needed.
   - Set the partition type. If you're using the entire disk, you can typically use the default partition type (Linux filesystem).
   - Press `w` to write the changes to the disk and exit.

  **Repeat for Other Disks**:
   Repeat the above steps for each of the remaining disks (`/dev/nvme2n1` and `/dev/nvme3n1`).

 **Verify**:
   After creating the partitions, you can verify them using the `lsblk` command again. You should see the newly created partitions listed for each disk.

<img width="332" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/17ed3ab5-2a49-4af1-aadd-7e85bba4fa4b">


5. **Install lvm package**

   ```
   sudo yum install lvm2 -y
   ```

   Use pvcreate utility to mark each of the 3 dicks as physical volumes (PVs) to be used by LVM. Also, use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG database-vg. Verify that each of the volumes and the VG have been created successfully.

```
sudo pvcreate database-vg /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
sudo pvs
```
<img width="318" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/1a37401a-41b0-4fb4-89e0-dadaf70f91b5">

```
sudo vgcreate database-vg /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
sudo vgs
```

<img width="590" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/94e49e20-4c67-48e2-bc50-1cc8dc32e807">


6. **Create Logical Volume (LV)**:
   Use lvcreate utility to create a logical volume, db-lv (Use 20G of the PV size since it is the only LV to be created). Verify that the logical volumes have been created successfully.

   ```
   sudo lvcreate -n db-lv -L 20G database-vg
   
   sudo lvs
   ```
   <img width="650" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f40f0953-608f-4193-9336-7978e1796c26">


7. **Mount the Logical Volume**:
   To format the logical volume `db-lv` with the ext4 filesystem and then mount it to the `/db` directory, follow these steps:

 **Format the Logical Volume**:
   Use the `mkfs.ext4` command to format the logical volume `db-lv` with the ext4 filesystem.

   ```bash
   sudo mkfs.ext4 /dev/database-vg/db-lv
   ```

   This command formats the logical volume `db-lv` with the ext4 filesystem, making it ready for use.

 **Mount the Logical Volume**:
   Once the logical volume is formatted, you can mount it to the `/db` directory using the `mount` command.

   ```bash
   sudo mount /dev/database-vg/db-lv /db
   ```

   This command mounts the logical volume `db-lv` to the `/db` directory, allowing you to access it as a filesystem.

After executing these commands, the logical volume `db-lv` will be formatted with the ext4 filesystem and mounted to the `/db` directory. You can now use the `/db` directory to store data for your database.

8. **Update `/etc/fstab`**:
   - Obtain the UUID of the `db-lv` logical volume using `blkid`.
   - Update the `/etc/fstab` file with an entry for the `db-lv` logical volume using its UUID and mount point (`/db`).
       
     ```bash
     # Mount point for database logical volume
      UUID=0c62841a-15a6-4dec-8745-16f892c70af1  /db  ext4  defaults  0  0
     ```
     
   <img width="610" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/75366b00-edf7-4949-8c8c-12fc02d0ea33">


9. **Test Configuration**:
    - Run `sudo mount -a` to test the configuration.
    - Run `sudo systemctl daemon-reload` to reload the systemd daemon.
    - Use `df -h` to verify that the `db-lv` logical volume is mounted to the `/db` directory.

      ```
      sudo mount -a   # Test the configuration

      sudo systemctl daemon-reload
      
      df -h   # Verifies the setup
      ```
   <img width="429" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/629beb8d-5fea-449b-b10f-92b5e01bfa39">


## Part Three - Install WordPress on the Web Server EC2

1. Update the repository
```
sudo yum -y update
```

2. Install wget, Apache and it's dependencies
```
sudo yum install wget httpd php-fpm php-json

```
