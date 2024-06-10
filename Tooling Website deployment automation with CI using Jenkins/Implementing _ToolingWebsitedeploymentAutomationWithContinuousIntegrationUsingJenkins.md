# Tooling Website deployment Automation with Continuous Integration using Jenkins

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools.

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments which is then automatically built and tested before it is merged with the shared repository.

## Task

Enhance the architecture prepared in the previous project by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how the updated architecture looks.

<img width="495" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b838a487-7a53-451f-b495-9586cca42468">

## Task 1

### Step 1: Create an AWS EC2 Instance

1. **Login to AWS Management Console**:
   - Open your web browser and navigate to the [AWS Management Console](https://aws.amazon.com/console/).

2. **Launch an EC2 Instance**:
   - Navigate to the EC2 Dashboard and click on "Launch Instance".
   - Choose the **Ubuntu Server 24.04 LTS** AMI (Amazon Machine Image).
   - Choose an instance type (e.g., t3.micro for a free-tier option).
   - Configure instance details as needed.
   - Configure the security group to allow SSH (port 22) and HTTP (port 8080) access.
   - Review and launch the instance.
   - Download the key pair and keep it safe.
  
   <img width="734" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dff8fa77-99d6-4872-9689-66a2104def50">

3. **Connect to your EC2 Instance**:
   - Use the downloaded key pair to SSH into your EC2 instance:
     ```sh
     ssh -i /path/to/your-key-pair.pem ubuntu@your-ec2-public-ip
     ```

### Step 2: Install JDK

Jenkins is a Java-based application, so you'll need to install the Java Development Kit (JDK).

1. **Update the package index**:
   ```sh
   sudo apt update
   ```

2. **Install OpenJDK**:
   ```sh
   sudo apt install openjdk-17-jdk -y
   ```

3. **Verify the installation**:
   ```sh
   java -version
   ```

   <img width="457" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dbd09154-5abb-4b47-9aa4-9d0d295479e3">


### Step 3: Install Jenkins

1. **Add the Jenkins repository**:
   ```sh
   curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```

2. **Update the package index again**:
   ```sh
   sudo apt update
   ```

3. **Install Jenkins**:
   ```sh
   sudo apt install jenkins -y
   ```

4. **Start Jenkins service**:
   ```sh
   sudo systemctl start jenkins
   ```

5. **Enable Jenkins to start on boot**:
   ```sh
   sudo systemctl enable jenkins
   ```
<img width="480" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c6b41c6a-78f8-408a-9017-7f75f2c110ec">


6. **Adjust the firewall to allow traffic on port 8080**:
   ```sh
   sudo ufw allow 8080
   ```
   <img width="278" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c458161e-eb53-4f7d-838b-0702d6dfbabf">


7. **Retrieve the initial admin password**:
   ```sh
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
   Copy this password, as you will need it to unlock Jenkins.


### Step 4: Configure Jenkins

1. **Access Jenkins**:
   - Open your web browser and navigate to `http://your-ec2-public-ip:8080`.
   - Enter the initial admin password retrieved earlier.
   - Follow the setup wizard to install suggested plugins and create an admin user.

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/adb06ecb-b754-4c4b-a8e4-05add92f7383">

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/76fef4bc-4327-409a-8822-9efc6b0c730c">

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b81dc4ca-52f8-4f05-90dd-682b34794845">

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0f588251-6ad9-4ed7-a68a-2485aa25ca37">

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/6b2751d9-7bf3-4de0-b832-19e926dc2464">



## Task 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks

In this part, we will learn how to configure a simple Jenkins job/project. This job will will be triggered by GitHub webhooks and will execute a build task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings.
On your GitHub repository,

Select Settings > Webhooks > Add webhook

<img width="769" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a6208f36-bd4c-4391-ab0a-809c7d2d6bdc">

2. Go to Jenkins web console, click New Item and create a Freestyle project
<img width="927" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/6e40a2f3-8bf6-4b77-aa1a-49c62f4f3440">

Save the configuration and try to run the build. For now we can only do it manually. Click Build Now button. After all was configured correctly, the build was successfull and was seen under #5 You can open the build and check in Console Output if it has run successfully.

<img width="956" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e8b50fcb-755d-4e25-bb91-efd88185a1dc">

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click Configure our job/project and add these two configurations
- Configure triggering the job from GitHub webhook 
- Configure Post-build Actions to archive all the files - files resulted from a build are called artifacts

  <img width="466" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8832ee47-868d-4131-a2f8-8b7927a2579e">

  <img width="647" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/045214f6-282f-4ed4-890c-08b2cd8b4bf6">

  Now, go ahead and make some change in any file in our GitHub repository (e.g. README.MD file) and push the changes to the main branch.
  We will see that a new build has been launched automatically by webhook and its results - artifacts, saved on Jenkins server.

<img width="597" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/45d0f5b9-587d-4ddb-af87-22ad84233f35">

<img width="480" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f855cd49-cb60-432f-9265-b4789ab26b63">

Now we configured an automated Jenkins job that receives files from GitHub by webhook trigger this method is considered as push because the changes are being pushed and files transfer is initiated by GitHub. There are also other methods: ***trigger one job (downstreadm) from another (upstream), poll GitHub periodically*** and others.

By default, the artifacts are stored on Jenkins server locally

```
ls /var/lib/jenkins/jobs/Devops_tooling/builds/<build_number>/archive/

ls /var/lib/jenkins/jobs/Devops_tooling/builds/6/archive/
```

<img width="576" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e2897cf4-2990-47bb-b1ab-4c1fdc067e76">




## Task 3 - Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are more than 1400 plugins available. now we will need a plugin that is called Publish Over SSH

**1. Install Publish Over SSH plugin.**
On main dashboard, Select Manage Jenkins > Manage Plugins > Available > Search for Publish over SSH and Install without restart.

<img width="950" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9eae8df8-f7bc-4e63-95bb-01bdb88ad112">


**2. Configure the job/project to copy artifacts over to NFS server**
On main dashboard select Manage Jenkins > Configure System menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

- Provide a private key (content of .pem file that we use to connect to NFS server via SSH/Putty)

- Arbitrary name

- Hostname - can be private IP address of our NFS server

- Username - ec2-user (since NFS server is based on EC2 with RHEL 9)

- Remote directory - /mnt/apps since our Web Servers use it as a mounting point to retrieve files from the NFS server

<img width="855" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d9c1806a-5729-45af-9f49-9e7ad13a5b46">

Test the configuration and make sure the connection returns Success. N.B that TCP port 22 on NFS server must be open to receive SSH connections

<img width="786" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0d19f4cc-1595-4e11-a685-a0a6975bad3a">

Save the configuration, open your Jenkins job/project configuration page and add another one Post-build Action (Send build artifact over ssh).

<img width="252" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/14c4ef0b-cfaa-485b-a9a2-5cc78743b028">

Also, Configure it to send all files produced by the build into our previouslys define remote directory In our case we want to copy all files and directories, so we use ** .

<img width="586" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/567520a5-0655-456c-8af4-a47b7a313214">

Save this configuration and go ahead, change something in README.MD file in our GitHub Tooling repository

Webhook will trigger a new job #7

<img width="572" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/90810b3b-37de-4335-81b6-52ef360c6eb3">

The error in the build #5 above indicates that we need to set permissions for user ec2-user on the NFS server : Ensure the target directory (/mnt) and it's contents on the NFS server has the correct permissions. We might need to change ownership or modify the permissions to allow the Jenkins user to write to it.

```
sudo chown -R ec2-user:ec2-user /mnt/apps
sudo chmod -R 777 /mnt/apps
```

Run the build again from jenkins GUI

Webhook triggers a new job and in the Console Output of the job we get something like this:

<img width="552" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/504af785-5e4d-41e1-b5be-e2802cfa0d84">

To make sure that the files in /mnt/apps have been updated - connect via SSH to our NFS Server and verify README.MD file

```
ls /mnt/apps

or

ls -l /mnt/apps
```

<img width="395" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/633670ca-6752-41dc-a146-db57cfd5f171">

```
cat /mnt/apps/README.md
```

<img width="476" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dac28bca-d34d-4168-b2e5-c8b526b7a181">


## If you see the changes you had previously made in your GitHub - the job works as expected.


<img width="400" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b6483718-14b0-4ec3-9741-7c8f187363c6">





