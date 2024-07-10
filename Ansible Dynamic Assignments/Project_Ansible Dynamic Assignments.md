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


