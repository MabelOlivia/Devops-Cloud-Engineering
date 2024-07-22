# Continuous Integration with Jenkins, Ansible, Artifactory, SonarQube, and PHP

In this project, we'll get hands-on with CI/CD using a PHP application. We'll expand to other languages like Java, Node.js, .NET, and Python later. By the time we reach Terraform, Docker, and Kubernetes, you'll grasp the platform side of CI/CD.

## 13 Key DevOps Metrics

1. **Deployment Frequency:** Track deployment frequency. Aim for small, frequent deployments to ease testing and releases. Include both production and non-production deployments.

2. **Lead Time:** Measure the time from starting a work item to its deployment. This shows how quickly code reaches production.

3. **Customer Tickets:** Customer support tickets are a clear indicator of application issues. They reflect the quality and performance problems of the application.

4. **Percentage of Passed Automated Tests:** High usage of unit and functional testing is recommended. This metric shows how often code changes break tests.

5. **Defect Escape Rate:** Track how many defects reach production versus QA. This helps ensure defects are found before production.

6. **Availability:** Track both planned and unplanned downtimes. Availability metrics help ensure the application is consistently up.

7. **Service Level Agreements (SLAs):** Track compliance with SLAs or non-functional requirements, even if they are not formally stated.

8. **Failed Deployments:** Track how often deployments cause outages or major issues. This metric also includes the mean time to failure (MTTF).

9. **Error Rates:** Track error rates to identify quality problems and performance issues. Proper exception handling and monitoring error rates are crucial.

10. **Application Usage & Traffic:** Monitor transaction and user activity post-deployment to detect anomalies like traffic spikes or drops.

11. **Application Performance:** Use tools like Retrace, DataDog, New Relic, or AppDynamics to monitor performance and spot issues.

12. **Mean Time to Detection (MTTD):** Quickly identify problems to minimize system outages. Robust monitoring and observability tools are essential.

13. **Mean Time to Recovery (MTTR):** Track how long it takes to recover from failures. Minimizing recovery time is key for business continuity.

These metrics help DevOps teams monitor and improve the CI/CD process, ensuring quality applications.

## Simulating a CI/CD Pipeline for a PHP Application

We'll create a CI/CD pipeline for a PHP application, using tools like Ansible for infrastructure setup.

### Setup

Initially, we'll focus on these environments:
- CI
- Dev
- Pentest

We'll use Nginx as a reverse proxy for our sites and tools.

### CI/CD Best Practices

- Maintain a code repository
- Automate the build process
- Ensure builds are self-tested
- Commit to the baseline daily
- Build every baseline commit
- Include test cases for every bug-fix commit
- Keep the build fast
- Test in a production-like environment
- Make deliverables easily accessible
- Make build results visible
- Automate deployment for a fully automated CI/CD pipeline

<img width="324" alt="image" src="https://github.com/user-attachments/assets/5416a85e-36f7-4389-a53e-8f0b33766bf5">


## Project Description

We'll set up a CI/CD pipeline for a PHP-based application, involving two main repositories:

- **ansible-config-mgt REPO:** Contains Jenkinsfile for infrastructure setup using Ansible roles.
- **PHP-todo REPO:** Contains Jenkinsfile for application-specific processes like building, linting, static code analysis, and artifact storage.

## Pre-requisites

We will use AWS virtual machines and require six servers:

- **Nginx Server:** Acts as the reverse proxy.
- **Jenkins Server:** For CI/CD workflows (t2.medium, Ubuntu 20.04, port 8080).
- **SonarQube Server:** For code quality analysis (t2.medium, Ubuntu 20.04, port 9000).
- **Artifactory Server:** For storing build artifacts (t2.medium, port 8081).
- **Database Server:** For the Todo application database.
- **Todo Webserver:** Hosts the Todo web application.

