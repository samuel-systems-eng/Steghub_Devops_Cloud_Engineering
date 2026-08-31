# ANSIBLE CONFIGURATION MANAGEMENT (AUTOMATE PROJECT 7 TO 10)

In Projects 7 to 10 a lot of manual operations was performed to set up virtual servers, install and configure required software and deploy our web application.

This Project increases appreciation for DevOps tools by making most of the routine tasks automated with Ansible Configuration Management, at the same time increases confidence with writing code using declarative languages such as YAML.

Ansible Client as a Jump Server (Bastion Host).
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. In the current architecture, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server - it provides better security and reduces attack surface.

## Task

- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

On the diagram below the Virtual Private Network (VPC) is divided into two subnets - Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

![architecture_pix.png](../Ansible_config_mgt_images/Ansible_config_step0_images/Ansible_config_mgt_S0_01_architecture_pix.png)

## Step 1 - Install and Configure ANSIBLE ON EC2 Instance

1. Update Name tag on Jenkins EC2 Instance to Jenkins-Ansible. This server will be used to run playbooks.

![change _to_jenkins-ansible](<../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_01_change _to_jenkins-ansible.png>)

2. In your GitHub account create a new repository and name it ansible-config-mgt

![create_ansible_repo.png](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_02_create_ansible_repo.png)

