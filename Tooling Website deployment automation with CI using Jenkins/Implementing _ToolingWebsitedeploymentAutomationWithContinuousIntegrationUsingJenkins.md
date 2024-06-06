# Tooling Website deployment Automation with Continuous Integration using Jenkins

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools.

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments which is then automatically built and tested before it is merged with the shared repository.

### Task

Enhance the architecture prepared in the previous project by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how the updated architecture looks.

<img width="495" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/b838a487-7a53-451f-b495-9586cca42468">



## Step 1 - Install Jenkins server

1. Create an AWS EC2 instance based on Ubuntu Server 24.04 LTS and name it Jenkins

<img width="734" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dff8fa77-99d6-4872-9689-66a2104def50">

2. Install JDK since Jenkins is a Java-based application





