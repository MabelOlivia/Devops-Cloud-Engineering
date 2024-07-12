# Ansible Dynamic Assignments (Include) and Community Roles

In this project, we will introduce dynamic assignments using the `include` module. You may be wondering, what is the difference between static and dynamic assignments?

From previous projects, you can already tell that static assignments use the `import` Ansible module. The module that enables dynamic assignments is `include`.

- **import**: Static
- **include**: Dynamic

When the `import` module is used, all statements are pre-processed at the time playbooks are parsed. This means that when you execute the `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. Thus, during actual execution, if any statement changes, such changes will not be considered. Hence, it is static.

On the other hand, when the `include` module is used, all statements are processed only during the execution of the playbook. This means that after the statements are parsed, any changes to the statements encountered during execution will be used.

Take note that in most cases it is recommended to use static assignments for playbooks because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment-specific variables as we will be introducing in this project.

## Introducing Dynamic Assignment Into Our structure


In your `https://github.com/<your-name>/ansible-config-mgt` GitHub repository, start a new branch and call it `dynamic-assignments`.

```bash
git checkout -b dynamic-assignments
```

Create a new folder, name it `dynamic-assignments`. Then inside this folder, create a new file and name it `env-vars.yml`. We will instruct `site.yml` to include this playbook later. For now, let us keep building up the structure.

```bash
mkdir dynamic-assignments
touch dynamic-assignments/env-vars.yml
```


Since we will be using the same Ansible configuration for multiple environments, and each of these environments will have certain unique attributes, such as server names and IP addresses, we will need a way to set values to variables specific to each environment.

For this reason, we will now create a folder to keep each environment's variables file. Therefore, create a new folder `env-vars`, then for each environment, create new YAML files which we will use to set variables.

```bash
mkdir env-vars
touch env-vars/dev.yml env-vars/stage.yml env-vars/uat.yml env-vars/prod.yml
```


Your layout should now look like this.

```
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml
```

<img width="191" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/fea9d9d3-75e3-4f3a-9b28-54fb1ea773ea">


Now paste the instruction below into the `env-vars.yml` file.

```
---
- name: looping through list of available files
  include_vars: "{{ item }}"
  with_first_found:
    - files:
        - dev.yml
        - stage.yml
        - prod.yml
        - uat.yml
      paths:
        - "{{ playbook_dir }}/../env-vars"
  tags:
    - always
```


Notice 3 things to notice here:

1. **Use of `include_vars` instead of `include`:**
   - This is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the `include` module is deprecated and variants of `include_*` must be used. These are:
     - `include_role`
     - `include_tasks`
     - `include_vars`
   - In the same version, variants of `import` were also introduced, such as:
     - `import_role`
     - `import_tasks`

2. **Special variables `{{ playbook_dir }}` and `{{ inventory_file }}`:**
   - `{{ playbook_dir }}` helps Ansible determine the location of the running playbook, and from there navigate to other paths on the filesystem.
   - `{{ inventory_file }}` dynamically resolves to the name of the inventory file being used, then appends `.yml` so that it picks up the required file within the `env-vars` folder.

3. **Including variables using a loop:**
   - `with_first_found` implies that while looping through the list of files, the first one found is used. This is beneficial because it allows us to always set default values in case an environment-specific `env` file does not exist.


## Update `site.yml` with dynamic assignments

Update the `site.yml` file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats.)

`site.yml` should now look like this:

```yaml
---
- hosts: all
  name: Include dynamic variables
  become: yes
  tasks:
    - include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always

- import_playbook: ../static-assignments/common.yml

- import_playbook: ../static-assignments/uat-webservers.yml

- import_playbook: ../static-assignments/loadbalancers.yml
```

## Community Roles

Now it is time to create a role for the MySQL database - it should install the MySQL package, create a database, and configure users. But why should we reinvent the wheel? There are tons of roles that have already been developed by other open-source engineers out there. These roles are actually production-ready and dynamic to accommodate most Linux flavors. With Ansible Galaxy, we can simply download a ready-to-use Ansible role and keep going.

**Download MySQL Ansible Role**

