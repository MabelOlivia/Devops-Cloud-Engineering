# Ansible Refactoring & Static Assignments (Imports and Roles)


## Task Overview

In this project, you will continue working with the `ansible-config-mgt` repository to improve and refactor your Ansible code. You will create assignments and learn to use the imports functionality to organize and reuse tasks efficiently. Imports allow you to include previously created playbooks into new playbooks.

---

### Step 1 - Jenkins Job Enhancement

**Create a New Directory for Artifacts**

1. **Log in to your Jenkins-Ansible server via SSH:**

   ```sh
   ssh -i keypair.pem ubuntu@public-ip
   ```

2. **Create a new directory to store artifacts after each build:**

   ```sh
   sudo mkdir /home/ubuntu/ansible-config-artifact
   ```

3. **Change permissions to allow Jenkins to save, write, and execute files:**

   ```sh
   sudo chmod -R 777 /home/ubuntu/ansible-config-artifact
   ```

4. **Add Jenkins to the Ubuntu group:**

   ```sh
   sudo usermod -a -G jenkins ubuntu
   ```

5.  **Go to Jenkins web console:**
   - Navigate to **Manage Jenkins**.
   - Select **Manage Plugins**.

 **On the Available tab:**
   - Search for **Copy Artifact**.
   - Install this plugin without restarting Jenkins.

<img width="943" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/26cbcdd9-208f-41cb-9967-f992cc54816a">

6. Create a new Freestyle project and name it `save_artifacts`.
7. This project will be triggered by completion of your existing ansible project. Configure it accordingly

<img width="700" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/24772b08-69cb-4e35-91c0-81a42967f312">



**Note:**

You can configure the number of builds to keep in order to save space on the server. For example, you might want to keep only the last 2 or 5 build results. You can also make this change to your Ansible job.

<img width="746" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d2cbfaa1-a578-466c-a1ba-cf0efed3595e">


8. The main idea of the `save_artifacts` project is to save artifacts into the `/home/ubuntu/ansible-config-artifact` directory. To achieve this:

 **Create a Build step:**
   - Choose **Copy artifacts from other project**.
   - Specify **ansible** as the source project.
   - Set `/home/ubuntu/ansible-config-artifact` as the target directory.


<img width="889" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5869f816-ea65-4275-b1cd-97949edb8bb4">

9. Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside main branch).

If both Jenkins jobs have completed one after another - you shall see your files inside `/home/ubuntu/ansible-config-artifact` directory and it will be updated with every commit to your main branch. Now your Jenkins pipeline is more neat and clean.

```
Started by upstream project "ansible" build number 5
originally caused by:
 Started by GitHub push by MabelOlivia
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/save_artifacts
FATAL: /home/ubuntu/ansible-config-artifact
java.nio.file.AccessDeniedException: /home/ubuntu/ansible-config-artifact
	at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:90)
	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:106)
	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
	at java.base/sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:397)
	at java.base/java.nio.file.Files.createDirectory(Files.java:700)
	at java.base/java.nio.file.Files.createAndCheckIsDirectory(Files.java:807)
	at java.base/java.nio.file.Files.createDirectories(Files.java:793)
	at hudson.FilePath.mkdirs(FilePath.java:3752)
	at hudson.FilePath$Mkdirs.invoke(FilePath.java:1418)
	at hudson.FilePath$Mkdirs.invoke(FilePath.java:1413)
	at hudson.FilePath.act(FilePath.java:1234)
	at hudson.FilePath.act(FilePath.java:1217)
	at hudson.FilePath.mkdirs(FilePath.java:1408)
	at PluginClassLoader for copyartifact//hudson.plugins.copyartifact.CopyArtifact.copy(CopyArtifact.java:711)
	at PluginClassLoader for copyartifact//hudson.plugins.copyartifact.CopyArtifact.perform(CopyArtifact.java:675)
	at PluginClassLoader for copyartifact//hudson.plugins.copyartifact.CopyArtifact.perform(CopyArtifact.java:559)
	at jenkins.tasks.SimpleBuildStep.perform(SimpleBuildStep.java:123)
	at hudson.tasks.BuildStepCompatibilityLayer.perform(BuildStepCompatibilityLayer.java:80)
	at hudson.tasks.BuildStepMonitor$1.perform(BuildStepMonitor.java:20)
	at hudson.model.AbstractBuild$AbstractBuildExecution.perform(AbstractBuild.java:817)
	at hudson.model.Build$BuildExecution.build(Build.java:199)
	at hudson.model.Build$BuildExecution.doRun(Build.java:164)
	at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:526)
	at hudson.model.Run.execute(Run.java:1894)
	at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:44)
	at hudson.model.ResourceController.execute(ResourceController.java:101)
	at hudson.model.Executor.run(Executor.java:446)
Finished: FAILURE
```