## DNS Configuration
  - Domain and Subdomains:
    - Set up DNS entries for subdomains using your main domain `mightygull.click`.

    | Server                 |            Domain                              |
    |------------------------|------------------------------------------------|
    | Jenkins                | https://ci.infradev.mightygull.click           |
    | Sonarqube              | https://sonar.infradev.mightygull.click        |
    | Artifactory            | https://artifacts.infradev.mightygull.click    |
    | Production Tooling     | https://tooling.mightygull.click               |
    | Pre-Prod Tooling       | https://tooling.preprod.mightygull.click       |
    | Pentest Tooling        | https://tooling.pentest.mightygull.click       |
    | UAT Tooling            | https://tooling.uat.mightygull.click           |
    | SIT Tooling            | https://tooling.sit.mightygull.click           |
    | Dev Tooling            | https://tooling.dev.mightygull.click           |
    | Production TODO-WebApp | https://todo.mightygull.click                  |
    | Pre-Prod TODO-WebApp   | https://todo.preprod.mightygull.click          |
    | Pentest TODO-WebApp    | https://todo.pentest.mightygull.click          |
    | UAT TODO-WebApp        | https://todo.uat.mightygull.click              |
    | SIT TODO-WebApp        | https://todo.sit.mightygull.click              |
    | Dev TODO-WebApp        | https://todo.dev.mightygull.click              |
	    

## Ansible Inventory Setup

Inventory structure:
```css
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

Example inventory files:
```css
ci:
[jenkins]
<Jenkins-Private-IP>

[nginx]
<Nginx-Private-IP>

[sonarqube]
<SonarQube-Private-IP>

[artifact_repository]
<Artifact-Repo-Private-IP>

dev:
[tooling]
<Tooling-Web-Server-Private-IP>

[todo]
<Todo-Web-Server-Private-IP>

[nginx]
<Nginx-Private-IP>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP>

pentest:
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-Todo-Private-IP>

[pentest-tooling]
<Pentest-Tooling-Private-IP>
```

Observations:
- Use `pentest:children` for grouping.
- Set group-specific variables with `group_vars`.

## Install Jenkins

Launch an AWS EC2 instance with Ubuntu and configure Jenkins:
```bash
sudo apt-get update
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt install openjdk-11-jdk
sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Open TCP port 8080 for Jenkins.

## Install Blue Ocean Plugin

Install Blue Ocean for better visualization of CD pipelines:
- Go to Manage Jenkins > Manage Plugins > Available
- Search and install Blue Ocean Plugin

<img width="530" alt="image" src="https://github.com/user-attachments/assets/094a7104-67ff-4064-bce5-55226d1025a5">


## Configure Blue Ocean Pipeline with GitHub

- Open Blue Ocean plugin and create a new pipeline.
- Select GitHub and connect using your personal access token.
- Select the repository and create the pipeline.

<img width="527" alt="image" src="https://github.com/user-attachments/assets/59371094-e9d8-4d2e-bd6e-cace0186eb65">

<img width="532" alt="image" src="https://github.com/user-attachments/assets/9015fcf1-19be-4aa7-9ce1-5dfba1026f65">

<img width="514" alt="image" src="https://github.com/user-attachments/assets/2f988765-d30e-4a01-bb73-cfb89dc0f886">

<img width="481" alt="image" src="https://github.com/user-attachments/assets/6b8c328a-5583-49cf-87ba-9bf303feaf2f">

<img width="533" alt="image" src="https://github.com/user-attachments/assets/92ee01ad-0fb9-4a28-98d3-81202775fb80">






## Create `Jenkinsfile`

