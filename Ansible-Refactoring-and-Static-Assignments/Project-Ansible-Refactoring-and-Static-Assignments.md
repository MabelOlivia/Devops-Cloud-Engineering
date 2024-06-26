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


