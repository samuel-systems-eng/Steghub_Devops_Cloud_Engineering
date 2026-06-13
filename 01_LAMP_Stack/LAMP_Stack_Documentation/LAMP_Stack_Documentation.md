# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

### Introduction
The LAMP Stack comprises of four main components: Linux, Apache, MySQL and PHP (or Python or Perl). This documentation presents how LAMP stack is setup, configured and used.


# STEP 0: Pre-requisites

1. EC2 instance of t3.micro type and Ubuntu Ubuntu 24.04 LTS (HVM) was launched in the us-east-1d availability zone using the AWS console.

![Creating EC2 instance](../LAMP_Stack_Images/creating_EC2_instance(0).png)

![Details of EC2 instance](../LAMP_Stack_Images/EC2_instance_detailss(0).png)

2. Created SSH. Created SSH key pair named STEG-LAMP to access the AWS instance on port 22.

3. The security group was configured with the following inbound rules:

    - Allow traffic on port 80 (HTTP) with source from anywhere on the internet (0.0.0.0/0).
    * Allow traffic on port 443 (HTTPS) with source from anywhere on the internet (0.0.0.0/0).
    + Allow traffic on port 22 (SSH) with source from any IP address (0.0.0.0/0). This is opened by default.

![EC2 instance security rule](../LAMP_Stack_Images/EC2_instance_security_rule(0).png)

4.  The default VPC and Subnet was used for the networking configuration.
  
![EC2 instance networking](../LAMP_Stack_Images/LAMP_networking(0).png)

5. The private ssh key that got downloaded was located, permission was changed for the private key file and then used to connect to the instance by running

    **chmod 400 STEG-LAMP.pem**

    **ssh -i STEG-LAMP.pem ubuntu@98.91.201.122**  
    (where username = ubuntu and public ip address = 98.91.201.122)

![ssh access](../LAMP_Stack_Images/ssh-access(0).png)


# STEP 1: Install Apache and Update the Firewall

Apache HTTP server is a fast, reliable, secure, highly customisable and most widely used web server developed and maintained by Apache Software Foundation.

1.  Update and upgrade list of packages in package manager
   
    **sudo apt update**

![Update package manager](../LAMP_Stack_Images/update_package_manager(1).png)

2.  Run apache2 package installation

    **sudo apt install apache2**

![install apache2-a](../LAMP_Stack_Images/install-1_apache2(1).png)

![install apache2-b](../LAMP_Stack_Images/install-2_apache2(1).png)

3.  Enable and verify that apache2 is running as a service on the OS.
   
    **sudo systemctl status apache2**

![check apache2 status](../LAMP_Stack_Images/check_apache_running_status(1).png)

4.  The server is running and can be accessed locally in the ubuntu shell by running the command below:

    **curl http://localhost:80**  
    
  ![curl localhost apache2-a](../LAMP_Stack_Images/curl-1_localhost_apache2(1).png)

  ![curl localhost apache2-b](../LAMP_Stack_Images/curl-2_localhost_apache2(1).png)

