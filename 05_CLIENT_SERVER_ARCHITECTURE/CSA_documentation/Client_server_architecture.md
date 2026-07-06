# Task - Implement a Client Server Architecture using MySQL Database Management System (DBMS)

The following instructions were followed to implement the above task:

## Step 1. Create and configure two linux-based virtual servers (EC2 instance in AWS)

`mysql server`  

`mysql client`

1. Two EC2 Instances of t3.micro type and Ubuntu 24.04 LTS (HVM) was launched in the us-east-1 region using the AWS console.

![create_two_ec2](../CSA_images/CSA_S1_images/CSA_S1_1_create_two_ec2.png)

**mysql server**
![create_ec2_mysqlserver](../CSA_images/CSA_S1_images/CSA_S1_2_create_ec2_mysqlserver.png)

**mysql client**
![create_ec2_mysqlclient](../CSA_images/CSA_S1_images/CSA_S1_3_create_ec2_mysqlclient.png)

The security group inbound rule for both instances was configured with the default SSH on port 22 with source from anywhere.

![mysqlserver_securitygroup](../CSA_images/CSA_S1_images/CSA_S1_4_mysql_server_securitygroup.png)

![mysqlclient_securitygroup](../CSA_images/CSA_S1_images/CSA_S1_5_mysql_client_securitygroup.png)

2. Attached SSH key named "STEG_MEAN.pem" to access the instance on port 22

## Step 2 - On mysql server Linux Server, install MySQL Server software

1. The private ssh key permission was changed for the private key file and then used to connect to the instance by running

**chmod 400 my-ec2-key.pem**

**ssh -i "STEG_MEAN" ubuntu@18.232.131.21**

Where username=ubuntu and public ip address=18.232.131.21

![ssh_mysqlserver](../CSA_images/CSA_S2_images/CSA_S2_1_ssh_mysql_server.png)

2. Update and upgrade Ubuntu

**sudo apt update && sudo apt upgrade -y**

![upgrade_update_ubuntu](../CSA_images/CSA_S2_images/CSA_S2_2a_update_upgrade_ubuntu.png)
![update_upgrade_ubuntu](../CSA_images/CSA_S2_images/CSA_S2_2b_update_upgrade_ubuntu.png)

3. Install MySQL Server software

**sudo apt install mysql-server -y**

![install_mysqserver](../CSA_images/CSA_S2_images/CSA_S2_3a_install_mysql_server.png)
![install_mysqlserver](../CSA_images/CSA_S2_images/CSA_S2_3b_install_mysql_server.png)

4. Enable mysql server

**sudo systemctl enable mysql**

![enable_mysqlserver](../CSA_images/CSA_S2_images/CSA_S2_4_enable_mysql_server.png)

## Step 3 - On mysql client Linux Server install MySQL Client software.

1. Connect to the instance

**ssh -i "STEG_MEAN.pem" ubuntu@54.209.179.238**

Where username=ubuntu and public ip address=54.209.179.238

![ssh_mysqlclient](../CSA_images/CSA_S3_images/CSA_S3_1_ssh_mysql_client.png)

2. Update and upgrade Ubuntu

**sudo apt update && sudo apt upgrade -y**

![update_upgrade_ubuntu](../CSA_images/CSA_S3_images/CSA_S3_2a_update_upgrade_ubuntu_mysql_client.png)
![upgrade_update_ubuntu](../CSA_images/CSA_S3_images/CSA_S3_2b_update_upgrade_ubuntu_mysql_client.png)

3. Install MySQL Client software

**sudo apt install mysql-client -y**

![install_mysqlclient](../CSA_images/CSA_S3_images/CSA_S3_3_install_mysql_client.png)

## Step 4 - Use mysql server's local IP address to connect from mysql client.

By default, both of the EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses.   
Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default so it has to be opened by creating a new entry in inbound rules in mysql server Security Groups.   
For extra security, access to mysql server by all IP addresses was not allowed, only the specific local IP address of mysql client was allowed.

![connect_localip_mysqlserver](../CSA_images/CSA_S4_images/CSA_S4_1_connect_localip_mysqlserver.png)

## Step 5 - Configure MySQL server to allow connections from remote hosts.

Befor the configuration stated above, the following were implemented:
 
1. The security script of MySQL was run on mysql server by running the command:

**sudo mysql_secure_installation**

![mysql_secure_installation](../CSA_images/CSA_S5_images/CSA_S5_1_mysql_secure_installation.png)

2. Now, configure MySQL server to allow connections from remote hosts.

**sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf**

Locate bind-address = 127.0.0.1  
Replace 127.0.0.1 with 0.0.0.0

![vim_mysqlserver](../CSA_images/CSA_S5_images/CSA_S5_2_vim_mysql_server.png)

![actual_vim_mysqlserver](../CSA_images/CSA_S5_images/CSA_S5_3_actual_vim_mysql_server.png)

## Step 6 - From mysql client Linxus Sever, connect remotely to mysql server Database Engine without using SSH. The mysql utility must be used to perform this action.

**sudo mysql -u client -h 172.31.44.192 -p**

![notconncted_mysqlserver](../CSA_images/CSA_S6_images/CSA_S6_1_not_connected_mysqlserver.png)

My sql server failed to connect with "ERROR 1130 (HY000)

**The Problem**

The MySQL server rejected the connection with ERROR 1130 because MySQL uses host-based authentication.  
By default, it only trusts connections originating from localhost (the server itself). It does not recognize or trust “client” EC2 instance's private IP address (172.31.37.198), even though both instances live in the same AWS network.

**The Solution**

Logged into the MySQL server machine and told MySQL to trust the client's identity.

1. Authorize the host: Ran a SQL command to create the “client” user, explicitly attaching a network wildcard ('client'@'%').

2. Apply permissions: Granted the required database privileges to this newly defined host-specific user “client”.

![create_mysqlserver_clientuser](../CSA_images/CSA_S6_images/CSA_S6_2_create_mysqlserver_clientuser.png)

3. Open network bindings: Confirmed the server's mysqld.cnf file is configured to listen on all interfaces (bind-address = 0.0.0.0) and that AWS Security Group allows inbound traffic on port 3306 from the client.

With this solution, mysql client successfully connected remotely to mysql server Database Engine without using SSH but using the mysql utility to perform this action.

![mysqlclient_connect_mysqlserver](../CSA_images/CSA_S6_images/CSA_S6_3_mysqlclient_connect_mysqlserver.png)

## Step 7 - Check that the connection to the remote MySQL server was successfull and can perform SQL queries.

**show databases;**

![mysqlclient_connect_mysqlserver](../CSA_images/CSA_S7_images/CSA_S7_1_mysqlclient_connect_mysqlserver.png)

Create table, insert rows into table and select from the table

CREATE TABLE test_db.test_table (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);

INSERT INTO test_db.test_table (content) VALUES ("My first choice football club is Chelsea");

INSERT INTO test_db.test_table (content) VALUES ("My second choice football club is R.Madrid");

SELECT * FROM test_db.test_table;

![more_added_data](../CSA_images/CSA_S7_images/CSA_S7_2_more_mysqlclient_connect_mysqlserver.png)

## Conclusion

At this stage, this project is successfully completed and demonstrates a fully functional MySQL Client-Server set up.




























