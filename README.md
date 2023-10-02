# Ansible-Dynamic-Assignments-Include-and-Community-Roles

This project i will be introducing dynamic assignments by using include module since i have previously used import function in static assignment in the previous [project](https://github.com/JohnUmeh/ansible-config-mgt.git) 

When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used. 

# Introducing Dynamic Assignment Into Our structure
Create a new branch in your https://github.com/<your-name>/ansible-config-mgt github repository and name it `dynamic-assignment` Checout into this branch

Create a new dynamic-assignments folder. Then inside this folder, create a new file and name it `env-vars.yml`. We will instruct site.yml to include this playbook later.

The structure of the the work set-up should look like this now especially if you used ansible-galaxy in the previous project 

![Screenshot from 2023-10-02 07-39-32](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/332ff662-4579-4a42-85b8-8d5af15caeaf)

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

![Screenshot from 2023-10-02 07-52-28](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/ea2a550c-62a9-45fa-bfe1-4f6b6e096ce8)

Add this code to env-vars.yml

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
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
# Update site.yml with dynamic assignments

Add this code to site.yml file

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

# Add Community Roles
we will use ansible-galaxy to create a role for MySQL database – it should install the MySQL package, create a database and configure users.

**Hint:** To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ‘ansible-config-mgt’ directory.

In jenkins-ansible server ensure git is installed, create a roles-feature branch and checkout into the branch
```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```
Intall mysql role and rename it

```
ansible-galaxy install geerlingguy.mysql
mv geerlingguy.mysql/ mysql
```
Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website previously created in [Project tooling]()
