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

   

