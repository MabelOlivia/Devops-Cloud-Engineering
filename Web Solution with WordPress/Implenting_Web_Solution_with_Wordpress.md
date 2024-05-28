# Web Solution with Wordpress


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



