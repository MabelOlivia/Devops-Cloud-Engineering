# Tooling Website deployment Automation with Continuous Integration using Jenkins

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools.

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments which is then automatically built and tested before it is merged with the shared repository.

### Task

Enhance the architecture prepared in the previous project by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how the updated architecture looks.

<img width="495" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b838a487-7a53-451f-b495-9586cca42468">


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

<img width="863" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/adb06ecb-b754-4c4b-a8e4-05add92f7383">

<img width="846" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/76fef4bc-4327-409a-8822-9efc6b0c730c">

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b81dc4ca-52f8-4f05-90dd-682b34794845">

<img width="749" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0f588251-6ad9-4ed7-a68a-2485aa25ca37">

<img width="811" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/6b2751d9-7bf3-4de0-b832-19e926dc2464">


### Step 5: Create a Jenkins Job to Deploy Code from Git to NFS

1. **Install Necessary Plugins**:
   - Go to `Manage Jenkins` -> `Manage Plugins` and install:
     - Git Plugin
     - NFS Plugin

2. **Create a New Jenkins Job**:
   - Go to `New Item`, enter a name for your job, and select `Freestyle project`.
   - In the job configuration:
     - **Source Code Management**:
       - Select `Git` and enter the repository URL.
       - Provide credentials if necessary.
     - **Build Triggers**:
       - Select `Poll SCM` and enter a schedule (e.g., `H/5 * * * *` for polling every 5 minutes).
     - **Build**:
       - Add a build step: `Execute shell`.
       - In the shell command, enter the script to deploy code to the NFS server. For example:
         ```sh
         git archive --format=tar HEAD | (cd /path/to/nfs/mount && tar xf -)
         ```

3. **Save and Run the Job**:
   - Save the job configuration and run it manually to test.
   - Ensure that subsequent changes to the Git repository trigger the job and deploy the code to the NFS server.

### Summary

You've now set up a Jenkins server on an AWS EC2 instance running Ubuntu 24.04 LTS, installed the necessary software, and created a Jenkins job to automate the deployment of source code changes from Git to an NFS server. This setup helps streamline your CI/CD pipeline and ensures that your code is always up to date on the NFS server.








