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