Create a `Jenkinsfile` in the `deploy` directory:
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'echo "Building Stage"'
                }
            }
        }
    }
}
```
Update the pipeline configuration in Jenkins and build.

<img width="531" alt="image" src="https://github.com/user-attachments/assets/90a08805-51b4-4868-8d21-d6c9cc9040e0">

<img width="535" alt="image" src="https://github.com/user-attachments/assets/ee8421d9-c92d-47ae-b821-42f23783d5df">

<img width="535" alt="image" src="https://github.com/user-attachments/assets/de559937-c015-48d4-9961-e5111f3f9072">





## Running Ansible Playbook from Jenkins

Install Ansible on Jenkins:
```bash
sudo apt update
sudo apt install ansible
ansible --version
```
Install the Ansible plugin in Jenkins.

### Create `Jenkinsfile` for Ansible

```groovy
pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/deploy/ansible.cfg"
    }

    stages {
        stage('Initial cleanup') {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/francdomain/ansible-config-mgt.git'
            }
        }

        stage('Prepare Ansible For Execution') {
            steps {
                sh 'echo ${WORKSPACE}'
                sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'
            }
        }

        stage('Test SSH Connections') {
            steps {
                script {
                    def allHosts = [
                        'ubuntu@172.31.89.253',
                        'ubuntu@172.31.91.227',
                        'ec2-user@172.31.81.236',
                        'ec2-user@172.31.85.118'
                    ]

                    sshagent(['private-key']) {
                        allHosts.each { host ->
                            sh "ssh -o StrictHostKeyChecking=no ${host} exit"
                        }
                    }
                }
            }
        }

        stage('Run Ansible playbook') {
            steps {
                sshagent(['private-key']) {
                    ansiblePlaybook(
                        become: true,
                        credentialsId: 'private-key',
                        disableHostKeyChecking: true,
                        installation: 'ansible',
                        inventory: '${WORKSPACE}/inventory/dev.yml',
                        playbook: '${WORKSPACE}/playbooks/site.yml'
                    )
                }
            }
        }

        stage('Clean Workspace after build') {
            steps {
                cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
            }
        }
    }
}
```

<img width="500" alt="image" src="https://github.com/user-attachments/assets/f8e60388-9e35-47d0-ab00-4bae819a6442">

<img width="500" alt="image" src="https://github.com/user-attachments/assets/e07175cb-8a87-4f25-a5fa-2645e872c18f">

<img width="498" alt="image" src="https://github.com/user-attachments/assets/ba7569fe-9a62-4cd5-ae5d-15d0438a904c">

<img width="517" alt="image" src="https://github.com/user-attachments/assets/e6c029c7-f495-4d7d-b255-92b0d06a016b">




## Troubleshooting

1. Ensure the git branch in the Jenkinsfile matches your repository.
2. Export `ANSIBLE_CONFIG` properly to point to the `ansible.cfg`.
3. Clean workspace before builds to avoid outdated files.
4. Verify Jenkins settings and server files for persistent issues.

## Install Ansible


## Configure ansible on Jenkins

Click on Dashboard > Manage Jenkins > Global Tool Configuration > Add Ansible. Add a name and the path ansible is installed on the jenkins server.


### To ensure jenkins properly connects to all servers, install another plugin called `ssh agent`



### Then go to `manage jenkins > credentials > global > add credentials`
Then follow the steps below:

- Kind: SSH Username with private key
- Scope: Global (Jenkins, nodes, items, all child items, etc)
-	ID: private-key (or any ID you prefer)
-	Username: Leave it blank or set a default value (e.g., defaultuser) # This is because we are using servers of different username i.e ubbuntu and ec2-user. This value won’t be used because the actual usernames will be specified in the Ansible inventory file.
-	Private Key: Enter the private key directly




## Update inventory/dev.yml by specifying the private IP address of the servers



### Update the playbook as well



## Run Ansible against the Dev environment


<img width="571" alt="image" src="https://github.com/user-attachments/assets/dd1c0fce-5ffc-4b77-b963-4203c2f4cccb">


# Parameterizing the Jenkinsfile for Ansible Deployment

To deploy to various environments, we need to parameterize the Jenkinsfile.

1. **Update `sit` Inventory with New Servers**

Modify the `inventory/sit.yml` file with the new server details:

```yaml
[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>
```

<img width="404" alt="image" src="https://github.com/user-attachments/assets/0bbc6ff8-b4f5-4d96-ba86-34db59172a36">


2. **Introduce Parameterization in Jenkinsfile**

Add parameters to your Jenkinsfile. Here’s an example with one parameter for the inventory file:

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'inventory', defaultValue: 'dev', description: 'This is the inventory file for the environment to deploy configuration')
    }
    ...
}
```

3. **Update Ansible Execution Section**

Replace hardcoded `inventory/dev` with `${inventory}`:

- Notice that `Build Now` is now `Build with Parameters`, allowing you to specify the environment for deployment. The default value will be used if no other value is specified. Simply type `sit` to run the build.

  <img width="569" alt="image" src="https://github.com/user-attachments/assets/994ceac6-a4b3-42db-ad85-5567075776e6">


- Check the results in Blue Ocean.

  <img width="554" alt="image" src="https://github.com/user-attachments/assets/117eacdb-b86f-4a45-a5b2-997dcbeeb28b">


4. **Add Parameter for Ansible Tags**

Introduce a parameter for tagging in Ansible to run specific roles or playbooks. For example, add a tag to run against `webserver` only:

- Update `playbook/site.yml` to include tags.

- Add the `ansible_tags` parameter to the Jenkinsfile:

```groovy
string(name: 'ansible_tags', defaultValue: 'webserver', description: 'Ansible tags to run specific roles or tasks')
```

- Modify the Ansible execution section to use the tags parameter.

- Update the inventory and `ansible_tags` fields and click `Run` to execute the build.

<img width="571" alt="image" src="https://github.com/user-attachments/assets/193d650e-cd85-465a-9666-894a1c102acf">

<img width="570" alt="image" src="https://github.com/user-attachments/assets/54cc60dd-d7f5-45b8-9dcc-8fa81a6b86e5">

<img width="570" alt="image" src="https://github.com/user-attachments/assets/42526a54-c860-49a9-bd2e-48d2fe92cb1e">




### Additional Parameterization Suggestions

To avoid hardcoding values in the Jenkinsfile, consider parameterizing the following elements:

1. **SCM URL and Branch**:
   - Parameterize the repository URL and branch to change them without modifying the Jenkinsfile.

2. **SSH Hosts**:
   - Parameterize the SSH hosts list for different environments or scenarios.

3. **Ansible Playbook Path**:
   - Parameterize the path to the Ansible playbook to allow flexibility.

4. **Credentials ID**:
   - Parameterize the credentials ID for SSH and Ansible to support different credentials.

5. **Role Path**:
   - Parameterize the roles path in the Ansible configuration.

Example Jenkinsfile with additional parameters:

```groovy
pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/deploy/ansible.cfg"
    }

    parameters {
        string(name: 'inventory', defaultValue: 'dev', description: 'This is the inventory file for the environment to deploy configuration')
        string(name: 'branch', defaultValue: 'main', description: 'Branch to checkout')
        string(name: 'repo_url', defaultValue: 'https://github.com/francdomain/ansible-config-mgt.git', description: 'Repository URL')
        text(name: 'ssh_hosts', defaultValue: 'ubuntu@172.31.89.253\nubuntu@172.31.91.227\nec2-user@172.31.81.236\nec2-user@172.31.85.118', description: 'List of SSH hosts, one per line')
        string(name: 'playbook_path', defaultValue: '${WORKSPACE}/playbooks/site.yml', description: 'Path to the Ansible playbook')
        string(name: 'credentials_id', defaultValue: 'private-key', description: 'Credentials ID for SSH and Ansible')
    }

    stages {
        stage('Initial Cleanup') {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: "${params.branch}", url: "${params.repo_url}"
            }
        }

        stage('Prepare Ansible For Execution') {
            steps {
                sh 'echo ${WORKSPACE}'
                sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'
            }
        }

        stage('Test SSH Connections') {
            steps {
                script {
                    def allHosts = params.ssh_hosts.split('\n')

                    sshagent([params.credentials_id]) {
                        allHosts.each { host ->
                            sh "ssh -o StrictHostKeyChecking=no ${host} exit"
                        }
                    }
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sshagent([params.credentials_id]) {
                    ansiblePlaybook(
                        become: true,
                        credentialsId: params.credentials_id,
                        disableHostKeyChecking: true,
                        installation: 'ansible',
                        inventory: "${WORKSPACE}/inventory/${params.inventory}",
                        playbook: params.playbook_path
                    )
                }
            }
        }

        stage('Clean Workspace After Build') {
            steps {
                cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
            }
        }
    }
}
```

<img width="378" alt="image" src="https://github.com/user-attachments/assets/e7e0bb93-325f-4063-8e14-b43524e56c25">



# CI/CD Pipline for TODO Application

We already have `tooling` website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

### Todo Project files path
```css
/path/to/your/laravel/project
├── app
│   ├── Console
│   ├── Exceptions
│   ├── Http
│   │   ├── Controllers
│   │   ├── Middleware
│   ├── Models
│   ├── Providers
├── bootstrap
│   ├── cache
├── config
│   ├── app.php
│   ├── database.php
│   └── ...
├── database
│   ├── factories
│   ├── migrations
│   ├── seeders
├── public
│   ├── index.php
│   ├── css
│   ├── js
│   ├── ...
├── resources
│   ├── js
│   ├── lang
│   ├── views
│   └── ...
├── routes
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   ├── web.php
├── storage
│   ├── app
│   ├── framework
│   ├── logs
├── tests
│   ├── Feature
│   ├── Unit
├── vendor
├── .env
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── phpunit.xml
└── webpack.mix.js
```

Our goal here is to deploy the application onto servers directly from `Artifactory` rather than from `git` `If you have not updated Ansible with an Artifactory role`, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). [Configure Artifactory on Ubuntu 20.04](https://www.atlantic.net/dedicated-server-hosting/how-to-install-jfrog-artifactory-on-ubuntu-22-04/)

## Create an Ansible role for Artifactory

### Prerequests

- Ensure port 8082 is opened in artifactory server

### Install Artifactory role using Ansible galaxy collection

```
ansible-galaxy collection install jfrog.platform
```



### Update Artifactory role in `roles/artifactory/tasks/main.yml` to install jfrog Artifactory


- Update `playbook/site.yml`



- Update `inventory/ci.yml`



### Run the playbook against ci.yml to install `jfrog artifactory`

<img width="570" alt="image" src="https://github.com/user-attachments/assets/7ba48d39-50b2-4d80-a60b-d666c5ed5345">


Access the artifactory GUI on a browser with `http://<server public IP address>:8082`. Use the default authentication credentials: `admin` and `password` to login.

<img width="570" alt="image" src="https://github.com/user-attachments/assets/18cb844c-a908-4e01-a44e-2cdfac30b529">

<img width="566" alt="image" src="https://github.com/user-attachments/assets/aa8b9f87-c072-4873-8d3a-f62f1556d0fe">



Create a local repository `Todo-artifact-local`

<img width="570" alt="image" src="https://github.com/user-attachments/assets/2d2e3221-de01-469b-b2c8-607e8df8e984">





# Phase 1 – Prepare Jenkins

## 1. Fork the repository below into your GitHub account

```bash
https://github.com/StegTechHub/php-todo.git
```


## 2. On your Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)

```bash
sudo apt update

# Install dependancies
sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}

# Download the installer to the current directory
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Install Composer Globally
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# Remove the Installer
php -r "unlink('composer-setup.php');"

# Verify Installation version
php -v
composer -v
```


The `php version` installed by the composer is `8.3.6` while the php version of the `todo application` is `7.4`. Ensure to remove the current version and [install php 7.4](https://www.digitalocean.com/community/questions/how-to-update-the-php-to-7-4) and its dependencies to avoid error.

## 3. Install Jenkins plugins

- Plot plugin and Artifactory plugin

<img width="569" alt="image" src="https://github.com/user-attachments/assets/24a91485-0041-4467-ab23-08735a1dae3c">


<img width="572" alt="image" src="https://github.com/user-attachments/assets/b5f2f9a5-bf31-4b63-ac96-7151a64e3f0b">


- We will use plot plugin to display tests reports, and code coverage information.
- The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.

## 4. In Jenkins UI configure Artifactory

- Configure the server ID, URL and Credentials, run Test Connection.

<img width="570" alt="image" src="https://github.com/user-attachments/assets/0199aa59-816e-4c65-9d4b-38ccf0355720">



# Phase 2 - Integrate Artifactory repository with Jenkins

1. Create a dummy Jenkinsfile in the repository
2. Using Blue Ocean, create a multibranch Jenkins pipeline



## 3. On the database server, create database and user
   - In jenkins server Install my sql client

```bash
sudo apt install mysql-client -y
```
  - Create database and user on database server

```sql
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```
- Update mysql role in `roles/mysql/vars/main.yml`



- Run ansible with jenkins to create the database



- Verify that database has benn created



## 4. Update the database connectivity requirements in the file .env.sample

```css
APP_ENV=local
APP_DEBUG=true
LOG_LEVEL=debug
APP_KEY=SomeRandomString
APP_URL=http://localhost

DB_HOST=172.31.91.227
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=sePret^i
DB_CONNECTION=mysql
BD_PORT=3306

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

## 5. Update Jenkinsfile with proper pipeline configuration

```groovy
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/StegTechHub/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```

__Notice__ the `Prepare Dependencies` section

- The required file by PHP is `.env` so we are renaming .env.sample to .env
- Composer is used by PHP to install all the dependent libraries used by the application
- `php artisan` uses the `.env` file to setup the required database objects - (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)

- Update the Jenkinsfile to include Unit tests step

  - Ensure that all neccesary php extensions are already installed.
  - Run the pipeline build , you will notice that the database has been populated with tables using a method in laravel known as `migration` and `seeding`.

### Install `phpunit`



### Install `phploc`


- __Update Jenkinsfile to include Unit tests__

```groovy
stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      }
```
- Run the pipieline




- __After successful run, login to the database__.

```sql
sudo mysql
```
- __Run show tables and you will see the tables being created__.



- Set the bind address



# Phase 3 - Code Quality Analysis

This is one of the areas where developers, architects and many stakeholders are mostly interested in as far as product development is concerned. As a DevOps engineer, you also have a role to play. Especially when it comes to setting up the tools.

For PHP the most commonly tool used for code quality analysis is [phploc](https://phpqa.io/projects/phploc.html). [Read the article here for more](https://matthiasnoback.nl/2019/09/using-phploc-for-quick-code-quality-estimation-part-1/)

The data produced by `phploc` can be ploted onto graphs in Jenkins.

1. Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.

```groovy
stage('Code Analysis') {
      steps {
            sh 'phploc app/ --log-csv build/logs/phploc.csv'

      }
    }
```
2. Plot the data using plot Jenkins plugin.
This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots' data series latest values are pulled from the CSV file generated by phploc.



- Run the pipeline and View the Plot chart in Jenkins



### 3. Bundle the application code for into an artifact (archived package) upload to Artifactory

```groovy
stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
```
### 4. Publish the resulted artifact into Artifactory

```groovy
stage ('Upload Artifact to Artifactory') {
          steps {
            script {
                 def server = Artifactory.server 'artifactory-server'
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }"""

                 server.upload spec: uploadSpec
               }
            }

        }
```
### 5. Deploy the application to the dev environment by launching Ansible pipeline

```groovy
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
```


### Create a task to set up the Dev environment and deploy artifact to webserver (todo server)

```yaml
# # tasks file for webserver - php-todo
---
# Install Apache
- name: Install Apache
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name: httpd
    state: present

# Install EPEL release
- name: Install EPEL release
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name: https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    state: present

# Install dnf-utils and Remi repository
- name: Install dnf-utils and Remi repository
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name:
      - dnf-utils
      - http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    state: present

# Reset PHP module
- name: Reset PHP module
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: dnf module reset php -y

# Enable PHP 7.4 module
- name: Enable PHP 7.4 module
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: dnf module enable php:remi-7.4 -y

# Install PHP and extensions
- name: Install PHP and extensions
  remote_user: ec2-user
  become: true
  ansible.builtin.yum:
    name:
      - php
      - php-opcache
      - php-gd
      - php-curl
      - php-mysqlnd
      - php-common
      - php-mbstring
      - php-intl
      - php-xml
      - php-fpm
      - php-json
    state: present
    enablerepo: remi-7.4

# Ensure PHP-FPM service is started and enabled
- name: Ensure PHP-FPM service is started and enabled
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: php-fpm
    state: started
    enabled: true

# Set SELinux boolean for httpd_execmem
- name: Set SELinux boolean for httpd_execmem
  remote_user: ec2-user
  become: true
  ansible.builtin.seboolean:
    name: httpd_execmem
    state: true
    persistent: yes

# Ensure httpd service is started and enabled
- name: Ensure httpd service is started and enabled
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

# Download the artifact
- name: Download the artifact
  remote_user: ec2-user
  become: true
  ansible.builtin.get_url:
    url: http://54.88.110.0:8082/artifactory/Todo-artifact-local/php-todo.zip
    dest: /home/ec2-user/php-todo.zip
    url_username: admin
    url_password: Admin123$

- name: Ensure unzip is installed
  become: yes
  ansible.builtin.yum:
    name: unzip
    state: present
  when: ansible_os_family == "RedHat"

- name: Unzip the artifacts
  ansible.builtin.unarchive:
    src: /home/ec2-user/php-todo.zip
    dest: /var/www/html/
    remote_src: yes

# Unzip the artifacts
- name: Unzip the artifacts
  ansible.builtin.unarchive:
    src: /home/ec2-user/php-todo.zip
    dest: /home/ec2-user/
    remote_src: yes
  become: true

# Run chown on /var/www/html
- name: Run chown on /var/www/html
  ansible.builtin.command:
    cmd: chown -R $USER:$USER /var/www/html
  become: true

# Deploy the code
- name: Deploy the code
  remote_user: ec2-user
  become: true
  ansible.builtin.copy:
    src: /home/ec2-user/var/lib/jenkins/workspace/php-todo-pipeline/.
    dest: /var/www/html/
    force: yes
    remote_src: yes

# Delete the zip file after unzipping
- name: Delete the zip file after unzipping
  ansible.builtin.file:
    path: /home/ec2-user/php-todo.zip
    state: absent
  become: true

# Delete the unzipped files after copying
- name: Delete the unzipped files after copying
  ansible.builtin.file:
    path: /home/ec2-user/php-todo/
    state: absent
  become: true

# Remove Apache default welcome page on RedHat-based systems
- name: Remove Apache default welcome page
  remote_user: ec2-user
  become: true
  ansible.builtin.file:
    path: /etc/httpd/conf.d/welcome.conf
    state: absent
  when: ansible_os_family == "RedHat"

# Restart httpd
- name: Restart httpd
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: httpd
    state: restarted

# Verify PHP version
- name: Verify PHP version
  remote_user: ec2-user
  become: true
  ansible.builtin.command: php -v
  register: php_version

# Display PHP version
- name: Display PHP version
  remote_user: ec2-user
  become: true
  ansible.builtin.debug:
    msg: "{{ php_version.stdout }}"

# Set SELinux context for web directory and PHP files
- name: Set SELinux context for web directory
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chcon -R -t httpd_sys_content_t /var/www/html

- name: Set SELinux context for PHP writable files
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chcon -R -t httpd_sys_rw_content_t /var/www/html

# Generate a new APP_KEY if missing or invalid
- name: Generate a new APP_KEY if missing or invalid
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: php artisan key:generate
  args:
    chdir: /var/www/html

# Clear the configuration cache
- name: Clear the configuration cache
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: php artisan config:cache
  args:
    chdir: /var/www/html

# Ensure .env file and config directory have correct permissions
- name: Ensure .env file and config directory have correct permissions - chown
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chown -R apache:apache /var/www/html/

- name: Ensure .env file and config directory have correct permissions - chmod
  remote_user: ec2-user
  become: true
  ansible.builtin.command:
    cmd: chmod -R 775 /var/www/html/

# Restart the web server and PHP-FPM
- name: Restart the web server
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: httpd
    state: restarted

- name: Restart PHP-FPM
  remote_user: ec2-user
  become: true
  ansible.builtin.service:
    name: php-fpm
    state: restarted
```


- Run the pipeline



- Check the artifactory for the uploaded artifact (`php-todo.zip`)


### Visit a browser using the publis IP address to access the todo application



### Configuring SonarQube: An Open-Source Platform for Continuous Code Quality Inspection

SonarQube, developed by SonarSource, is a powerful tool for continuously reviewing code quality through static code analysis to identify bugs, code smells, and security vulnerabilities.

#### SonarQube Installation

##### Setting Up SonarQube on Ubuntu 20.04 with PostgreSQL

Follow this manual installation guide for SonarQube 7.9.3, which requires Java and uses PostgreSQL as MySQL support is deprecated. We will also configure Linux kernel settings to optimize performance.

- **Tune Linux Kernel**

Temporary session changes can be made with:

```bash
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

To make these changes permanent, edit `/etc/security/limits.conf`:

```bash
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```

- **Update and Upgrade System Packages**

```bash
sudo apt-get update
sudo apt-get upgrade
```

- **Install wget and unzip**

```bash
sudo apt-get install wget unzip -y
```

- **Install OpenJDK and Java Runtime Environment (JRE) 11**

```bash
sudo apt-get install openjdk-11-jdk -y
sudo apt-get install openjdk-11-jre -y
```

- **Set Default JDK**

To set or switch to OpenJDK, use:

```bash
sudo update-alternatives --config java
```

Select OpenJDK 11 from the list.

- **Verify JAVA Version**

```bash
java -version
```

##### Installing and Setting Up PostgreSQL 10 for SonarQube

- **Add PostgreSQL Repository**

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
```

- **Download PostgreSQL Key**

```bash
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```

- **Start PostgreSQL Server**

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

- **Change Password for Default User**

```bash
su - postgres
```

- **Create New User**

```sql
createuser sonar
```

- **Switch to PostgreSQL Shell**

```sql
psql
```

- **Set Password for SonarQube User**

```sql
ALTER USER sonar WITH ENCRYPTED password 'sonar';
```

- **Create SonarQube Database**

```sql
CREATE DATABASE sonarqube OWNER sonar;
```

- **Grant Privileges to Sonar User**

```sql
grant all privileges on DATABASE sonarqube to sonar;
```

- **Exit PostgreSQL Shell**

```sql
\q
```

##### Installing SonarQube on Ubuntu 20.04 LTS

- **Download SonarQube**

```bash
cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip
```

- **Unzip and Move SonarQube**

```bash
sudo unzip sonarqube-7.9.3.zip -d /opt
sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube
```

##### Configuring SonarQube

SonarQube cannot run as the root user. Create a separate user and group for it:

- **Create SonarQube Group**

```bash
sudo groupadd sonar
```

- **Add User and Set Permissions**

```bash
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```

- **Edit SonarQube Configuration**

Open the configuration file:

```bash
sudo vim /opt/sonarqube/conf/sonar.properties
```

Uncomment and set the PostgreSQL username and password.

- **Set RUN_AS_USER in Sonar Script**

Edit the sonar script:

```bash
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
```

##### Starting SonarQube

- **Switch to Sonar User**

```bash
sudo su sonar
```

- **Navigate to Script Directory**

```bash
cd /opt/sonarqube/bin/linux-x86-64/
```

- **Start SonarQube**

```bash
./sonar.sh start
```

- **Verify Status**

```bash
$./sonar.sh status
```

- **Check Logs**

```bash
tail /opt/sonarqube/logs/sonar.log
```

##### Configuring SonarQube as a Systemd Service

- **Stop SonarQube Service**

```bash
cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh stop
```

- **Create Systemd Service File**

```bash
sudo nano /etc/systemd/system/sonar.service
```

Add the following configuration:

```yaml
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

- **Start and Enable Service**

```bash
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
```

- **Configure Port**

Edit the SonarQube configuration:

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

Uncomment `sonar.web.port=9000`.

##### Accessing SonarQube

Open your browser and visit:

```bash
http://server_IP:9000 OR http://localhost:9000
```

Log in with the default credentials: username `admin`, password `admin`.

#### Automate SonarQube and PostgreSQL Setup with Ansible

- **Update Inventory and Playbooks**

Update `inventory/ci.yml`, `playbook/site.yml`, `roles/sonar/tasks/main.yml`, `roles/sonar/tasks/postgresql.yml`, and `roles/sonar/tasks/sonarqube-setup.yml`.

- **Run the Playbook**

Execute the playbook against `ci.yml`.

#### Configure SonarQube and Jenkins for Quality Gate

- **Install SonarScanner Plugin**

Add SonarQube server in Jenkins: `Manage Jenkins > Configure System`.

- **Generate Authentication Token**

In SonarQube: `User > My Account > Security > Generate Tokens`.

- **Configure Quality Gate Webhook**

In SonarQube: `Administration > Configuration > Webhooks > Create`, set URL to `http://{JENKINS_HOST}/sonarqube-webhook/`.

- **Setup SonarQube Scanner in Jenkins**

Add configuration in Jenkins Global Tool Configuration.

#### Update Jenkins Pipeline for SonarQube Scanning and Quality Gate

Add a `Quality Gate` stage in your `Jenkinsfile`:

```groovy
stage('SonarQube Quality Gate') {
    when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP" }
    environment {
        scannerHome = tool 'SonarQubeScanner'
    }
    steps {
        withSonarQubeEnv('sonarqube') {
            sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
        }
        timeout(time: 1, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

Configure `sonar-scanner.properties`:

```bash
cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/
sudo vi sonar-scanner.properties
```

Add:

```properties
sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
```

#### End-to-End Pipeline Overview

Run your pipeline script. If successful, check the SonarQube UI for the quality gate status. The pipeline should fail if code quality does not meet the criteria.

#### Conditionally Deploy to Higher Environments

Update `Jenkinsfile` to include conditional deployment based on branch names and quality gate results.

#### Additional Tasks

1. **Introduce Jenkins Agents/Slaves**

   - Add two more servers as Jenkins slaves.
   - Install Java and Ansible on slave nodes.
   - Configure Jenkins to use these slaves for pipeline jobs.

2. **Configure Webhook Between Jenkins and GitHub**

   - Set up a webhook in GitHub repository settings to trigger Jenkins builds on code push.

3. **Optional: Experience Pentesting**

   - Configure Wireshark in a pentest environment for exploration.

   - Add the Wireshark role
  
     ### Conclusion

By finalizing these steps, you'll establish a comprehensive CI/CD pipeline using Jenkins, enhanced with quality gates through SonarQube, automated deployment with Ansible, scalability via Jenkins agents, and integration with GitHub through webhooks. This well-rounded setup will equip you to manage real-world DevOps challenges effectively. Be sure to adjust the configurations to fit your specific project needs and infrastructure.


