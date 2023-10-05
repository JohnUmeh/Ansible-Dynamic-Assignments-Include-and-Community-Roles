
# Ansible-Dynamic-Assignments-Include-and-Community-Roles

In this project, I will be introducing dynamic assignments by using include module since i have previously used import function in static assignment in the previous [project](https://github.com/JohnUmeh/ansible-config-mgt.git) 

**NB:** When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static. On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used. 

# Introducing Dynamic Assignment Into Our structure
Create a new branch in your https://github.com/<your-name>/ansible-config-mgt github repository and name it `dynamic-assignment` Checout into this branch

![dynamicass](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/5371d0c6-4045-4018-a5b7-f62e6ea6a82e)


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
![env-varupdate](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/372771b4-4736-4350-9b22-be37c51a0adc)


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

![editsiteyml](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/6d84846e-2be3-465d-9f0e-f8f93c59cc82)


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
![gitrolemsql](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/d72b1fef-83d6-40ff-8268-14d9ca3af69d)


Intall mysql role and rename it

```
ansible-galaxy install geerlingguy.mysql
mv geerlingguy.mysql/ mysql
```
Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website previously created in [Project tooling](https://github.com/JohnUmeh/Project7-Tooling.git)

Add and push the changes in your repository 

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
![gitinitndpull](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/955429fc-c00b-4f88-9da5-78e68006ef40)

![gitmysqlpush](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/9941d4aa-c026-4f7b-b203-609c370e9ead)


Pull and maerge the changes on your github.

Create a directory in the root ansible-config-mgt folder and create these four files in it dev.yml, prod.yml, staging.yml and uat.yml

![initialcreate](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/1c4969c3-c241-4f92-9a10-274062aa201b)


# Load Balancer roles

We will be choosing either of apache or nginx loadbalancer to use in this project

Inside roles folder, add nginx and apache roles from community

```
ansible-galaxy install geerlingguy.apache
ansible-galaxy install geerlingguy.nginx

```
![nginxapache](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/a431d9ec-97a4-4037-9345-eb293d8984b7)


Rename the roles to nginx and apache respectively

```
sudo mv geerlingguy.nginx nginx
sudo mv geerlingguy.apache apache
```
Create the import functions file in the static-assignments folder and name them `loadbalancer.yml` and `db.yml`.

Put this code in loadbalacer.yml file

```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

![loadbedit](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/dcd1340b-481b-4e5f-bf00-3e49ce047206)


Put the following folder in db.ym to reference the database file

```
-hosts: dbserver
 roles:
   - mysql
```
![dbroles](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/ea0bbfcc-2a05-42ff-bc02-c3854f403131)


Add lines of code to the default folders of main.yml files of the apache and nginx role

Paste this code in the main.yml file of the default folder of apache role

```
enable_apache_lb: false
load_balancer_is_required: false
```
![admyml](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/c50dfc48-3758-4516-87ee-27ffcf0d269d)


Add the same to the main.yml file of the nginx role

```
enable_nginx_lb: false
load_balancer_is_required: false
```
![nginxmain](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/af76e09c-7b18-4cb5-b0b7-de6ef19e00b1)


Let us make use of the env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

---
enable_nginx_lb: true
load_balancer_is_required: true

enable_apache_lb: false
load_balancer_is_required: false
```
![uatlbupdate](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/974b32e7-1886-4a75-98a1-c48d14c711b6)


Update site.yml to include the roles for ansible playbook

```
---
- name: Import common file
  import_playbook: ../static-assignments/common.yml
  tags:
    - always

- name: Import env-vars file
  import_playbook: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- name: Import webservers
  import_playbook: ../static-assignments/uat-webservers.yml

- name: Import database file
  import_playbook: ../static-assignments/db.yml

- name: Loadbalancers assignment
  import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancer_is_required
```
![finalsiteyml](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/76abe19f-1a43-4f1d-a1f7-4a87c69ec5bd)

Go to ansible-config-mgt root repository and run playbook of `uat` against site.yml

`ansible-playbook -i inventory/uat.yml playbooks/site.yml`

![playbookrun](https://github.com/JohnUmeh/Ansible-Dynamic-Assignments-Include-and-Community-Roles/assets/77943759/43632729-7477-41f2-b8c2-f7f22a02faca)


