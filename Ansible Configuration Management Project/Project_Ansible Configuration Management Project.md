# Ansible Configuration Management Project

### **Project Overview**

This project will enhance our appreciation for DevOps tools by automating routine tasks with Ansible Configuration Management. Concurrently, we will gain confidence in writing code using declarative languages such as YAML.

**Ansible Client as a Jump Server (Bastion Host)**

A Jump Server (sometimes referred to as a Bastion Host) is an intermediary server that provides access to an internal network. In a typical architecture, web servers are within a secured network that cannot be directly accessed from the Internet. This setup means that even DevOps engineers cannot SSH directly into the web servers and must go through a Jump Server. Using a Jump Server improves security and reduces the attack surface.

**Tasks**

1. Install and configure Ansible client to act as a Jump Server/Bastion Host.
2. Create a simple Ansible playbook to automate server configuration.

In the diagram below, the Virtual Private Network (VPC) is divided into two subnets:
- **Public Subnet:** Contains public IP addresses.
- **Private Subnet:** Accessible only by private IP addresses.

<img width="514" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5f6473f0-e96b-4220-acb4-ced6e1c28acf">

### Step 1 - Install and Configure Ansible on EC2 Instance

1. Update the name tag on your Jenkins EC2 instance to "Jenkins-Ansible." This server will be used to run playbooks.

<img width="791" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/cdd54a02-7945-462f-a3ac-b319680f07d0">

2. In your GitHub account create a new repository and name it `ansible-config-mgt`

<img width="650" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/3a99c234-33f0-4358-a252-e9a915de2809">