The error message indicates that Jenkins is unable to access the directory `/home/ubuntu/ansible-config-artifact` due to permission issues. This typically happens when the Jenkins user does not have the necessary permissions to write to the specified directory.

Here are the steps to resolve this issue:

a. **Ensure Correct Permissions for the Directory:**
   - Log in to your Jenkins-Ansible server via SSH.

     ```sh
     ssh -i keypair.pem ubuntu@public-ip
     ```

   - Verify the permissions of the directory `/home/ubuntu/ansible-config-artifact`:

     ```sh
     ls -ld /home/ubuntu/ansible-config-artifact
     ```

   - If the directory does not have the correct permissions, set them using:

     ```sh
     sudo chmod -R 777 /home/ubuntu/ansible-config-artifact
     ```

b. **Verify Jenkins User Group Membership:**
   - Ensure that the Jenkins user is part of the `ubuntu` group:

     ```sh
     sudo usermod -a -G ubuntu jenkins
     ```

   - After adding the Jenkins user to the `ubuntu` group, restart the Jenkins service to apply the changes:

     ```sh
     sudo systemctl restart jenkins
     ```

c. **Check Directory Ownership:**
   - Change the ownership of the directory to the Jenkins user if necessary:

     ```sh
     sudo chown -R ubuntu:jenkins /home/ubuntu/ansible-config-artifact

     sudo groups ubuntu  # Confirm jenkins user have been added to ubuntu group
     ```


d. **Re-run the Jenkins Job:**
   - After making these changes, re-run your Jenkins job to see if the issue is resolved.

These steps should help in resolving the `AccessDeniedException` and allow Jenkins to access the directory successfully.

<img width="830" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/77535998-5da8-4114-af81-329a0895c860">

### Step 2 - Refactor Ansible code by importing other playbooks into site.yml
Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and create a new branch, name it refactor.

<img width="372" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/d58a33be-2d25-4ba5-be0e-f0241445e076">

<img width="322" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a5838cd2-17ec-4a22-a8c4-b2832c5fa39e"> <br>

<br>

**DevOps Philosophy and Refactoring**

DevOps philosophy implies constant iterative improvement for better efficiency. Refactoring is one of the techniques that can be used, but you always have to answer the question "why?". Why do we need to change something if it works well?

In a previous project, all tasks were written in a single playbook `common.yml`. Initially, this was a straightforward set of instructions for only 2 types of operating systems. However, imagine having many more tasks and needing to apply this playbook to servers with different requirements. In such cases, you would have to read through the entire playbook to check if all tasks are applicable and if anything needs to be added for specific server or OS families. This can quickly become a tedious exercise, leading to a messy playbook with many commented-out parts. Such organization may not be appreciated by your DevOps colleagues and could make the playbook difficult to use.

Let's see code re-use in action by importing other playbooks.

1. **Creating `site.yml` as an Entry Point**

   Within the `playbooks` folder, create a new file named `site.yml`. This file will serve as the entry point into the entire infrastructure configuration. Other playbooks will be included here as references. In other words, `site.yml` will become a parent to all other playbooks that will be developed, including `common.yml` that you created previously.