5.  Test with the public IP address (http://34.230.83.92) if the Apache HTTP server can respond to request from the internet using the url on a browser.  
   
![default apache2 page](../LAMP_Stack_Images/default_apache2_page(1).png)  
This shows that the web server is correctly installed and it is accessible through the firewall.


# STEP 2: Install MySQL

MYSQL is a popular relational database management system used within PHP environments to store and manage data. It is a popular relational database management system used within LAMP stack.

1. Install a relational database (RDB)

**sudo apt install mysql-server**
    
![install-1 mysql server](../LAMP_Stack_Images/install-1_mysql(2).png)

![install-2 mysql server](../LAMP_Stack_Images/install-2_mysql(2).png)
When prompted, install was confirmed by typing y and then Enter.

2. Log in to mysql console and set a password for root user using mysql_native_password as default authentication method using the command:

**sudo mysql**

![access mysql set root password](../LAMP_Stack_Images/access_mysql_set_rootpassword(2).png)
This connects to the MySQL server as the administrative database user root infered by the use of sudo when running the command.  

Here, the user's password was defined as "PassWord.1" with the command:  

**ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';**  

Exited MySQL shell by typing  

**exit**

3. Run an interactive script to secure MySQL

The security script comes pre-installed with mysql. This script removes some insecure settings and lock down access to the database system.

**sudo mysql_secure_installation**

![mysql secure installation](../LAMP_Stack_Images/mysql_secure_installation(2).png)
Regardless of whether the VALIDATION PASSWORD PLUGIN is set up, the server will ask to select and confirm a password for MySQL root user.

4. After changing root user password, log in to MySQL console.

A command prompt for password was noticed after running the command below:

**sudo mysql -p**

![access mysql with password](../LAMP_Stack_Images/access_mysql_withpassword(2).png)

5. Confirm MYSQL is running using the command below:  

**sudo systemctl status mysql**

![confirm mysql running](../LAMP_Stack_Images/confirm_mysql_running(2).png)

Exited MYSQL shell by typing the command:  

**exit**


# STEP 3: Install PHP

Apache is installed to serve the content and MySQL is installed to store and manage data. PHP is the component of the LAMP setup that processes code to display dynamic content to the end user.

1. Install PHP.  

The following were installed:

* php package  
* php-mysql - a PHP module that allows PHP to communicate with MySQL-based databases  
* libapache2-mod-php - to enable Apache to handle PHP files.

**sudo apt install php libapache2-mod-php php-mysql**

![install php](../LAMP_Stack_Images/install_php(3).png)

2. Confirm the PHP version using the command:

**php -v**

![confirm php install](../LAMP_Stack_Images/confirm_php_install(3).png)

At this point, the LAMP stack is completely installed and fully operational.

To test the set up with a PHP script, it's best to set up a proper Apache Virtual Host to hold the website files and folders. 

*Virtual host* allows to have multiple websites located on a single machine and it won't be noticed by the website users.


# STEP 4: Create a virtual host for the website using Apache

A domain called "projectlamp" is being setup.

1. The default directory serving the apache default page is /var/www/html. Create your document directory next to the default one.

Created the directory for projectlamp using "mkdir" command:

**sudo mkdir /var/www/projectlamp**

Assign the directory ownership with $USER environment variable which references the current system user using the command:

**sudo chown -R \$USER:\$USER /var/www/projectlamp**

![create dir assign user](../LAMP_Stack_Images/create_dir_assign_user(4).png)

2. Create and open a new configuration file in apache’s “sites-available” directory using vim.

**sudo vim /etc/apache2/sites-available/projectlamp.conf**

![new apache2 config](../LAMP_Stack_Images/new_apache2_config(4).png)

Paste in the bare-bones configuration below:  

\<VirtualHost *:80>  
  &emsp; ServerName projectlamp  
  &emsp; ServerAlias www.projectlamp  
  &emsp; ServerAdmin webmaster@localhost  
  &emsp; DocumentRoot /var/www/projectlamp  
  &emsp; ErrorLog ${APACHE_LOG_DIR}/error.log  
  &emsp; CustomLog ${APACHE_LOG_DIR}/access.log combined  
\</VirtualHost>  

![virtualhost parameters](../LAMP_Stack_Images/virtualhost_parameters(4).png)

3. Show the new file in sites-available directory

**sudo ls /etc/apache2/sites-available**  

Output:  

*000-default.conf default-ssl.conf projectlamp.conf*

![is-sites available](../LAMP_Stack_Images/ls_sites_available(4).png)
With the VirtualHost configuration, Apache will serve projectlamp using /var/www/projectlamp as its web root directory.

4. Enable the new virtual host using the command:

**sudo a2ensite projectlamp**

![enable root directory](../LAMP_Stack_Images/enable_root_directory(4).png)

5. Disable apache’s default website.

This is because Apache’s default configuration will overwrite the virtual host if not disabled. This is required if a custom domain is not being used.

![disable apache root dir](../LAMP_Stack_Images/disable_apache_root_dir(4).png)

6. Ensure the configuration does not contain syntax error using the command:  

**sudo apache2ctl configtest**

![confirm virtualhost syntax](../LAMP_Stack_Images/confirm_virtualhost_syntax_OK(4).png)

7. Reload apache for changes to take effect using the command:  

**sudo systemctl reload apache2**  

![reload apache2](../LAMP_Stack_Images/reload_apache2(4).png)

8. The new website is now active but the web root /var/www/projectlamp is still empty.  

Create an index.html file in this location so to test the virtual host work as expected.

**sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html**

![fill root dir](../LAMP_Stack_Images/fill_root_dir(4).png)

### (1) The problem/situation

When testing the new virtual host using the instance's public IP address, the browser displayed "Hello LAMP" but with a "404 Not Found" error instead of the custom web page. 

This happened because Apache did not know how to route the raw IP address, and it defaulted to an inactive configuration.  

Later, when trying to pull the AWS metadata to display on the page, the terminal kept freezing or printing a mess of dates because the AWS Instance Metadata Service (IMDSv2) was set to "Required" by default, and the command was missing the exact subfolder paths.

![problem with virtualhost](../LAMP_Stack_Images/problem_with_virtualhost(4).png)

### (2) The task required

The objective was to successfully configure a custom Apache Virtual Host for a local test domain (www.projectlamp). The page needed to dynamically fetch and display the AWS EC2 instance's public hostname and public IPv4 address using curl commands, ensuring it loaded perfectly in a web browser without any errors.

### (3) The actions taken

    • Adjusted AWS Security Settings: Modified the EC2 instance metadata settings in the AWS Console, switching IMDSv2 from Required to Optional to allow standard metadata queries.

![modify EC2 metadata](../LAMP_Stack_Images/modify_EC2instance_metadata_setttings(4).png)

 
    • Executed the Correct Command: Ran the final string using sudo sh -c alongside the explicit /latest/meta-data/public-hostname and /latest/meta-data/public-ipv4 paths. This successfully generated the clean index.html file showing the actual AWS server details.

![test index.html file](../LAMP_Stack_Images/test_index.html_file(4).png)
The output showed a clean index.html file.

9. Open the website on a browser using the public IP address (http://34.230.83.92)

![ip address site routing](../LAMP_Stack_Images/ip-address_site_routing(4).png)

10. Open the website with public dns name (port is optional)(http://ec2-34-230-83-92-compute-1.amazonaws.com)

![dns site routing](../LAMP_Stack_Images/dns_site_routing(4).png)

This file can be left in place as a temporary landing page for the application until an index.php file is set up to replace it. Once this is done, the index.html file should be renamed or removed from the document root as it will take precedence over index.php file by default.


# STEP 5:  Enable PHP on the website

With the default DirectoryIndex setting on Apache, index.html file will always take precedence over index.php file. This is useful for setting up maintenance page in PHP applications, by creating a temporary index.html file containing an informative message for visitors. The index.html then becomes the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root bringing back the regular application page. If the behaviour needs to be changed, /etc/apache2/mods-enabled/dir.conf file should be edited and the order in which the index.php file is listed within the DirectoryIndex directive should be changed.

1. Open the dir.conf file with vim to change the behaviour

**sudo vim /etc/apache2/mods-enabled/dir.conf**

![open php dir config](../LAMP_Stack_Images/open_php_dir_config(5).png)

\<IfModule mod_dir.c>  
    &emsp; \# Change this:  
    &emsp; \# DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm  
    &emsp; \# To this:  
    &emsp; DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm  
\</IfModule>

![index php config](../LAMP_Stack_Images/index_php_config(5).png)

2. Reload Apache to effect the changes using the command:

**sudo systemctl reload apache2**

![reload php to apache](../LAMP_Stack_Images/reload_php_to_apache(5).png)

3. Create a php test script to confirm that Apache is able to handle and process requests for PHP files.

A new index.php file was created inside the custom web root folder.

**vim /var/www/projectlamp/index.php**

![create index php](../LAMP_Stack_Images/create_index.php(5).png)

Add the text below in the index.php file

\<?php  
\phpinfo();

![index php root](../LAMP_Stack_Images/index_php_root(5).png)

4. Refresh the page

![php page](../LAMP_Stack_Images/php_page(5).png)

This page provides information about the server from the perspective of PHP. It is useful for debugging and to ensure the settings are being applied correctly.

After checking the relevant information about the server through this page, It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. It can always be recreated if the information is needed later.

**sudo rm /var/www/projectlamp/index.php**  

![delete php page](../LAMP_Stack_Images/delete_php_page(5).png)

### Conclusion:

The LAMP stack provides a robust and flexible platform for developing and deploying web applications. By following the guidelines outlined in this documentation, It was possible to set up, configure, and maintain a LAMP environment effectively, enabling the creation of a powerful and scalable web solution.


![LAMP stack Medal](../LAMP_Stack_Images/LAMP_stack_medal.png)