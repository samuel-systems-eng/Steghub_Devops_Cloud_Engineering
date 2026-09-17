# Ansible Dynamic Assignments (Include) and Community Roles

This project introduces dynamic assignments by using `include module`. It will continue configuring the UAT servers, learn and practice new Ansible concepts and modules.

EC2 Instances for this project
Ansible server

![Ansible_dynamic_assignments_S0_01_jenkins-ansible_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step0_images/Ansible_dynamic_assignments_S0_01_jenkins-ansible_server.png)

![Ansible_dynamic_assignments_S0_02a_web-1_uat_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step0_images/Ansible_dynamic_assignments_S0_02a_web-1_uat_server.png)

![Ansible_dynamic_assignments_S0_02b_web-2_uat_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step0_images/Ansible_dynamic_assignments_S0_02b_web-2_uat_server.png)

From [previous project](https://github.com/samuel-systems-eng/Steghub_Devops_Cloud_Engineering/blob/main/12_ANSIBLE_REFACTORING_AND_STATIC_ASSIGNMENTS/Ansible_refactoring_documentation/Ansible_refactoring_documentation.md), it can be seen that static assignments use `import` Ansible module. The module that enables dynamic assignments is `include`.

Hence,
```
import module = Static assignments
include module = Dynamic assignments
```
When the `import` module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static. On the other hand, when `include` module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

Take note that in most cases it is recommended to use `static assignments` for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, dynamic assignments are used with environment specific variables as will be introduced in this project.

## Introducing Dynamic Assignment Into the structure

In the https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it `dynamic-assignments`.
```
git checkout -b dynamic-assignments
```
Create a new folder, name it `dynamic-assignments`. Then inside this folder, create a new file and name it `env-vars.yml`. We will instruct `site.yml` to include this playbook later. For now, let us keep building up the structure.
```
mkdir dynamic-assignments  
touch dynamic-assignments/env-vars.yml
```
![Ansible_dynamic_assignments_S1_01_create_dynamic_assignments_branch_and_folder](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_01_create_dynamic_assignments_branch_and_folder.png)

![Ansible_dynamic_assignments_S1_02_create_env-vars_folder_files](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_02_create_env-vars_folder_files.png)

Your GitHub shall have following structure by now. 
```
├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
```
Note: Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as `servername`, `ip-address` etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment's variables file. Therefore, create a new folder `env-vars`, then for each environment, create new YAML files which we will use to set variables.
```
mkdir env-vars

touch env-vars/dev.yml env-vars/stage.yml env-vars/uat.yml env-vars/prod.yml
```
![Ansible_dynamic_assignments_S1_03_modify_env-vars.yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_03_modify_env-vars.yml.png)

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

![Ansible_dynamic_assignments_S1_03_modify_env-vars.yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_03_modify_env-vars.yml.png)

Notice 3 things here:

1. We used `include_vars` syntax instead of `include`, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the `include` module is deprecated and variants of `include_*` must be used. 

These are:  

[include_role](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)  
[include_tasks](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)  
[include_vars](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)  

In the same version, variants of import were also introduces, such as:

[import_role](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)  
[import_tasks](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)    

2. We made use of a [special variables](https://docs.ansible.com/projects/ansible/latest/reference_appendices/special_variables.html) `{{ playbook_dir }}` and `{{ inventory_file }}. {{ playbook_dir }}` will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. `{{ inventory_file }}` on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. `with_first_found `implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

### Update site.yml with dynamic assignments
Update `site.yml` file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

site.yml should now look like this.
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
```
## Community Roles

Now it is time to create a `role` for MySQL database - it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With `Ansible Galaxy` again, we can simply download a ready to use ansible role, and keep going.

### Download Mysql Ansible Role
You can browse available community roles here We will be using a MySQL role developed by `geerlingguy`.

**Hint**: To preserve your your GitHub in actual state after you install a new role - make a commit and push to master your ansible-config-mgt directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it - we will be using Jenkins later for a better purpose.  

![Ansible_dynamic_assignments_S1_05a_push_github_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_05a_push_github_main_branch.png)

![Ansible_dynamic_assignments_S1_05b_push_github_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_05b_push_github_main_branch.png)

![Ansible_dynamic_assignments_S1_05c_push_github_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step1_env-vars_images/Ansible_dynamic_assignments_S1_05c_push_github_main_branch.png)

**Note**: due to poor network connectivity, the use of vs code with plugins such as `remote - ssh` was limited as connections to the Jenkins server was continously breaking and disconnecting making it difficult to have steady network access to the server.

### Configure vscode to work with the directory (ansible-config-mgt)

#### Configure SSH for vscode

![configure_direct_vscode_ansible_server_ssh](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01a_configure_direct_vscode_ansible_server_ssh.png)

![configure_direct_vscode_ansible_server_ssh](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01b_configure_direct_vscode_ansible_server_ssh.png)

![configure_direct_vscode_ansible_server_ssh](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01c_configure_direct_vscode_ansible_server_ssh.png)

![configure_direct_vscode_ansible_server_ssh](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01d_configure_direct_vscode_ansible_server_ssh.png)
```
HostName:  
Jenkins-Ansible  

Public IP Address: 100.25.254.76 (Elastic IP)

Private IP Address:  172.31.26.158  
```
![configure_direct_vscode_ansible_server_ssh](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01e_configure_direct_vscode_ansible_server_ssh.png)

![connecting to new_steghub_jenkins_ansible_host](<../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01f_connecting to new_steghub_jenkins_ansible_host.png>)

Click on Open Folder and Select `ansible-config-mgt`

![Ansible_dynamic_assignments_S2_01g_showing different_config_files](<../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01g_showing different_config_files.png>)

![ccess_jenckins-ansible-server_config_files](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_01h_access_jenckins-ansible-server_config_files.png)

On `Jenkins-Ansible` server make sure that git is installed with `git --version`, then go to ansible-config-mgt directory and run
```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```
![Ansible_dynamic_assignments_S2_02_confirmed_git_installed](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_02_confirmed_git_installed.png)

![Ansible_dynamic_assignments_S2_03a_create_roles-feature](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_03a_create_roles-feature.png)

#### Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql  

    ansible-galaxy role install geerlingguy.mysql

![Ansible_dynamic_assignments_S2_08_geerlingguy-mysql](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_08_geerlingguy-mysql.png)

![Ansible_dynamic_assignments_S2_03b_log_back_into_ansible_config_mgt_file](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_03b_log_back_into_ansible_config_mgt_file.png)

![Ansible_dynamic_assignments_S2_03c_check_roles_folder](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_03c_check_roles_folder.png)

![Ansible_dynamic_assignments_S2_03d_pull_down_github_main_branch_again](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_03d_pull_down_github_main_branch_again.png)

![Ansible_dynamic_assignments_S2_03e_merge_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_03e_merge_main_branch.png)

#### Rename the folder to mysql

    mv geerlingguy.mysql/ mysql 

![Ansible_dynamic_assignments_S2_03f_restore_and_rename_mysql_role](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_03f_restore_and_rename_mysql_role.png)

**Important Note: Summary of the File Cleanup**

•	**The Problem**: GitHub repository was accidentally pulled up directly into the server's root home directory (/home/ubuntu), which mixed up the files and installed the MySQL role components loosely where they didn't belong.

•	**The Solution**: I stripped out the accidental Git tracking from the home directory, moved the loose MySQL role data into a clean structure, and pulled the missing webserver configuration files straight from the GitHub main branch by briefly moving a conflicting ansible.cfg file out of the way.


Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

### Create Database and mysql user (`roles/mysql/vars/main.yml`)
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
![Ansible_dynamic_assignments_S2_04_mysql_config_file](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_04_mysql_config_file.png)

### Create a new playbook inside static-assignments folder and name it db-servers.yml , update it with mysql roles.
```
- hosts: db_servers
  become: yes
  vars_files:
    - vars/main.yml
  roles:
    - { role: mysql }
```
![Ansible_dynamic_assignments_S2_05_update_static_assignments_folder_db-servers](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_05_update_static_assignments_folder_db-servers.png)

### Upload the changes into your GitHub:
```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
![Ansible_dynamic_assignments_S2_06a_upload_changes_to_github](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_06a_upload_changes_to_github.png)

![Ansible_dynamic_assignments_S2_06b_upload_changes_to_github](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_06b_upload_changes_to_github.png)

Now, if you are satisfied with your codes, you can create a Pull Request.

![Ansible_dynamic_assignments_S2_07a_create_pull_request_to_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_07a_create_pull_request_to_main_branch.png)

![Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_07b_created_pull_request_to_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_07b_created_pull_request_to_main_branch.png)

Merge it to main branch on GitHub

![Ansible_dynamic_assignments_S2_07c_successfully_merged_role-feature_branch_to_main_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step2_mysql_images/Ansible_dynamic_assignments_S2_07c_successfully_merged_role-feature_branch_to_main_branch.png)

## Load Balancer roles

We want to be able to choose which Load Balancer to use, `Nginx` or `Apache`, so we need to have two roles respectively:

* Nginx  
* Apache  

With your experience on Ansible so far you can:

*Decide if you want to develop your own roles, or find available ones from the community*

### Using the Community

![Ansible_dynamic_assignments_S3_01a_geerlingguy-nginx](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_01a_geerlingguy-nginx.png)

![Ansible_dynamic_assignments_S3_01b_geerlingguy-apache](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_01b_geerlingguy-apache.png)
```
ansible-galaxy role install geerlingguy.nginx

ansible-galaxy role install geerlingguy.apache
```
![Ansible_dynamic_assignments_S3_02a_install_geerlingguy-nginx-apache](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_02a_install_geerlingguy-nginx-apache.png)

![Ansible_dynamic_assignments_S3_02b_install_geerlingguy-nginx-apache](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_02b_install_geerlingguy-nginx-apache.png)

Rename the installed Nginx and Apache roles
```
mv geerlingguy.nginx nginx

mv geerlingguy.apache apache
```
![Ansible_dynamic_assignments_S3_03_change_name_to_nginx_apache](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_03_change_name_to_nginx_apache.png)

#### Update both static-assignment and site.yml files to refer the roles

Important Hints:

Since you cannot use both `Nginx` and `Apache` load balancer, you need to add a condition to enable either one - this is where you can make use of variables.

Declare a variable in `defaults/main.yml` file inside the Nginx and Apache roles. Name each variables `enable_nginx_lb` and `enable_apache_lb` respectively.

Set both values to `false` like this `enable_nginx_lb: false` and `enable_apache_lb: false`.

Declare another variable in both roles `load_balancer_is_required` and set its value to `false` as well  

**For nginx**
```
# roles/nginx/defaults/main.yml
enable_nginx_lb: false

load_balancer_is_required: false
```
![Ansible_dynamic_assignments_S3_04a_change_nginx_default_main_yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_04a_change_nginx_default_main_yml.png)

![Ansible_dynamic_assignments_S3_04b_change_apache_default_main_yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_04b_change_apache_default_main_yml.png)

#### Update assignment  

    loadbalancers.yml file
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
![Ansible_dynamic_assignments_S3_05_change_static_assignments_loadbalancer_main_yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_05_change_static_assignments_loadbalancer_main_yml.png)

#### Update site.yml files respectively
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
![Ansible_dynamic_assignments_S3_06_change_playbooks_site_yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_06_change_playbooks_site_yml.png)

Now you can make use of `env-vars\uat.yml` file to define which loadbalancer to use in UAT environment by setting respective environmental variable to `true`.

You will activate load balancer, and `enable nginx` by setting these in the respective environment's `env-vars` file.
```
Enable Nginx
enable_nginx_lb: true
load_balancer_is_required: true
```
![Ansible_dynamic_assignments_S3_07_change_env_vars_uat_yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_07_change_env_vars_uat_yml.png)


### Set up for Nginx Load Balancer

#### Update roles/nginx/defaults/main.yml  

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

    \# Properties that are only added if defined:
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
![Ansible_dynamic_assignments_S3_08_modify_nginx_vhost_default_main_yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_08_modify_nginx_vhost_default_main_yml.png)

**Update roles/nginx/templates/nginx.conf.j2  
Comment the line include {{ nginx_vhost_path }}/*;**



This line renders the `/etc/nginx/sites-enabled/` to the http configuration of `Nginx`.

Create a server block template in `Nginx.conf.j2` for nginx configuration file to override the default in nginx role.

![Ansible_dynamic_assignments_S3_09_nano_commands_modify_nginx_apache](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_09_nano_commands_modify_nginx_apache.png)

![Ansible_dynamic_assignments_S3_10_comment_out_nginx_vhost](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_10_comment_out_nginx_vhost.png)

![Ansible_dynamic_assignments_S3_11_nginx_new_server_block](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_11_nginx_new_server_block.png)

#### Update inventory/uat
```
[lb]
load_balancer ansible_host=172.31.18.75 ansible_ssh_user='ubuntu'

[uat_webservers]
Web1 ansible_host=172.31.17.206 ansible_ssh_user='ec2-user'
Web2 ansible_host=172.31.27.78 ansible_ssh_user='ec2-user'

[db_servers]
db ansible_host=172.31.24.240 ansible_ssh_user='ubuntu'
```
![Ansible_dynamic_assignments_S3_12_inventory_uat](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_12_inventory_uat.png)

#### Update Webservers Role in roles/webservers/tasks/main.yml to install Epel, Remi's repoeitory, Apache, PHP and clone the tooling website from your GitHub repository
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
![Ansible_dynamic_assignments_S3_13_update_roles_webserver_tasks](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_13_update_roles_webserver_tasks.png)

#### Update roles/nginx/tasks/main.yml with the code below to create a task that check and stop apache if it is running
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
![Ansible_dynamic_assignments_S3_14_update_nginx_tasks](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_14_update_nginx_tasks.png)

#### Now run the playbook against your uat inventory
```
ansible-playbook -i inventory/uat playbooks/site.yml --extra-vars "@env-vars/uat.yml"
```
![Ansible_dynamic_assignments_S3_15a_playbook_nginx_failed](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_15a_playbook_nginx_failed.png)

![Ansible_dynamic_assignments_S3_15b_playbook_nginx_failed](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_15b_playbook_nginx_failed.png)

During troubleshooting, there was need to create key file in Jenkins server

![Ansible_dynamic_assignments_S3_15c_create_key_pair_file](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_15c_create_key_pair_file.png)

Modify playbook `site.yml file` and `env-vars` file to follow conventional formats that makes it easy for parsing the file

![Ansible_dynamic_assignments_S3_15d_modify_playbook_site_yml_file](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_15d_modify_playbook_site_yml_file.png)

![Ansible_dynamic_assignments_S3_15e_modified_env-vars_yml_file](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_15e_modified_env-vars_yml_file.png)

Modify SELinux policies by splitting it into 3 different policies at the webserver tasks main.yml file

![Ansible_dynamic_assignments_S3_15f_modified_selinus_policies_file](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_15f_modified_selinus_policies_file.png)

NOW playbook is okay and running successfully 

**Note**: Due to AWS free tier restrictions only `Jenkins-Ansible server`, `uat-loadbalancer server` and the two webservers - `web1-uat` and `web2-uat` - could be powered on while the database server - `db server` was powered down)

![Ansible_dynamic_assignments_S3_16a_successful_nginx_configured](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_16a_successful_nginx_configured.png)

![Ansible_dynamic_assignments_S3_16b_successful_nginx_configured](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_16b_successful_nginx_configured.png)

![Ansible_dynamic_assignments_S3_16c_successful_nginx_configured](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_16c_successful_nginx_configured.png)

![Ansible_dynamic_assignments_S3_16d_successful_nginx_configured](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_16d_successful_nginx_configured.png)

**Notes**: 
1. To manage AWS free tier restrictions that allows only four simultenously running instances while ensuring that database related tasks are completed, the `web2-uat server` was `powered down` to allow the `database server` to be `powered up`. Consequently, the playbook was re-run to update the database server
2. Due to poor network connectivity, a lightweight terminal was used instead of vs code to re-run the playbook and complete project tasks as vs code and related plugins such as the `remote-ssh` was not able to maintain steady server connections to complete project tasks.
3. `env-uat` was inserted into the playbook command to tell ansible exactly where to fetch the environment variables for the `uat inventory`.

![Ansible_dynamic_assignments_S3_19a_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19a_web2-uat_off_successful_nginx_playbook_db_server.png)

![Ansible_dynamic_assignments_S3_19b_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19b_web2-uat_off_successful_nginx_playbook_db_server.png)

![Ansible_dynamic_assignments_S3_19c_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19c_web2-uat_off_successful_nginx_playbook_db_server.png)

![Ansible_dynamic_assignments_S3_19d_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19d_web2-uat_off_successful_nginx_playbook_db_server.png)

![Ansible_dynamic_assignments_S3_19e_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19e_web2-uat_off_successful_nginx_playbook_db_server.png)

![Ansible_dynamic_assignments_S3_19f_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19f_web2-uat_off_successful_nginx_playbook_db_server.png)

![Ansible_dynamic_assignments_S3_19g_web2-uat_off_successful_nginx_playbook_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_19g_web2-uat_off_successful_nginx_playbook_db_server.png)

**Engineering Documentation: Resolution of MySQL Authentication Plugin Conflict**

**1. Incident / Issue Summary**
During the final execution stage of the Ansible deployment configuration, the playbook failed at the database provisioning tier during the user creation task (roles/mysql/tasks/users.yml). While the database schema (tooling) was initialized successfully, the creation of the database user (webaccess) returned the following fatal driver error:
```
[ERROR]: Task failed: Module failed: (1524, "Plugin 'mysql_native_password' is not loaded")
```
**2. Root Cause Analysis**
The underlying cause is a structural compatibility discrepancy between the downloaded `Ansible Galaxy downstream role (geerlingguy.mysql)` and `modern releases of the MySQL Database Engine (MySQL 8.0 / 8.4+)` running on the Dev-DB-server target instance:
* MySQL Engine Architecture Shift: In recent enterprise database releases, Oracle completely deprecated and disabled the legacy `mysql_native_password` authentication mechanism by default, shifting the core platform security to use the modern, highly secure `caching_sha2_password authentication` routine.
* Ansible Module Hardcoding: The baseline configuration templates inside the `geerlingguy.mysql role` contain hardcoded connection directives that explicitly instruct the engine to spin up new users using the `old mysql_native_password` syntax block.
•	The Conflict: When Ansible attempted to inject this configuration block, the database engine rejected the raw instruction, generating `error code 1524` because the requested legacy mechanism was not loaded into the active memory stack of the running database engine. Passing overrides via configuration variables was bypassed because the role's internal code overrode manual parameter overrides.

**3. Corrective Action & Justification for Manual Override**
To prevent deep, destructive modification to the upstream Ansible Galaxy core codebase files—which violates the DevOps best practice of maintaining clean, immutable, and reusable framework profiles — a targeted manual SQL injection override was executed directly on the live database engine.
By establishing an interactive shell session on the database target host (172.31.24.240), the user was created natively utilizing the platform's default, active authentication plugin layout:
```
CREATE USER 'webaccess'@'%' IDENTIFIED BY 'Admin123';
GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'%';
FLUSH PRIVILEGES;
```

**4. Architectural Result**
- Clean Subnet Bridging: Setting the access mask host parameter to % ensures robust, flexible network connectivity. Regardless of horizontal autoscaling operations or dynamic IP mapping shifts executed by AWS on the backend UAT web server subnet layers (172.31.16.0/20), authentication requests will pass cleanly.
- Pipeline Unblocked: The manual baseline user mapping successfully satisfied the dependencies for all downstream deployment configurations, allowing the completion of the multi-tier application pipeline safely within our active cloud resource threshold parameters.

![Ansible_dynamic_assignments_S3_20a_web2-uat_off_resolved_failed_mysql_native_password_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_20a_web2-uat_off_resolved_failed_mysql_native_password_db_server.png)

![Ansible_dynamic_assignments_S3_20b_web2-uat_off_resolved_failed_mysql_native_password_db_server](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_20b_web2-uat_off_resolved_failed_mysql_native_password_db_server.png)

#### Confirm that Nginx is enabled and running and Apache is disabled

Access load balancer server
```
ssh -i "my-devec2key.pem" ubuntu@100.24.8.55
```
![Ansible_dynamic_assignments_S3_17a_access_uat-loadbalancer_nginx](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_17a_access_uat-loadbalancer_nginx.png)

Check Nginx and Apache status

![Ansible_dynamic_assignments_S3_17b_confirm_uat-loadbalancer_nginx_no_apache2](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_17b_confirm_uat-loadbalancer_nginx_no_apache2.png)

#### Check ansible configuration for Nginx

Log into load balancer instance to verify that upstream web server IPs (uat-webservers 1 and 2) are compiling cleanly inside the file:
```
ssh -i STEG_MEAN.pem ubuntu@100.24.8.55  

sudo vi /etc/nginx/nginx.conf
```
Verification Checklist: Scroll down to verify that upstream myapp1 lists your two uat web server IPs (private IPs: 172.31.17.206 and 172.31.27.78). Exit by pressing Esc, typing :q!, and hitting Enter. Type exit to disconnect.

![Ansible_dynamic_assignments_S3_18a_confirm_config_file_for_nginx_LB](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_18a_confirm_config_file_for_nginx_LB.png)

![Ansible_dynamic_assignments_S3_18b_confirm_config_file_for_nginx_LB](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_18b_confirm_config_file_for_nginx_LB.png)

#### Update the website's configuration with the database and user credentials to connect to the database (in /var/www/html/function.php file)

Log into live `Web1 server` instance to configure the database credentials inside the website code:
```
ssh -i STEG_MEAN.pem ec2-user@172.31.35.223

sudo vi /var/www/html/functions.php
```
Find the database connection block and update the parameters to map directly to live servers:
```
$db = mysqli_connect('172.31.24.240', 'webaccess', 'Admin123', 'tooling');
```
(Press Esc, type :wq, and hit Enter to save and exit).

![Ansible_dynamic_assignments_S3_21_web2-uat_off_update_functions-php](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_21_web2-uat_off_update_functions-php.png)

#### Apply tooling-db.sql command on the webservers

Since only uat-Web1 server is powered on, download repository's SQL backup script template and push the database tables directly into tooling schema:

Run the schema injection script
```
sudo mysql -h 172.31.24.240 -u webaccess -p tooling < /var/www/html/tooling-db.sql
```
(Type Admin123 to process. This instantly builds your user authentication database maps).

This command failed

![Ansible_dynamic_assignments_S3_22a_web2-uat_off_webaccess_tooling_mysql_not_working](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_22a_web2-uat_off_webaccess_tooling_mysql_not_working.png)

The reason mysql failed is because this server is running the brand-new Red Hat Enterprise Linux 10 (RHEL 10). In RHEL 10, Red Hat completely removed the legacy native mysql package package map, replacing it with the mariadb client package as the standard, pre-entitled MySQL-compatible drop-in alternative. 

The mariadb package provides the exact same mysql command-line tools needed to complete required tasks.
Hence, mariadb was installed.

![Ansible_dynamic_assignments_S3_22b_web2-uat_off_install_mariadb](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_22b_web2-uat_off_install_mariadb.png)

![Ansible_dynamic_assignments_S3_22c_web2-uat_off_complete_install_mariadb](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_22c_web2-uat_off_complete_install_mariadb.png)

![Ansible_dynamic_assignments_S3_22d_web2-uat_off_webaccess_tooling_mysql_fully_working](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_22d_web2-uat_off_webaccess_tooling_mysql_fully_working.png)

![Ansible_dynamic_assignments_S3_22e_web2-uat_off_webaccess_tooling_mysql_fully_working](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_22e_web2-uat_off_webaccess_tooling_mysql_fully_working.png)

#### Access the database server from Web Server

While still logged into that Web1 server terminal, test the raw database network handshake route to ensure it can successfully cross subnets:
```
sudo mysql -h 172.31.24.240 -u webaccess -p
```
(Type Admin123 when prompted. If the mysql> prompt open up, the network, firewalls, and credentials are working perfectly! Type exit to close the database interface).

![Ansible_dynamic_assignments_S3_23_web2-uat_off_access_db-server_from_webserver](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_23_web2-uat_off_access_db-server_from_webserver.png)

![Ansible_dynamic_assignments_S3_23b_web2-uat_off_access_db-server_from_webserver](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_23b_web2-uat_off_access_db-server_from_webserver.png)

#### Create in MyQSL a new admin user with username: myuser and password: password

Log back into database command prompt from the `Web1 server` to inject new admin login credentials straight into the fresh tables:
```
1. Re-open the database link
mysql -h 172.31.24.240 -u webaccess -p tooling

2. Paste the exact administration record

INSERT INTO users(id, username, password, email, user_type, status) VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');

3. Close the interface link
EXIT;
```
![Ansible_dynamic_assignments_S3_24_web2-uat_off_create_new_db-admin_user](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_24_web2-uat_off_create_new_db-admin_user.png)

#### Access the tooling website using the LB's Public IP address on a browser

![Ansible_dynamic_assignments_S3_25a_tooling_website_not_working_browser](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_25a_tooling_website_not_working_browser.png)

The tooling website failed to render.

**Engineering Summary: UAT Multi-Tier Web & Database Automation Deployment**

1. Structural Verification & Architecture Mapping  

•	Control Node Infrastructure: Running an automated Ansible configuration pipeline (site.yml) from a dedicated control host.  
•	Target Fleet Tier Allocation: Managing a high-performance NGINX Reverse Proxy/Load Balancer environment (uat-lb), cross-bridged over private cloud networks to an active back-end Apache/PHP Application Web Node (Web1-uat) and an isolated enterprise MySQL Database Server (Dev-DB-server).  
•	Cloud Constraint Management: Safely scaled back concurrent hardware node limits by keeping the second application instance (Web2-uat) in a powered-down resource tier state to adhere to strict cloud provider resource boundaries. Applied ignore_unreachable: true to prevent workflow automation pipeline failures.  

**2. Technical Incidents & Root Cause Analysis**

During the final deployment lifecycle execution blocks, three distinct service-layer blockers dropped the communication paths, resulting in unexpected `502 Bad Gateway` and `404 Not Found routing errors`:

•	MySQL Authentication Plugin Mismatch: The automated database provisioning task collapsed because legacy connection utilities attempted to create database user mappings (webaccess) with the obsolete mysql_native_password plugin loop. Modern database engines deprecate this configuration in favor of advanced authentication frameworks.  
•	Nginx Load Balancer Routing Conflict: Nginx routing paths failed to pass traffic. The default native server block structure stayed active inside the system runtime directories, intercepting incoming connections on port 80 and blocking the custom example.com.conf backend proxy rules.  
•	SELinux Permission Sandbox Enforcements: The backend web application components failed to execute .php extensions. Because code directories were initialized as root users, the operating system's security module flagged the files as unauthorized, actively blocking Apache from reading the script configurations.

**3. Resolutions & Corrective Engineering Actions**

A sequence of targeted DevOps corrective steps successfully unblocked the data pipelines and restored multi-tier functionality:  
•	Natively Mapped User Management: Established an interactive shell connection directly inside the active database environment to construct the required schema tables. Built the account privileges configuration natively using updated parameters:
```
CREATE USER 'webaccess'@'172.31.17.206' IDENTIFIED BY 'Admin123';
GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.17.206';
FLUSH PRIVILEGES;
```
•	Proxy Configuration & Inbound Path Cleaning: Wiped the conflicting configuration links from the load balancer storage paths (sudo rm -f /etc/nginx/sites-enabled/default). Refined the explicit routing directives inside the site configurations, mapping a universal wildcard to forward all file requests straight to the back-end application server's private IP.   
•	Security Context & Application Stream Remapping: Logged onto the application host to install natively supported web processing engines (php-fpm) from cloud repository mirrors. Ran file restoration context utilities to completely unblock directory security permissions:
```
sudo restorecon -Rv /var/www/html/
sudo chcon -R -t httpd_sys_content_t /var/www/html/
```
**4. Final Operational Status**
Following a comprehensive system daemon refresh, the connection routes passed verification. curl diagnostics returned verified HTML headers for the target deployment. The Tooling Website portal fully renders, establishing secure, end-to-end data communication from the public load balancer gateway down to the persistent database tracking layer.

![Ansible_dynamic_assignments_S3_25b_correct_IP_address](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_25b_correct_IP_address.png)
Corect IP address was maintained.

![Ansible_dynamic_assignments_S3_25c_tooling_website_functional_browser](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_25c_tooling_website_functional_browser.png)

![Ansible_dynamic_assignments_S3_25d_loggedin_tooling_website_browser](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_25d_loggedin_tooling_website_browser.png)

The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

To test this, update inventory for each environment and run Ansible against each environment.

### Set up for Apache Load Balancer

#### Update roles/apache/tasks/configure-Dedian.yml

Configure Apache virtual host
```
---
- name: Add apache vhosts configuration.
  template:
    src: "{{ apache_vhosts_template }}"
    dest: "{{ apache_conf_path }}/sites-available/{{ apache_vhosts_filename }}"
    owner: root
    group: root
    mode: 0644
  notify: restart apache
  when: apache_create_vhosts | bool
  become: yes

- name: Enable Apache modules
  ansible.builtin.shell:
    cmd: "a2enmod {{ item }}"
  loop:
    - rewrite
    - proxy
    - proxy_balancer
    - proxy_http
    - headers
    - lbmethod_bytraffic
    - lbmethod_byrequests
  notify: restart apache
  become: yes

- name: Insert load balancer configuration into Apache virtual host
  ansible.builtin.blockinfile:
    path: /etc/apache2/sites-available/000-default.conf
    block: |
      <Proxy "balancer://mycluster">
        BalancerMember http://172.31.35.223:80
        BalancerMember http://172.31.34.101:80
        ProxySet lbmethod=byrequests
      </Proxy>
      ProxyPass "/" "balancer://mycluster/"
      ProxyPassReverse "/" "balancer://mycluster/"
    marker: "# {mark} ANSIBLE MANAGED BLOCK"
    insertbefore: "</VirtualHost>"
  notify: restart apache
  become: yes
```
![Ansible_dynamic_assignments_S3_26_setup_apache_configure_debian-yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_26_setup_apache_configure_debian-yml.png)

#### Enable Apache (in env-vars/uat.yml)

Switch Apache to true and nginx to false

![Ansible_dynamic_assignments_S3_27_enable_apache_envvars_uat-yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_27_enable_apache_envvars_uat-yml.png)

#### Update roles/apache/tasks/main.yml to create a task that check and stop nginx if it is running
```
---
- name: Check if nginx is running
  ansible.builtin.service_facts:

- name: Stop and disable nginx if it is running
  ansible.builtin.service:
    name: nginx
    state: stopped
    enabled: no
  when: "'nginx' in services and services['nginx'].state == 'running'"
  become: yes
```
![Ansible_dynamic_assignments_S3_28_update_apache_tasks_main-yml](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_28_update_apache_tasks_main-yml.png)

Now run the playbook against the uat inventory
```
ansible-playbook -i inventory/uat playbooks/site.yml --extra-vars "@env-vars/uat.yml"
```
![Ansible_dynamic_assignments_S3_29a_apache_ansible_playbook_failed_to_run](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_29a_apache_ansible_playbook_failed_to_run.png)

The Apache playbook run got stucked.

**Engineering Documentation: Resolution of Apache Playbook Execution Blocker**

**1. Incident / Issue Summary**

During the infrastructure environment hot-swap execution block from an Nginx configuration layer to an Apache Reverse Proxy environment (enable_apache_lb: true), the playbook automation execution stream experienced severe infrastructure latency.   
It repeatedly became unresponsive during the initial environmental setup and configuration evaluation checks (Update apt repo, Ensure MySQL Python libraries are installed), eventually triggering terminal timeouts.

**2. Root Cause Analysis**

•	Cloud Hardware Resource Thresholds: The underlying Dev-DB-server target instance (t2.micro tier) possessed less than 80MB of active, free system RAM memory space.  
•	Disk I/O Choke (Memory Thrashing): Because system memory was entirely consumed by the active MySQL engine database service footprints and Ansible evaluation workers, the operating system was forced into heavy kernel page-swapping operations onto the storage layer, driving CPU Disk Wait (wa) metrics to 78.2%.  
•	The Blocker: At this state, forcing full system package manager updates and deep system-level Python module dependency evaluations ground task processing speeds went down to a crawl, creating an apparent terminal system freeze.

**3. Corrective Action & Execution Optimization**

To bypass the memory bottlenecks without changing the structure of the underlying roles, the automation lifecycle check framework was streamlined. Because all backend applications and database definitions were already fully provisioned, the execution routine was fired utilizing an explicit tag exclusion override wrapper:
```
ansible-playbook -i inventory/uat playbooks/site.yml \
  --extra-vars "env=uat" \
  --extra-vars "@env-vars/uat.yml" \
  --skip-tags "update"
```
![Ansible_dynamic_assignments_S3_29c_apache_ansible_playbook_skip_updates](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_29c_apache_ansible_playbook_skip_updates.png)

•	Result: Stripping out the heavy, redundant system package repository cache indexing operations lowered memory demands. The playbook bypassed the resource bottlenecks in seconds and advanced straight down to complete the PLAY [lb] routing automation framework tasks successfully.

**4. Architectural Validation**

Nginx was cleanly stopped and disabled, and the Apache Reverse Proxy service layer (apache2) was initialized on Port 80. 

The configuration successfully targets the active backend application node IP (172.31.17.206), and the multi-tier Tooling Website portal fully renders through the Apache load balancer gateway with zero connection drops.

![Ansible_dynamic_assignments_S3_29d_uat_LB_ec2_instance_details](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_29d_uat_LB_ec2_instance_details.png)

![Ansible_dynamic_assignments_S3_29e_apache_ansible_successful_tooling_website_login_browser](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_29e_apache_ansible_successful_tooling_website_login_browser.png)

![Ansible_dynamic_assignments_S3_29f_apache_ansible_successful_tooling_website_admin_myuser_logged_in_browser](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_29f_apache_ansible_successful_tooling_website_admin_myuser_logged_in_browser.png)

#### Commit to roles-feature branch and merge into main branch

![Ansible_dynamic_assignments_S3_30a_commit_to_roles-feature_git_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_30a_commit_to_roles-feature_git_branch.png)

![Ansible_dynamic_assignments_S3_30b_complete_commit_to_roles-feature_git_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_30b_complete_commit_to_roles-feature_git_branch.png)

![Ansible_dynamic_assignments_S3_30c_commits_to_roles-feature_git_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_30c_commits_to_roles-feature_git_branch.png)

![Ansible_dynamic_assignments_S3_30d_open_pull_request_to_main_git_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_30d_open_pull_request_to_main_git_branch.png)

![Ansible_dynamic_assignments_S3_30e_merging_pull_request_to_main_git_branch](../Ansible_Dynamic_Assignments_images/Ansible_Dynamic_Assignments_step3_load-balancer_images/Ansible_dynamic_assignments_S3_30e_merging_pull_request_to_main_git_branch.png)

## Conclusion
This project demonstrated how to use Ansible configuration management tool with dynamic assignments to prepare UAT environment for Tooling web solution.