2. **Organizing Playbooks**

   Create a new folder in the root of the repository named `static-assignments`. This folder is where all other children playbooks will be stored. This organizational structure is for ease of work management. While not an Ansible-specific concept, it allows you to choose how you organize your work effectively. The prefix `static` in the folder name hints at its purpose, which will become clearer soon. For now, follow along with the organizational setup.

3. **Moving `common.yml`**

   Move the `common.yml` file into the newly created `static-assignments` folder.


This structured approach will help in organizing your Ansible playbooks effectively, making them more modular and easier to manage across different server configurations and requirements.

<img width="249" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8927f3ed-9f34-4743-ad38-ce9dfc31f96f">


4. **Inside `site.yml`, Import `common.yml` Playbook:**

```yaml
---
- hosts: all
  tasks:
  - name: Import common.yml playbook
    import_playbook: ../static-assignments/common.yml
```

The code above utilizes the built-in `import_playbook` Ansible module to include `common.yml`.

**Folder Structure:**

```
├── static-assignments
│   └── common.yml
├── inventory
│   ├── dev
│   ├── stage
│   ├── uat
│   └── prod
└── playbooks
    └── site.yml
```


**5. Run `ansible-playbook` Command Against the Dev Environment**

To apply tasks to your dev servers where Wireshark is already installed, proceed with creating another playbook under `static-assignments` named `common-del.yml`.

In this playbook, configure deletion of wireshark utility:

```yaml
---
- name: update web and nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB and DB servers
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes


```

<img width="620" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f2ba1e73-19f7-4fa3-84db-82e679bd0db5">


Update `site.yml` with - `import_playbook: ../static-assignments/common-del.yml`  instead of `common.yml`


```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
```

Run it against dev servers

```
cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```


<img width="660" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/23692796-afe5-45bf-b909-6cdc50a522bb">

Esure that wireshark is deleted on all the servers

Run `wireshark --version` to check

<img width="293" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/688e142a-5607-4aae-8615-2f0573100b46">

<img width="351" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/2722b9cc-55e9-4d75-983c-fcc9a109523a">


### Step 3 - Configure UAT Webservers with a Role

We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers for UAT. Instead of writing tasks to configure Web Servers in the same playbook (which would be too messy), we will use a dedicated role to make our configuration reusable.

#### 1. Launch EC2 Instances
Launch 2 fresh EC2 instances using the RHEL 9 image. These will be our UAT servers, so give them names accordingly:
- **Web1-UAT**
- **Web2-UAT**

<img width="785" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/82ffdcad-f778-4bf3-a636-69f4baa3ee5d">

#### 2. Create the Webserver Role

 **Create Role Directory Structure:**
To create a role, you must create a directory called `roles/`, either relative to the playbook file or in the `/etc/ansible/` directory. Here are the two ways you can create this folder structure:

#### Method 1: Using `ansible-galaxy`

 **Create the roles directory:**
   ```sh
   mkdir roles
   cd roles
   ```

 **Initialize the role:**
   ```sh
   ansible-galaxy init webserver
   ```

> **Note:** While you can choose either method, it is recommended to create folders and files in GitHub since you store all your code there, rather than locally on the Jenkins-Ansible server.

#### Method 2: Manually Creating the Folder Structure

You can manually create the folder structure if you prefer. If you create it manually, you can skip creating `tests`, `files`, and `vars` directories, or remove them if you used `ansible-galaxy`.

### Role Directory Structure

Your role directory structure should look like this:

```plaintext
ansible-config-mgt/
├── roles
│   └── webserver
│       ├── tasks
│       │   └── main.yml
│       ├── handlers
│       ├── templates
│       ├── files
│       ├── vars
│       ├── defaults
│       ├── meta
│       └── README.md
├── inventory
│   └── dev.yml
│   └── stage.yml
│   └── uat.yml
│   └── prod.yml
└── playbooks
    └── site.yml
    └── uat.yml
```

   ```sh
   mkdir roles
   cd roles
   mkdir -p webserver/{defaults,handlers,meta,tasks,templates}
   touch webserver/README.md
   touch webserver/defaults/main.yml
   touch webserver/handlers/main.yml
   touch webserver/meta/main.yml
   touch webserver/tasks/main.yml

   ```

After removing unnecessary directories and files, the roles structure should look like this

```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```

<img width="244" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e358757b-cbb3-40c2-b0b6-8c66a142298b">


#### 3. Update Inventory 
Ensure your inventory file `ansible-config-mgt/inventory/uat.yml` file is updated with IP addresses of your 2 UAT Web servers

NOTE: Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance

```ini
# inventory/uat.yml
[webservers_uat]
Web1-UAT ansible_host=<Web1-UAT_IP> ansible_user=ec2-user
Web2-UAT ansible_host=<Web2-UAT_IP> ansible_user=ec2-user

or

[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```

<img width="475" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8602d71a-e0d7-46b1-bc4d-8ac00ebd263a">

#### 4. 
In `/etc/ansible/ansible.cfg` file uncomment roles_path string and provide a full path to your roles directory `roles_path = /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.

<img width="487" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/11c62312-f0fc-484c-aaaf-3b9b4a543453">


#### 5. 
It is time to start adding some logic to the `webserver` role. Go into the `tasks` directory, and within the `main.yml` file, start writing configuration tasks to do the following:

- Install and configure Apache (`httpd` service)
- Clone the Tooling website from GitHub: [https://github.com//tooling.git](https://github.com//tooling.git)
- Ensure the Tooling website code is deployed to `/var/www/html` on each of the 2 UAT Web servers
- Make sure the `httpd` service is started

Your `main.yml` consists of the following tasks:

```yaml
# webserver/tasks/main.yml
# tasks file for webserver
---
- name: install apache
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.git:
    repo: https://github.com/MabelOlivia/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  remote_user: ec2-user
  become: true
  become_user: root
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

### Step 4 - Reference Webserver role

Within the static-assignments folder, create a new assignment for uat-webservers `uat-webservers.yml`. This is where you will reference the role.

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

<img width="796" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dce15f0b-f2b5-4dbc-a11a-46e1251b25b5">


Remember that the entry point to our ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`.

So, we should have this in `site.yml`

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

<img width="531" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/16b51470-70d0-4b95-97ed-6c15e03a3b26">

### Step 5 - Commit & Test

Commit your changes, create a Pull Request and merge them to main branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your `Jenkins-Ansible` server into `/home/ubuntu/ansible-config-artifact/` directory.

<img width="932" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/51c5861d-e5ef-4bd6-8d58-96c887c450e6">

<img width="440" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/470f48d6-611f-4bf4-b9d8-0b4eacd063ae">


Now run the playbook against your uat inventory and see what happens:

#### NOTE: Before running your playbook, ensure you have tunneled into your Jenkins-Ansible server via ssh-agent

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
```


```
cd /home/ubuntu/ansible-config-artifact

ansible-playbook -i /inventory/uat.yml playbooks/site.yml
or
ansible-playbook -i ~/ansible-config-mgt/inventory/uat.yml playbooks/site.yml

```

<img width="720" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/53478a5d-df34-4f8c-95ab-7aa654398ad7"> <br>

<br>

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:


```
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

or

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

```

Access Web1-UAT

<img width="940" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/42dfe864-d6e6-441b-ae29-3d95a87b8253">

Access Web2-UAT

<img width="940" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/70c6c308-f00f-4658-82f9-4ae78abe009f"> <br>

<br>

Our Ansible architecture now looks like this:

<img width="850" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0cae5c0e-bea2-43ab-9fa3-0285ec861bb4">