3. Install Ansible ([See: install ansible with pip](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html#installing-ansible-with-pip))
   
**sudo apt update**

![ssh](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_03_ssh.png)

![update_ubuntu_packages](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_04_update_ubuntu_packages.png)

**sudo apt install ansible**

![install_ansible](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_05a_install_ansible.png)

![install_ansible](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_05b_install_ansible.png)

Check your ansible version

**ansible --version**

![check_ansible_version](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_06_check_ansible_version.png)

4. Configure Jenkins build job to save your repository content every time you change it – this will solidify Jenkins configuration skills acquired in Project 9

Configure a Webhook in GitHub and set the webhook to trigger ansible build. On ansible-config-mgt repository, select `Settings` > `Webhooks` > `Add webhook`

![ansible_github_webhook](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_07a_ansible_github_webhook.png)

![nsible_github_webhook](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_07b_ansible_github_webhook.png)

Create a new Freestyle project ansible in Jenkins

![create_ansible_freestyle](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_08_create_ansible_freestyle.png)

Point it to the ansible-config-mgt repository Copy the repository URL

![ansible_github_url](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_09_ansible_github_url.png)

In configuration of our ansible freestyle project choose `Git`, provide there the link to our `ansible-config-mgt` GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![ansible_config_code](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_10_ansible_config_code.png)

![nsible_config_trigger](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_11_ansible_config_trigger.png)

Configure a Post-build job to save all (**) files, like you did it in [Project 9](https://github.com/samuel-systems-eng/Steghub_Devops_Cloud_Engineering/tree/main/09_TOOLING_WEBSITE_WITH_CI_JENKINS).

![ansible_config_postbuild](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_12a_ansible_config_postbuild.png)

![ansible_config_postbuild](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_12b_ansible_config_postbuild.png)

5. Test your setup by making some change in `README.MD` file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

![ansible_README_change-1](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_13a_ansible_README_change-1.png)

Check ansible project on jenkins for the build

**Issue Summary: Jenkins Automated Build Failure via GitHub Webhook**

❌ The Problem
The automated build failed to trigger because of two configuration mismatches between GitHub and the Jenkins server:

1.	Incorrect Webhook URL Endpoint: The original GitHub payload URL was pointing to an invalid endpoint (/ansible-webhook/). Jenkins' built-in Git plugin only listens for triggers on the exact /github-webhook/ path. This caused Jenkins to ignore the connection entirely.

2.	Strict Security Restraints: The Jenkins global authorization strategy was set to "Logged-in users can do anything," with anonymous read access disabled. Because GitHub pings the server externally without user login credentials, Jenkins rejected the incoming automatic notification with an `HTTP 403 Forbidden error`.

________________________________________
**The Solution**

The automated build pipeline was fixed by aligning the security policies and correcting the communication path:

•	Step 1: Authorized Anonymous Triggers  
In `Manage Jenkin`s ➔ `Security`, the authorization policy was updated to `"Allow anonymous read access."` This gave GitHub's outside service permission to read matching project schemas without needing a password. In production environments, standard github authentication plugin may be used to ensure that strictly authenticated webhooks is allowed to communicate with the Jenkins-ansible server.

•	Step 2: Corrected the Payload URL  
The Webhook settings inside the GitHub repository were updated to route notifications to the correct plugin path:
http://18.209.15.135:8080/github-webhook/  

Once the path was corrected to /github-webhook/, the anonymous authorization loophole closed seamlessly, enabling immediate automatic builds upon every repository push.

![ansible_README_change-1](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_13b_ansible_README_change-1_status.png)

![ansible_README_change-1](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_13c_ansible_README_change-1_console.png)

Second change to the README file

![ansible_README_change-2](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_14a_ansible_README_change-2.png)

![ansible_README_change-2](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_14b_ansible_README_change-2_status.png)

![ansible_README_change-2](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_14c_ansible_README_change-2_console.png)

**ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/**

![ansible_terminal_confirmation](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_15_ansible_terminal_confirmation.png)

Note: Trigger Jenkins project execution only for /main (master) branch.

Now your setup will look like this:

![ansible_schematic](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_15b_ansible_schematic.png)

**Tip:** Allocate an Elastic IP to your Jenkins-Ansible server to avoid reconfigure of GitHub webhook to a new IP address anytime you stop/start your Jenkins-Ansible server.

Allocate elastic IP

![create_elastic_IP](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_16a_create_elastic_IP.png)

Associate the elastic IP

![associate_elastic_IP_ansible](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_16b_associate_elastic_IP_ansible.png)

![success_associate_elastic_IP](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_16c_success_associate_elastic_IP.png)

Update the webhook

![update_ansible_github_webhook](../Ansible_config_mgt_images/Ansible_config_step1_images/Ansible_config_mgt_S1_17_update_ansible_github_webhook.png)

**Note:** Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

## Step 2 – Prepare your development environment using Visual Studio Code

1. First part of DevOps is Dev, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor.

There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC).

2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

![configure_visual_studio](../Ansible_config_mgt_images/Ansible_config_step2_images/Ansible_config_mgt_S2_01_configure_visual_studio.png)

```https://github.com/samuel-systems-eng/ansible_config_mgt```

3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance:

**git clone \<ansible-config-mgt repo link>**

![clone_repo_to_server](../Ansible_config_mgt_images/Ansible_config_step2_images/Ansible_config_mgt_S2_02_clone_repo_to_server.png)

## Step 3 - Begin Ansible Development

1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature
   
**Tip:** Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool - include ticket number (`e.g. PRJ-num`) in the name of your branch and add a topic and a brief description what this branch is about - a `bugfix`, `hotfix`, `feature`, `release` (`e.g. feature/prj-145-lvm`)

**git checkout -b feature/prj-11-ansible-config**

![create_feature_branch](../Ansible_config_mgt_images/Ansible_config_step3_images/Ansible_config_mgt_S3_01_create_feature_branch.png)

2. Checkout the newly created feature branch to your local machine and start building your code and directory structure

```git fetch```  

```git checkout feature/prj-11-ansible-config```

3. Create a directory and name it `playbooks` - it will be used to store all your playbook files.

**mkdir playbooks**

4. Create a directory and name it `inventory` - it will be used to keep your hosts organised

**mkdir inventory**

![create_playbook_inventory_directory](../Ansible_config_mgt_images/Ansible_config_step3_images/Ansible_config_mgt_S3_02_create_playbook_inventory_directory.png)

5. Within the playbooks folder, create your first playbook, and name it common.yml

**touch playbooks/common.yml**

6. Within the inventory folder, create an inventory file (.yml) for each environment (`Development`, `Staging`, `Testing` and `Production`) `dev`, `staging`, `uat`, and `prod` respectively.

**touch inventory/dev.yml inventory/staging.yml inventory/uat.yml inventory/prod.yml**

These inventory files use `.ini` languages style to configure Ansible hosts.

![populate_playbook_inventory_directory](../Ansible_config_mgt_images/Ansible_config_step3_images/Ansible_config_mgt_S3_03a_populate_playbook_inventory_directory.png)

![populate_playbook_inventory_directory](../Ansible_config_mgt_images/Ansible_config_step3_images/Ansible_config_mgt_S3_03b_populate_playbook_inventory_directory.png)

## Step 4 - Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs, it is important to have a way to organize our hosts in such an Inventory

Save the below `inventory` structure in the `inventory/dev file` to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

**Note:** Ansible uses `TCP port 22` by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this you can implement the concept of `ssh-agent`. Now you need to import your key into `ssh-agent`:

To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:

For Windows users - [ssh-agent on windows](https://www.youtube.com/watch?v=OplGrY74qog)  
For Linux users - [ssh-agent on linux](https://www.youtube.com/watch?v=OplGrY74qog)

**Start the SSH Agent:**

This starts the SSH agent in your current terminal session and sets the necessary environment variables.

**eval `ssh-agent -s`**

**Add Your SSH Key:**

Add your SSH private key to the agent. replace the path with the correct path to the private key.

**ssh-add \<path-to-private-key>**

```ssh-add -l```

Verify the Key is Loaded:

Check that your key has been successfully added to the SSH agent. you should see the name of your key

![create_verify_ssh_agent](../Ansible_config_mgt_images/Ansible_config_step4_images/Ansible_config_mgt_S4_01_create_verify_ssh_agent.png)

Now, ssh into your Jenkins-Ansible server using ssh-agent

```ssh -A ubuntu@public-ip```

![access_jenkins_server_via_ssh_agent](../Ansible_config_mgt_images/Ansible_config_step4_images/Ansible_config_mgt_S4_02_access_jenkins_server_via_ssh_agent.png)

To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, See this video: Windows Linux

Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user

Update your `inventory/dev.yml` file with this snippet of code:

![setup_ansible_inventory](../Ansible_config_mgt_images/Ansible_config_step4_images/Ansible_config_mgt_S4_03_setup_ansible_inventory.png)

## Step 5 - Create a Common Playbook

It is time to start giving Ansible the instructions on what you need to be performed on all servers listed in inventory/dev

In `common.yml` playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your `playbooks/common.yml` file with following code

    ---
    name: Update web and NFS servers
    hosts: webservers, nfs
    remote_user: ec2-user
    become: true
    become_user: root
    tasks:
       - name: Ensure wireshark is at the latest version
        yum:
            name: wireshark
            state: latest

    name: Update LB and DB servers
    hosts: lb, db
    remote_user: ubuntu
    become: true
    become_user: root
    tasks:
         - name: Update apt repo
            apt:
                update_cache: yes

         - name: Ensure wireshark is at the latest version
            apt:
                name: wireshark
                state: latest


![create_common_playbook](../Ansible_config_mgt_images/Ansible_config_step5_images/Ansible_config_mgt_S5_01_create_common_playbook.png)

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task :

install wireshark utility (or make sure it is updated to the latest version) on your RHEL 9 and Ubuntu servers. It uses root user to perform this task and respective package manager: 

`yum` for RHEL 9 and `apt` for Ubuntu.

    Feel free to update this playbook with following tasks:

    Create a directory and a file inside it

    Change timezone on all servers

    Run some shell script

For a better understanding of Ansible playbooks - [watch this video from RedHat](https://www.youtube.com/watch?v=ZAdJ7CdN7DY) and [read this article](https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook) (What is an Ansible Playbook?)

## Step 6 - Update GIT with the latest code

Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.

In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of GIT. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes - it is also called Four eyes principle. Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the main branch.

Commit your code into GitHub:

Use `git` commands to add, commit and push your branch to GitHub.

    git status

    git add <selected files>

    git commit -m "commit message"

    git push origin <the feature branch>

![push_config_to_git](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_01_push_config_to_git.png)

**Create a Pull Request (PR)**

![create_git_pull_request](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_02a_create_git_pull_request.png)

![create_open_git_pull_request](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_02aa_create_open_git_pull_request.png)

Wear the hat of another developer for a second, and act as a reviewer.

![review_dev-yml_git_pull_request](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_02b_review_dev-yml_git_pull_request.png)

![review_common-yml_git_pull_request](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_02c_review_common-yml_git_pull_request.png)

![alt text](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_03a_merge_git_pull_request.png)

![success_merge_git_pull_request](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_03b_success_merge_git_pull_request.png)

Head back on your terminal, checkout from the `feature` branch into the `main`, and pull down the latest changes

![pull_to_computer_git_pull_request](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_04_pull_to_computer_git_pull_request.png)

Once your code changes appear in main branch - Jenkins will do its job and save all the files (build artifacts) to the jenkins server.

![playbook_inventory_in_main_branch](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_05_playbook_inventory_in_main_branch.png)

![alt text](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_06a_successful_ansible_pull_request_build.png)

Console output

![successful_ansible_pull_request_build](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_06b_successful_ansible_pull_request_build.png)

Check the artifact directory

```/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/```

![successful_ansible_artifact_check](../Ansible_config_mgt_images/Ansible_config_step6_images/Ansible_config_mgt_S6_07_successful_ansible_artifact_check.png)

## Step 7 - Run first Ansible test

Now, it is time to execute ansible-playbook command and verify if your playbook actually works: first setup our vs code to connect our instance for remote development, follow these steps:

1. Run ansible-playbook command:

```ansible-playbook -i inventory/dev.yml playbooks/common.yml```

![confirm_ansible_hosts_updates](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_01_confirm_ansible_hosts_updates.png)

You can go to each of the servers and check if wireshark has been installed by running

    which wireshark

    or

    wireshark --version

Check Web Server 1

![ssh_webserver-1.png](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_02a_ssh_webserver-1.png)

![check_webserver-1_wireshark_version](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_02b_check_webserver-1_wireshark_version.png)

Check Web Server 2

![ssh_webserver-2.png](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_03a_ssh_webserver-2.png)

![onfirm_webserver-2_wireshark_version](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_03b_confirm_webserver-2_wireshark_version.png)

Check NFS Server

![ssh_nfs_server.png](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_04a_ssh_nfs_server.png)

![confirm_nfs_server_wireshark_version](../Ansible_config_mgt_images/Ansible_config_step7_images/Ansible_config_mgt_S7_04b_confirm_nfs_server_wireshark_version.png)

### Summary:

•	Webservers 1 & 2 (updated): Ansible successfully connected, verified the settings, and installed/updated Wireshark on them.

•	NFS Server (updated): This updated Wireshark to the latest version. Ansible is smart enough to skip it if nothing needs to change.

•	LB & DB Servers (unreachable=1): These show as unreachable because they are stopped in AWS. The limited number of servers allowed to be running in a free account made it impossible to have the Load Balancer server and database server to be simultaneously running alongside the two webservers and NFS server.

Your updated with Ansible architecture now looks like this:

![architecture_pix](../Ansible_config_mgt_images/Ansible_config_step0_images/Ansible_config_mgt_S0_01_architecture_pix.png)

## Optional step - Repeat once again
Update your ansible playbook with some new Ansible tasks and go through the full checkout -> change codes->commit -> PR -> merge -> build -> ansible-playbook cycle again to see how easily you can manage a servers fleet of any size with just one command!

## Conclusion

We have just automated our routine tasks by implementing with Ansible configurations.