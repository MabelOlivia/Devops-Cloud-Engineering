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