3. **Install Ansible**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ansible -y
```

 **Verify Ansible Installation**

  ```bash
  ansible --version
  ```
<img width="706" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/80410e28-a60d-4c4a-9e95-cc70844b0393">



4. **Configure Jenkins Build Job**
   - Set up Jenkins to save your repository content every time you make changes. This will reinforce the Jenkins configuration skills you acquired in Project 9.

    **Configure a Webhook in GitHub:**
     - Set the webhook to trigger the Ansible build. 
     - In your `ansible-config-mgt` repository on GitHub, navigate to **Settings > Webhooks > Add webhook**. 
       - Enter the payload URL of your Jenkins server.
       - Select `application/json` as the content type.
       - Choose the events that will trigger the webhook (e.g., `push` events).
       - Save the webhook.

  <img width="862" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c4c498fc-dade-4df1-b41a-9458aa068b1c">

  **Create a new Freestyle project `ansible` in Jenkins**

  <img width="835" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9f8ecdce-cc3a-453f-b741-9b66704561bc">
  

  **Point it to the `ansible-config-mgt` repository Copy the repository URL**

  In configuration of our ansible freestyle project choose Git, provide there the link to our ansible-config-mgt GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
  
  <img width="846" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f5bac65c-6a48-4799-b728-390d29bd7936">


  <img width="369" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/db7a899f-16a8-4fe2-9884-3e4f7579b9ff">
  

  Configure a Post-build job to save all (**) files, like we did it in Project 9.

  <img width="476" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/88840d0f-54c6-4ecb-b69e-5d128a09fc5c">


  5. **Test your setup by making some change in `README.MD` file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder**

  <img width="540" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2d5d8d42-9b6a-4056-9942-e54e29789203">

  <img width="856" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c5aabd43-97e9-4539-9e9d-066f4c029162">

  ```
  ls /var/lib/jenkins/jobs/ansible/builds/1/archive/
  ```

  <img width="491" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/255e72ac-ba98-413d-a611-251cb5cd9639">

<br>

**Tip**: Allocate an Elastic IP to your Jenkins-Ansible server to avoid reconfigure of GitHub webhook to a new IP address anytime you stop/start your Jenkins-Ansible server.

Allocate elastic IP

<img width="504" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/c3be470e-cb5f-4b2f-966e-4ee62cffc621">

Associate the elastic IP

<img width="506" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/58a03843-b75a-46fe-a53f-85c5ac760520">

Update the webhook with the new Elastic Public IP

<img width="707" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9acd03fa-3d58-4550-9a82-a9fc51270492">


Note: Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.


### Step 2 – Prepare Your Development Environment Using Visual Studio Code

1. **Choose and Install an IDE:**
   - The first part of DevOps is **Dev**, which means you will need to write code and have the proper tools to make coding and debugging comfortable. You need an Integrated Development Environment (IDE) or Source-code Editor.
   - There is a plethora of different IDEs and Source-code Editors for different languages, each with its own advantages and drawbacks. You can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – **Visual Studio Code (VSC)**.

2. **Configure Visual Studio Code:**
   - After successfully installing VSC, configure it to connect to your newly created GitHub repository.
     - Open Visual Studio Code.
     - Go to **View > Command Palette** or press `Ctrl+Shift+P`.
     - Type `Git: Clone` and select the option.
     - Enter your GitHub repository URL and select the location on your local machine where you want to clone the repository.
     - Once cloned, you can start working on your code directly within Visual Studio Code.
    
       
    
     <img width="911" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9093da4c-b010-490b-a662-d03c73e8fa07">


### Step 3 - Begin Ansible Development

1. **Create a New Branch for Development:**
   - In your `ansible-config-mgt` GitHub repository, create a new branch dedicated to developing a new feature.
     - Tip: Give your branches descriptive and comprehensive names. For example, if you use project management tools like Jira or Trello, include the ticket number in the branch name along with a brief description of the branch's purpose (e.g., bugfix, hotfix, feature, release).
   
   ```bash
   git checkout -b ansible-config
   ```

2. **Checkout the Newly Created Feature Branch:**
   - Fetch the latest changes and checkout the new feature branch to your local machine.
   ```bash
   git checkout ansible-config
   ```

3. **Create Directory for Playbooks:**
   - Create a directory named `playbooks` to store all your playbook files.
   ```bash
   mkdir playbooks
   ```

4. **Create Directory for Inventory:**
   - Create a directory named `inventory` to keep your hosts organized.
   ```bash
   mkdir inventory
   ```

5. **Create Your First Playbook:**
   - Within the `playbooks` folder, create your first playbook and name it `common.yml`.
   ```bash
   touch playbooks/common.yml
   ```

6. **Create Inventory Files for Each Environment:**
   - Within the `inventory` folder, create an inventory file (`.yml`) for each environment: Development, Staging, Testing, and Production. Name them `dev.yml`, `staging.yml`, `uat.yml`, and `prod.yml` respectively.
   ```bash
   touch inventory/dev.yml inventory/staging.yml inventory/uat.yml inventory/prod.yml
   ```

    <img width="251" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/ff518209-d256-4fa0-800f-6322e47fd44f">


### Step 4 - Set Up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts and ensure the intended configuration on a particular server, it is important to organize our hosts in such an inventory.

Save the below inventory structure in the `inventory/dev.yml` file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

```yaml
# inventory/dev.yml
all:
  hosts:
    dev_server:
      ansible_host: <your-dev-server-ip>
  children:
    webservers:
      hosts:
        dev_web1:
          ansible_host: <your-dev-web1-ip>
        dev_web2:
          ansible_host: <your-dev-web2-ip>
    dbservers:
      hosts:
        dev_db1:
          ansible_host: <your-dev-db1-ip>