You can browse available community roles [here](https://galaxy.ansible.com). We will be using a MySQL role developed by geerlingguy.

*Hint:* To preserve your GitHub in its current state after you install a new role, make a commit and push to the master branch of your `ansible-config-mgt` directory. Of course, you must have git installed and configured on the Jenkins-Ansible server. For more convenient work with code, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhooks and Jenkins jobs to update your code on the Jenkins-Ansible server, so you can disable them - we will be using Jenkins later for a better purpose.



On Jenkins-Ansible server make sure that git is installed with git --version, then go to ansible-config-mgt directory and run

```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```

<img width="243" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/3267a9e6-6014-4f7b-b3e0-4c8fdd43e806"> <br>
<br>

<img width="461" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5b34da6c-553a-41dc-b946-bb9a964a89e3"> <br>

<br>

<img width="325" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/0dd90497-7fb2-42bf-931a-f5f17ce69401"> <br>

<br>

Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql

<img width="500" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f2682216-b187-4115-916d-b5e984e15db9">


```
ansible-galaxy role install geerlingguy.mysql
```

<img width="402" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/a6056091-674b-4b53-874e-ddd2d869494d">

**Rename the folder to mysql**

```
mv geerlingguy.mysql/ mysql
or
mv roles/geerlingguy.mysql roles/mysql
```

<img width="339" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/8cfccd2f-b9ba-4430-8ce0-35c25d943086">



Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

**Create Database and mysql user (roles/mysql/vars/main.yml)**

```
mysql_root_password: ""
mysql_databases:
  - name: tooling
    encoding: utf8
    collation: utf8_general_ci
mysql_users:
  - name: webaccess
    host: "172.31.32.0/20" # Webserver subnet cidr
    password: Admin123
    priv: "tooling.*:ALL"
```


### Create a new playbook inside `static-assignments` folder and name it `db-servers.yml` , update it with mysql roles.

```
- hosts: db_servers
  become: yes
  vars_files:
    - vars/main.yml
  roles:
    - { role: mysql }
```


Now it is time to upload the changes into your GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```

<img width="523" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/7bc8fa5f-b61d-4a61-b4ad-e5bff4fb1653">

Now, if you are satisfied with your codes, you can create a Pull Request and Merge it to main branch on GitHub

![image](https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dc4ffdbe-0eb2-4ac8-95a1-5b24f9d35b57)

## Load Balancer roles

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

1. Nginx
2. Apache
   
With your experience on Ansible so far you can:

- Decide if you want to develop your own roles, or find available ones from the community
  
**Using the Community**

<img width="586" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/5f18e515-2260-4160-a9ee-a79d505854c7">

<img width="586" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/1f484c0a-8178-4b7b-82fc-3a5bb4bab840">

```
ansible-galaxy role install geerlingguy.nginx

ansible-galaxy role install geerlingguy.apache
```

<img width="466" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/dc62950e-6acc-4e1f-8568-531353a56dee">

**Rename the installed Nginx and Apache roles**

```
mv roles/geerlingguy.nginx roles/nginx

mv roles/geerlingguy.apache roles/apache
```

**Update both `static-assignment` and `site.yml` files to refer to the roles.**

**Important Hints:**

- Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one - this is where you can make use of variables.
- Declare a variable in `defaults/main.yml` file inside the Nginx and Apache roles. Name each variable `enable_nginx_lb` and `enable_apache_lb` respectively.
- Set both values to false like this: `enable_nginx_lb: false` and `enable_apache_lb: false`.
- Declare another variable in both roles `load_balancer_is_required` and set its value to false as well.

#### For nginx

```
# roles/nginx/defaults/main.yml
enable_nginx_lb: false
load_balancer_is_required: false
```

<img width="437" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/21206c17-e669-43db-b67b-5b155c2c7577">

#### For apache

```
# roles/apache/defaults/main.yml
enable_apache_lb: false
load_balancer_is_required: false
```

#### Update assignment

`loadbalancers.yml` file

```
---
- hosts: lb
  become: yes
  roles:
    - role: nginx
      when: enable_nginx_lb | bool and load_balancer_is_required | bool
    - role: apache
      when: enable_apache_lb | bool and load_balancer_is_required | bool
```

<img width="644" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/ff84bfbd-27ec-43ad-b50e-b0186e38381a">

#### Update `site.yml` files respectively

```
---
- hosts: all
  name: Include dynamic variables
  become: yes
  tasks:
    - include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always

- import_playbook: ../static-assignments/common.yml

- import_playbook: ../static-assignments/uat-webservers.yml

- import_playbook: ../static-assignments/loadbalancers.yml

- import_playbook: ../static-assignments/db-servers.yml
```

Now you can make use of `env-vars/uat.yml` file to define which load balancer to use in the UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable Nginx by setting these in the respective environment's `env-vars` file:

#### Enable Nginx

```yaml
enable_nginx_lb: true
load_balancer_is_required: true
```


### Set up for Nginx Load Balancer

Update `roles/nginx/defaults/main.yml`


Configure Nginx virtual host

```
---
nginx_vhosts:
  - listen: "80"
    server_name: "example.com"
    root: "/var/www/html"
    index: "index.php index.html index.htm"
    locations:
              - path: "/"
                proxy_pass: "http://myapp1"

    # Properties that are only added if defined:
    server_name_redirect: "www.example.com"
    error_page: ""
    access_log: ""
    error_log: ""
    extra_parameters: ""
    template: "{{ nginx_vhost_template }}"
    state: "present"

nginx_upstreams:
- name: myapp1
  strategy: "ip_hash"
  keepalive: 16
  servers:
    - "172.31.35.223 weight=5"
    - "172.31.34.101 weight=5"

nginx_log_format: |-
  '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"'
become: yes
```

### Update roles/nginx/templates/nginx.conf.j2

Comment the line include {{ nginx_vhost_path }}/*;

This line renders the /etc/nginx/sites-enabled/ to the http configuration of Nginx.

Create a server block template in Nginx.conf.j2 for nginx configuration file to override the default in nginx role.

```
{% for vhost in nginx_vhosts %}
    server {
        listen {{ vhost.listen }};
        server_name {{ vhost.server_name }};
        root {{ vhost.root }};
        index {{ vhost.index }};

    {% for location in vhost.locations %}
        location {{ location.path }} {
            proxy_pass {{ location.proxy_pass }};
        }
    {% endfor %}
  }
{% endfor %}

```

<img width="463" alt="image" src="https://github.com/user-attachments/assets/974b2c7d-ef49-4e05-8649-f1bb1ea4e0f1">


### Update inventory/uat

```
[uat-webservers]
<private-ip> ansible_ssh_user='ec2-user'
<private-ip> ansible_ssh_user='ec2-user'

[db-server]
<private-ip> ansible_ssh_user='ubuntu'

[lb]
<private-ip> ansible_ssh_user='ubuntu'

```

```
[lb]
load_balancer ansible_host=172.31.38.203 ansible_ssh_user='ubuntu'

[uat_webservers]
Web1 ansible_host=172.31.43.69 ansible_ssh_user='ec2-user'
Web2 ansible_host=172.31.35.242 ansible_ssh_user='ec2-user'

[db_servers]
db ansible_host=172.31.36.183 ansible_ssh_user='ubuntu'
```

### Update Webservers Role in roles/webservers/tasks/main.yml to install Epel, Remi's repoeitory, Apache, PHP and clone the tooling website from your GitHub repository

```
---
- name: install apache
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: Enable apache
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
    cmd: sudo systemctl enable httpd

- name: Install EPEL release
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
    cmd: sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y

- name: Install dnf-utils and Remi repository
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
    cmd: sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y

- name: Reset PHP module
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
    cmd: sudo dnf module reset php -y

- name: Enable PHP 8.2 module
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
    cmd: sudo dnf module enable php:remi-8.2 -y

- name: Install PHP and extensions
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
    name:
      - php
      - php-opcache
      - php-gd
      - php-curl
      - php-mysqlnd
    state: present

- name: Install MySQL client
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
    name: "mysql"
    state: present

- name: Start PHP-FPM service
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
    name: php-fpm
    state: started

- name: Enable PHP-FPM service
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
    name: php-fpm
    enabled: true

- name: Set SELinux policies for web servers
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
    cmd: sudo setsebool -P httpd_execmem 1
    cmd: sudo setsebool -P httpd_can_network_connect=1
    cmd: sudo setsebool -P httpd_can_network_connect_db=1

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
    repo: https://github.com/francdomain/tooling.git
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

### Update roles/nginx/tasks/main.yml with the code below to create a task that check and stop apache if it is running

```
---
- name: Check if Apache is running
  ansible.builtin.service_facts:

- name: Stop and disable Apache if it is running
  ansible.builtin.service:
    name: apache2
    state: stopped
    enabled: no
  when: "'apache2' in services and services['apache2'].state == 'running'"
  become: yes
```

<img width="551" alt="image" src="https://github.com/user-attachments/assets/450af8e4-5d87-4e7f-8e24-61573eaec8c9">


### Now run the playbook against your uat inventory

```
ansible-playbook -i inventory/uat playbooks/site.yml --extra-vars "@env-vars/uat.yml"
```