```

Note: Ansible uses TCP port 22 by default, which means it needs to SSH into target servers from the Jenkins-Ansible host. For this, you can implement the concept of `ssh-agent`. You need to import your key into `ssh-agent`.

#### To learn how to set up SSH agent and connect Visual Studio Code to your Jenkins-Ansible instance, please refer to the following resources:
- For Windows users: [ssh-agent on Windows](https://www.youtube.com/watch?v=OplGrY74qog)
- For Linux users: [ssh-agent on Linux](https://www.youtube.com/watch?v=OplGrY74qog)

### Start the SSH Agent

This starts the SSH agent in your current terminal session and sets the necessary environment variables.

```bash
eval `ssh-agent -s`
```
![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0c9ee207-828c-4687-9802-2009a6cbd7f8)

### Add Your SSH Key

Add your SSH private key to the agent. Replace `<path-to-private-key>` with the correct path to your private key.

```bash
ssh-add <path-to-private-key>
```

Replace `<path-to-private-key>` with the actual path to your SSH private key file. This will allow Ansible to SSH into your target servers from the Jenkins-Ansible host.

<img width="543" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f2ee8ea3-3509-4ecc-95b2-20c8113daecd">


### Verify the Key is Loaded

Check that your key has been successfully added to the SSH agent. You should see the name of your key.

```bash
ssh-add -l
```

This command will list the keys that are currently loaded into the SSH agent. If your key has been successfully added, its name will be displayed in the output.

<img width="555" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/1d127a55-b4d9-47c0-90e8-ce1b8f9bd05c">

Now, ssh into your Jenkins-Ansible server using ssh-agent

```
ssh -A ubuntu@public-ip

ssh -A ubuntu@13.50.43.191
```
<img width="512" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dab567cd-8499-44e6-aabb-ddeb06a46a0d">

<img width="566" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a0200ac8-27c0-4203-839e-4cfd83c1fd65">


Also, note that your Load Balancer user is `ubuntu` and the user for RHEL-based servers is `ec2-user`.

### Update Your `inventory/dev.yml` File

Update your `inventory/dev.yml` file with the following snippet of code. Replace the placeholder IP addresses with your actual server IP addresses:

```yaml
all:
  children:
    nfs:
      hosts:
        <NFS-Server-Private-IP-Address>:
          ansible_ssh_user: ec2-user
    webservers:
      hosts:
        <Web-Server1-Private-IP-Address>:
          ansible_ssh_user: ec2-user
        <Web-Server2-Private-IP-Address>:
          ansible_ssh_user: ec2-user
    db:
      hosts:
        <Database-Private-IP-Address>:
          ansible_ssh_user: ubuntu
    lb:
      hosts:
        <Load-Balancer-Private-IP-Address>:
          ansible_ssh_user: ubuntu
```

Replace the following placeholders:
- `<NFS-Server-Private-IP-Address>`
- `<Web-Server1-Private-IP-Address>`
- `<Web-Server2-Private-IP-Address>`
- `<Database-Private-IP-Address>`
- `<Load-Balancer-Private-IP-Address>`

with the corresponding IP addresses of your servers.

<img width="437" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/9311bd75-5885-4fcb-81fb-6a9a63cc7659">


### Step 5 - Create a Common Playbook

It's time to start giving Ansible instructions on what tasks to perform on all servers listed in `inventory/dev`.

In `common.yml` playbook, you will write configurations for repeatable, reusable, and multi-machine tasks common to systems within your infrastructure.

### Update Your `playbooks/common.yml` File

Update your `playbooks/common.yml` file with the following code:

```yaml
---
- name: Update web and NFS servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: true
  become_user: root
  tasks:
    - name: Ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: Update LB and DB servers
  hosts: lb, db
  remote_user: ubuntu
  become: true
  become_user: root
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: Ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

This playbook defines tasks to update different groups of servers:
- **web and NFS servers** (using `yum` package manager)
- **LB and DB servers** (using `apt` package manager)

Ensure you have correctly specified the hosts and user credentials (`remote_user`) according to your inventory setup (`inventory/dev.yml`). Adjust the tasks and package names (`wireshark` in this example) as per your specific requirements.


### Enhancing `common.yml` Playbook

Feel free to update this playbook with the following tasks to further customize your infrastructure configuration:

- Create a directory and a file inside it

- Change timezone on all servers

- Run some shell script

```yaml
---
- name: Update web and NFS servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: true
  become_user: root
  tasks:
    - name: Ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest
    
    - name: Create a directory and a file inside it
      file:
        path: /path/to/directory/example_file.txt
        state: touch

    - name: Change timezone on all servers
      timezone:
        timezone: America/New_York

    - name: Run some shell script
      shell: |
        #!/bin/bash
        echo "Hello, this is a shell script running on {{ inventory_hostname }}"
        # Add your shell script commands here

- name: Update LB and DB servers
  hosts: lb, db
  remote_user: ubuntu
  become: true
  become_user: root
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: Ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

    - name: Create a directory and a file inside it
      file:
        path: /path/to/directory/example_file.txt
        state: touch

    - name: Change timezone on all servers
      timezone:
        timezone: America/New_York

    - name: Run some shell script
      shell: |
        #!/bin/bash
        echo "Hello, this is a shell script running on {{ inventory_hostname }}"
        # Add your shell script commands here
```

### Explanation:

#### Directory and File Creation
- **`file`**: Ansible module used to manage files and directories.
- **`path`**: Path to the directory and file you want to create.
- **`state: touch`**: Ensures the file exists (creates it if it doesn't).

#### Change Timezone
- **`timezone`**: Ansible module to manage timezones on remote servers.
- **`timezone: America/New_York`**: Example of changing the timezone to Eastern Time.

#### Run Shell Script
- **`shell`**: Ansible module used to execute shell commands or scripts.
- **`#!/bin/bash`**: Example of a Bash script header.
- **`echo "Hello..."`**: Sample command to print a message indicating the script is running.

### Additional Learning Resources

For a deeper understanding of Ansible playbooks:
- Watch this video from RedHat: [Ansible Playbooks Video](https://www.youtube.com/watch?v=ZAdJ7CdN7DY)
- Read this article: [What is an Ansible Playbook?](https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook)

These resources will help you grasp the concepts and best practices of using Ansible playbooks effectively in your infrastructure management. Adjust the tasks and commands in the playbook according to your specific requirements and infrastructure setup.


### Step 6 - Update GIT with the Latest Code

Now that all your directories and files are set up locally, it's time to push the changes to GitHub.

In real-world scenarios, collaboration with other DevOps engineers and developers using Git is crucial. Many organizations follow a development rule known as the "Four eyes principle," where code cannot be deployed until it has been reviewed by another team member. Once you have your changes in a separate branch, you'll need to raise a Pull Request (PR), have your branch peer-reviewed, and then merge it into the main branch.

### Commit Your Code to GitHub

Use Git commands to add, commit, and push your branch to GitHub:

```bash
# Assuming you are in the root directory of your Git repository

# Add all changes to the staging area
git add .

# Commit the changes with a descriptive message
git commit -m "Added initial configuration files and playbooks"

# Push your branch to GitHub (replace <branch-name> with your actual branch name)
git push origin <branch-name>
```

Replace `<branch-name>` with the name of your branch (e.g., `ansible-config`). This sequence of commands adds all changes to the staging area, commits them with a meaningful message, and then pushes the branch to your GitHub repository.

<img width="504" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7a7fbfb6-4c4c-46af-a390-8421452496a2">

### Create a pull request and review the changes.

- Open your GitHub repository

- Create a Pull Request (PR)

<img width="856" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/1941694c-684e-4982-9deb-3bd58715dd34">

- Wear the hat of another developer for a second, and act as a reviewer.

<img width="932" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/02f4be35-4c3d-477c-822c-bb5f7d556365">

- If the reviewer is happy with your new feature development, merge the code to the main branch

<img width="789" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/329bbb85-fb6e-45f7-808c-a7501882d4f3">

<img width="545" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7c8121fc-3b91-4cf2-ad3c-0255fa9988f3">

- Head back on your terminal, checkout from the ansible-config branch into the main, and pull down the latest changes

<img width="574" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/be67cf05-e9fa-4264-89ff-9c17f6278c41">

- ﻿Once your code changes appear in main branch - Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-
Ansible server.
  
<img width="609" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5f96c137-7a50-4af4-b453-aa1dcdc16528">








