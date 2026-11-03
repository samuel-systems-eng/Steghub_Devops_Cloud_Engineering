# AWS CLOUD SOLUTION FOR TWO (2) COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

## INTRODUCTION

This project builds a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for an organization (`samueldomain.dpdns.org`) that uses [WordPress CMS](https://wordpress.com/) for its main business website, and a [Tooling Website](https://github.com/samuel-systems-eng/tooling) (link: https://tooling.samueldomain.dpdns.org) for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a [reverse proxy technology from NGINX](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, `WordPress` and `Tooling`, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![project_15_architecture](../AWS_CLOUD_SOLUTION_images/project_15_architecture.png)

## PROJECT IMPLEMENTATION

### Set Up a Sub-account And Create A Hosted Zone

Key prerequisites include:

1. Properly configure your AWS account and Organization Unit ([Watch How To Do This Here](https://www.youtube.com/watch?v=9PQYCc_20-Q))

     - Create an AWS Master account. (Also known as Root Account)

     - Go to AWS console, and navigate to Services > All Services > Management & Governance > AWS Organizations

![setup_subaccount_hosted_zone_01_create_organization](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_01_create_organization.png)

   - Within the Root account, create a sub-account and name it DevOps. (another email address to complete this)

   - Navigate to Add an AWS accout > fill the name (DevOps)

![setup_subaccount_hosted_zone_02_create_devops_account](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_02_create_devops_account.png)

 - Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)

From the AWS Organization page, Click on root > Action > Create new

![acreate_dev_org_unit](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_03_create_dev_org_unit.png)

   - Move the DevOps account into the Dev OU.

Slecte the account to move, then Click Action > Move, then select the OU to move the account to and click move AWS account.

![move_devops_account](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_04a_move_devops_account.png)

![success_moved_devops_account](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_04b_success_moved_devops_account.png)

 - Login to the newly created AWS account using the new email address.

![login_devops_account](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_05_login_devops_account.png)

2. Create a free domain name for the organization at [DigitalPlat One](https://digitalplat.org/) domain registrar. (DigitalPlat One was chosen as it allows full transfer of root domain access to a third party dns service like AWS route53)

![create_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_06_create_domain.png)

3. Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that here

Go to `route53` and select create hosted zone

![create_hosted_zone](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_07a_create_hosted_zone.png)

![populate_hosted_zone](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_07b_populate_hosted_zone.png)

![success_create_hosted_zone](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_07c_success_create_hosted_zone.png)

![details_hosted_zone](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_07d_details_hosted_zone.png)

- Map the hosted zone to free domain

Copy the name server (NS) values from AWS, then go to your free domain, edit the default NS values and update them with the values from AWS.

![map_aws_route53_to_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_08a_map_aws_route53_to_domain.png)

![dnschecker_map_aws_route53_to_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_01_setup_subaccount_hosted_zone/AWS_cloud_solution_01_setup_subaccount_hosted_zone_08b_dnschecker_map_aws_route53_to_domain.png)

### Set Up a Virtual Private Network (VPC)

Make reference to the architectural diagram and ensure that all configuration is aligned with it.

1. Create a [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html).

![create_vpc](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_01a_create_vpc.png)

![success_create_vpc](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_01b_success_create_vpc.png)

Enable DNS hostnames.
Actions > Edit VPC Settings > Enable DNS hostnames

![enable_vpc_hostnames](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_01c_enable_vpc_hostnames.png)

![success_enable_vpc_hostnames](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_01d_success_enable_vpc_hostnames.png)

2. Create subnets as shown in the architecture

Create public and private subnets in each availablity zones respectively. Thus, we create public subnet in availability zone A and B respectively. We create additional 4 private subnets with respect to the architecture which means creating 2 private subnets each in availability zone A and B.

![overview_create_subnet](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02a_overview_create_subnet.png)

![create_public_subnet_AZ-A](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02b_create_public_subnet_AZ-A.png)

![create_private_webserver_subnet_AZ-A](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02c_create_private_webserver_subnet_AZ-A.png)

![create_private_db_subnet_AZ-A](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02d_create_private_db_subnet_AZ-A.png)

![create_public_subnet_AZ-B](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02e_create_public_subnet_AZ-B.png)

![create_private_webserver_subnet_AZ-B](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02f_create_private_webserver_subnet_AZ-B.png)

![create_private_db_subnet_AZ-B](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_02g_create_private_db_subnet_AZ-B.png)

Enable Auto-assign public IPv4 address in the public subnets. Actions > Edit subnet settings > Enable auto-assign public IPv4

![enable_auto_ip_assignment_in_public_subnet-B](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_03a_enable_auto_ip_assignment_in_public_subnet-B.png)

![enable_auto_ip_assignment_in_public_subnet-A](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_03b_enable_auto_ip_assignment_in_public_subnet-A.png)

All Subnets

![all_subnets](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_04_all_subnets.png)

3. Create a route table and associate it with public subnets

Create a public route table for the public subnets

![create_public_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_05a_create_public_route_table.png)

![success_create_public_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_05b_success_create_public_route_table.png)

Associate it with public subnets. 

Click on the route table > Subnet Association > Edit Subnet Association.

![associate_public_subnet_to_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_05c_associate_public_subnet_to_route_table.png)

The public route table and it's associated public subnets

![details_associate_public_subnet_to_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_05d_details_associate_public_subnet_to_route_table.png)

4. Create a route table and associate it with private subnets

Create a private route table for the private subnets

![create_private_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_06a_create_private_route_table.png)

![success_create_private_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_06b_success_create_private_route_table.png)

Associate it with private subnets

![associate_private_subnet_to_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_06c_associate_private_subnet_to_route_table.png)

![details_associate_public_subnet_to_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_06d_details_associate_public_subnet_to_route_table.png)

The route tables

![all_route_tables](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_07_all_route_tables.png)

5. Create an [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

![create_internet_gateway](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_08a_create_internet_gateway.png)

![success_create_internet_gateway](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_08b_success_create_internet_gateway.png)

Attach it to the created VPC.

![attach_vpc_to_internet_gateway](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_09a_attach_vpc_to_internet_gateway.png)

![success_attach_vpc_to_internet_gateway](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_09b_success_attach_vpc_to_internet_gateway.png)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

In the public route table, Click route tab > Edit route > Add route since we are routing traffic to the internet, the destination will be 0.0.0.0/0

![attach_internet_gateway_to_public_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_10a_attach_internet_gateway_to_public_route_table.png)

![details_attach_internet_gateway_to_public_route_table](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_10b_details_attach_internet_gateway_to_public_route_table.png)

7. Create 3 Elastic IPs

![allocate_elastic_IP](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_11a_allocate_elastic_IP.png)

![success_allocate_elastic_IP](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_11b_success_allocate_elastic_IP.png)

![all_elastic_IPs](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_11c_all_elastic_IPs.png)

8. Create a [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

- Edit a route in private route table, and associate it with the Nat Gateway.

![create_NAT](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_12a_create_NAT.png)

![success_create_NAT](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_12b_success_create_NAT.png)

![edit_private_route_table_NAT](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_13a_edit_private_route_table_NAT.png)

![success_edit_private_route_table_NAT](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_13b_success_edit_private_route_table_NAT.png)

9. Create a [Security Group](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html#CreatingSecurityGroups) for:

External [Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/): External ALB will be available from the Internet

![create_security_group_ext_alb](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_14a_create_security_group_ext_alb.png)

![success_create_security_group_ext_alb](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_14b_success_create_security_group_ext_alb.png)

- Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

![create_security_group_bastion_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_15a_create_security_group_bastion_server.png)

![success_create_security_group_bastion_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_15b_success_create_security_group_bastion_server.png)

**Important Note:**
1. The command/URL provided (`curl www.canhazip.com`) did not work and was returning a webpage redirect rather than the raw IP address. 
2. Cloudflare changed how `canhazip.com` behaves on certain configurations, which is why it sent a `302 Found` error.

![camhazip_not_working](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_01a_camhazip_not_working.png)

![confirm_personal_IP](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_01b_confirm_personal_IP.png)

**Bastion Host Security Configuration Documentation**

**Objective**  

To secure the infrastructure, inbound access to the Bastion Hosts (Jump Boxes) must be strictly restricted via the Security Group. Access is locked down exclusively to authorized operator workstations using an explicit IP whitelist.  

**Retrieve Your Workstation's Public IP**  
Because standard tools can sometimes trigger web redirects, use one of the following reliable methods from your local terminal or browser to find your active public IPv4 address:

**Using the Terminal (Recommended):**  
Open local terminal (e.g., Ubuntu/WSL, macOS Terminal, or Git Bash) and execute:
```
curl ifconfig.me
```
Alternative if the above fails: `curl -4 icanhazip.com`

- Nginx Servers: Access to Nginx should only be allowed from an external Application Load balancer (ALB).

![create_security_group_nginx_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_16a_create_security_group_nginx_server.png)

![success_create_security_group_nginx_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_16b_success_create_security_group_nginx_server.png)

-Internal Application Load Balancer: This is not an internet facing ALB rather used to distribute internal traffic comming from Nginx (reverse proxy) to our Webservers Auto Scalling Groups in our private subnets. It also helps us to prevent single point of failure.

![create_security_group_internal-ALB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_17a_create_security_group_internal-ALB.png)

![success_create_security_group_internal-ALB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_17b_success_create_security_group_internal-ALB.png)

- Webservers: Access to Webservers should only be allowed from the internal ALB.

![create_security_group_webserver](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_18a_create_security_group_webserver.png)

![success_create_security_group_webserver](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_18b_success_create_security_group_webserver.png)

- Data Layer: Access to the Data layer, which is comprised of [Amazon Relational Database Service (RDS)](https://aws.amazon.com/rds/) and [Amazon Elastic File System (EFS)](https://aws.amazon.com/efs/) must be carefully desinged - only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

![create_security_group_datalayer](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_19a_create_security_group_datalayer.png)

![success_create_security_group_datalayer](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_19b_success_create_security_group_datalayer.png)

All Security Groups

![ll_security_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_02_VPC_setup/Aws_cloud_solution_02_VPC_setup_20_all_security_group.png)

### TLS Certificates From Amazon Certificate Manager (ACM)

TLS certificate is needed to handle secured connectivity to our Application Load Balancers (ALB).

1. Navigate to AWS ACM
2. Request a public wildcard certificate for the domain name we registered in Cloudns
3. Use DNS to validate the domain name
4. Tag the resource

![create_tls_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_03_TLS_certificate/Aws_cloud_solution_03_TLS_certificate_01a_create_tls_certificate.png)

![pending_approval_create_tls_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_03_TLS_certificate/Aws_cloud_solution_03_TLS_certificate_01b_pending_approval_create_tls_certificate.png)

- Ensure to create record on Route 53 after creating the certificate. This will generate a CNAME record type in Route 53. We will attach this certificate to all the load balancers.

Click, Create records in Route 53 > Create records

![create_route53_record_for_tls_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_03_TLS_certificate/Aws_cloud_solution_03_TLS_certificate_02a_create_route53_record_for_tls_certificate.png)

![success_create_route53_cname_record_for_tls_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_03_TLS_certificate/Aws_cloud_solution_03_TLS_certificate_02b_success_create_route53_cname_record_for_tls_certificate.png)

![success_aws_issued_tls_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_03_TLS_certificate/Aws_cloud_solution_03_TLS_certificate_03_success_aws_issued_tls_certificate.png)

### Setup Compute Resources

We will need to set up and configure compute resources inside our VPC. The recources related to compute are:

[EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)  
[Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-templates.html)  
[Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)  
[Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)  
[TLS Certificates](https://aws.amazon.com/certificate-manager/)  
[Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)

#### Set Up Compute Resources for Nginx, Bastion and Webservers

NB: To create the Autoscaling Groups, we need Launch Templates and Load Balancers. The Launch Templates requires AMI and Userdata while the Load balancer requires Target Group

##### Provision EC2 Instances for Nginx, Bastion and Webservers

1. Create EC2 Instances based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region. 
- Use EC2 instance of T2 family (e.g. t2.micro or similar) with security group (All traffic - anywhere).

2. Ensure that it has the following software installed:

`python`  
`ntp`  
`net-tools`  
`vim`  
`wget`  
`telnet`  
`epel-release`  
`htop`  

For Nginx, Bastion and Webserver.

**For Nginx**  

![ec2_instance_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_01_ec2_instance_details.png)

![success_nginx_ami_nginx_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_03_success_nginx_ami_nginx_server.png)

  sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

![update_nginx_server_general](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_02a_update_nginx_server_general.png)

![update_nginx_server_general](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_02b_update_nginx_server_general.png)

    sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

![epel-release__update_nginx_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_02c_epel-release__update_nginx_server.png)

    sudo yum install wget vim python3 telnet htop net-tools ntp -y

![vim_htop_python_update_nginx_server](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_02d_vim_htop_python_update_nginx_server.png)

**Create Nginx target group**

![create_nginx_server_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_04a_create_nginx_server_target_group.png)

![success_create_nginx_server_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_04b_success_create_nginx_server_target_group.png)

**Create Nginx launch template**

![create_nginx_server_launch_template-AZ-A](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_05a_create_nginx_server_launch_template-AZ-A.png)

![create_nginx_server_launch_template-AZ-B](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_05b_create_nginx_server_launch_template-AZ-B.png)

- Userdata for Nginx launch template
```
#!/bin/bash
# 1. Update and install Nginx + OpenSSL
sudo yum update -y
sudo yum install nginx openssl -y

# 2. Create a secure folder for the SSL keys
sudo mkdir -p /etc/nginx/ssl

# 3. Generate a Self-Signed TLS/SSL Certificate (Valid for 365 days)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/nginx.key \
  -out /etc/nginx/ssl/nginx.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/OU=DevOps/CN=localhost"

# 4. Create the custom Nginx configuration block to listen on Port 443 HTTPS
sudo cat << 'EOF' > /etc/nginx/conf.d/ssl.conf
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    root /usr/share/nginx/html;
    index index.html;

    location /healthstatus {
        access_log off;
        return 200 'OK';
        add_header Content-Type text/plain;
    }
}
EOF

# 5. Start and enable Nginx + Chrony Time Sync
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl start chronyd
sudo systemctl enable chronyd
```
**Create Nginx Auto Scaling Group**

![create_nginx_server_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_06a_create_nginx_server_auto_scaling_group.png)

![create_nginx_server_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_06b_create_nginx_server_auto_scaling_group.png)

![success_create_nginx_server_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_06c_success_create_nginx_server_auto_scaling_group.png)

**Instances created from Nginx Auto Scaling Group**

![instances_created_from_nginx_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/nginx/Aws_cloud_solution_04_compute_resources_nginx_06d_ec2_instances_created_from_nginx_auto_scaling_group.png)

**Repeat for Bastion and webserver**

**For Bastion**  

![ec2_instance_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_01_ec2_instance_details.png)
```
sudo yum update -y
```
![bastion_server_install_update_general.](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02a_bastion_server_install_update_general.png)

![bastion_server_install_update_general](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02b_bastion_server_install_update_general.png)
```
sudo yum install epel-release -y
```

![bastion_server_install_epel_release](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02c_bastion_server_install_epel_release.png)
```
sudo yum install python3 net-tools vim telnet wget htop -y
```

![bastion_server_install_vim_htop_wget](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02d_bastion_server_install_vim_htop_wget.png)

```
sudo systemctl start chronyd
sudo systemctl enable chronyd
sudo systemctl status chronyd
```

![bastion_server_chronyd_active_running](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02e_bastion_server_chronyd_active_running.png)
```
sudo yum install git ansible -y
```

![bastion_server_install_git_ansible](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02f_bastion_server_install_git_ansible.png)

![bastion_server_install_git_ansible](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02g_bastion_server_install_git_ansible.png)

![uccess_bastion_server_install_git_ansible](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_02h_success_bastion_server_install_git_ansible.png)

**Create Bastion Server AMI**

![success_create_bation_server_AMI](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_03_success_create_bation_server_AMI.png)

**Create Bastion Target Group**

![create_bation_server_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_04a_create_bation_server_target_group.png)

![success_create_bation_server_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_04b_success_create_bation_server_target_group.png)

**Create Bastion launch template**

![success_create_bation_server_launch_template](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_05_success_create_bation_server_launch_template.png)

- Userdata for Bastion Launch Template
```
#!/bin/bash
sudo yum update -y
# Install Git and Ansible (Ansible is fetched safely via the EPEL repo we installed)
sudo yum install git ansible -y
```
![launch_template_bastion_nginx](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_02a_launch_template_bastion_nginx.png)

**Create Bastion Auto Scaling Group**

![reate_bation_server_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_06a_create_bation_server_auto_scaling_group.png)

![success_create_bation_server_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_06b_success_create_bation_server_auto_scaling_group.png)

**Instances created from Bastion Auto Scaling Group**

![success_ec2_instances_created_from_bation_server_ASG](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/bastion/Aws_cloud_solution_04_compute_resources_bastion_06c_success_ec2_instances_created_from_bation_server_ASG.png)

![instances_by_bastion](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_03a_instances_by_bastion.png)

**For Webserver**  

![ebserver_ec2_instance_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_01_webserver_ec2_instance_details.png)

```
sudo yum update -y
```
![webserver_install_updates_general](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_02a_webserver_install_updates_general.png)

![webserver_install_updates_general](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_02b_webserver_install_updates_general.png)

```
sudo yum install epel-release -y
```
![webserver_install_epel_release](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_02c_webserver_install_epel_release.png)

```
sudo yum install python3 net-tools vim telnet wget htop php php-mysqlnd php-fpm -y
```
![webserver_install_php_vim_wget](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_02d_webserver_install_php_vim_wget.png)

![webserver_install_php_vim_wget](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_02e_webserver_install_php_vim_wget.png)
```
sudo systemctl start chronyd
sudo systemctl enable chronyd
sudo systemctl status chronyd
```

![webserver_confirm_updates_ok](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_02f_webserver_confirm_updates_ok.png)

**Create Webserver AMI**

![create_webserver_ami_image](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_03_create_webserver_ami_image.png)

**Create Tooling and Wordpress Launch Template**

![create_tooling_launch_template](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_04_create_tooling_launch_template.png)

- Userdata for tooling launch template
```
bash
#!/bin/bash
sudo yum update -y
```
- tooling target group

![tooling_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_05a_tooling_target_group.png)

![create_wordpress_launch_template](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_05_create_wordpress_launch_template.png)

- Userdata for wordpress launch template
```
bash
#!/bin/bash
sudo yum update -y

# Download and extract WordPress into the deployment directory
cd /var/www/html
sudo wget https://wordpress.org
sudo tar -xzf latest.tar.gz
sudo rm -f latest.tar.gz

# Set directory permissions so PHP can interact with the files
sudo chown -R apache:apache /var/www/html/wordpress
```
![wordpress_launch_template](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_04a_wordpress_launch_template.png)

**Create Wordpress and Tooling ASG**

![create_wordpress_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_06a_create_wordpress_auto_scaling_group.png)

![success_create_wordpress_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_06b_success_create_wordpress_auto_scaling_group.png)

![create_tooling_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_07a_create_tooling_auto_scaling_group.png)

![success_create_tooling_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_07b_success_create_tooling_auto_scaling_group.png)

**Instances from Wordpress and Tooling ASG**
![instances_created_from_wordpress_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_06c_ec2_instances_created_from_wordpress_auto_scaling_group.png)

![instances_created_from_tooling_auto_scaling_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_04_compute_resources/webserver/Aws_cloud_solution_04_compute_resources_webserver_07c_ec2_instances_created_from_tooling_auto_scaling_group.png)

### Setup EFS

Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

1. Create an EFS filesystem

2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer. 

NB: Any subnet we specify our mount target, the Amazon EFS becomes available in that subnet. As such, we will specify it in private webserver subnets so that all the resources in that subnet will have the ability to mount to the file system.

3. Associate the Security groups created earlier for data layer.

![create_efs_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_01a_create_efs_details.png)

![success_create_efs](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_01b_success_create_efs.png)

![success_create_efs](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_01c_success_create_efs.png)

4. Create an EFS access point.

This will specify where the webservers will mount with, thus creating 2 mount points for Tooling and Wordpress servers each.

Access point for wordpress server

![reate_efs_access_point](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_02a_create_efs_access_point.png)

![success_create_efs_access_point](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_02b_success_create_efs_access_point.png)

### Setup RDS

Pre-requisite:

Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

![create_rds_key](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_03a_create_rds_key.png)

![success_create_rds_key](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_03b_success_create_rds_key.png)

![rds_key_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_03c_rds_key_details.png)

Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases.

To configure RDS, follow steps below:

1. Create a subnet group and add 2 private subnets (data Layer)

![create_db_subnet_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_04a_create_db_subnet_group.png)

![db_subnet_group_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_04b_db_subnet_group_details.png)

**Create database**

![create_database](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_05a_create_database.png)

**Database Configuration details**

![create_database_configuration](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_05_EFS_RDS/Aws_cloud_solution_05_EFS_RDS_05b_create_database_configuration.png)

### Configure Application Load Balancer (ALB)

#### External Application Load Balancer To Route Traffic To NGINX

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

1. Create an Internet facing ALB
2. Ensure that it listens on HTTPS protocol (TCP port 443)
3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets
4. Choose the Certificate from ACM
5. Select Security Group
6. Select Nginx Instances as the target group

**Pre-requisite**  
Create `Wordpress` and `Tooling` Target groups

![create_wordpress_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_01a_create_wordpress_target_group.png)

![create_wordpress_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_01b_success_create_wordpress_target_group.png)

![create_tooling_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_02a_create_tooling_target_group.png)

![create_tooling_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_02b_success_create_tooling_target_group.png)

**Create External Loadbalancer**  

![create_external_ALB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_03a_create_external_ALB.png)

![uccess_create_external_ALB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_03b_success_create_external_ALB.png)

#### Application Load Balancer To Route Traffic To Webservers

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

1. Create an Internal ALB
2. Ensure that it listens on HTTPS protocol (TCP port 443)
3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets
4. Choose the Certificate from ACM
5. Select Security Group
6. Select webserver Instances as the target group
7. Ensure that health check passes for the target group

NOTE: This process must be repeated for both WordPress and Tooling websites.

**Create Internal Loadbalancer**  

![create_internal_ALB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_04a_create_internal_ALB.png)

![success_create_internal_ALB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_04b_success_create_internal_ALB.png)

![create_internal_ALB_routing_for_tooling_traffic](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_05a_create_internal_ALB_routing_for_tooling_traffic.png)

![create_internal_ALB_routing_for_tooling_traffic](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_05b_create_internal_ALB_routing_for_tooling_traffic.png)

![success_create_internal_ALB_routing_for_tooling_traffic](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_06_ALB/Aws_cloud_solution_06_ALB_05c_success_create_internal_ALB_routing_for_tooling_traffic.png)

#### Configuring DNS with Route53

Earlier in this project we registered a free domain with DigitalPlat One and configured a hosted zone in [Route53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html).

We need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as [CNAME, alias and A records](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/route-53-concepts.html).

NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. [Read here](https://support.dnsimple.com/articles/differences-between-a-cname-alias-url/#:~:text=The%20A%20record%20maps%20a,a%20name%20to%20another%20name.&text=The%20ALIAS%20record%20maps%20a,the%20HTTP%20301%20status%20code) to get to know more about the differences.

- Create an alias record for the root domain and direct its traffic to the ALB DNS name.
- Create an alias record for tooling.samueldomain.dpdns.org and direct its traffic to the ALB DNS name.

![create_route53_wordpress_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_01a_create_route53_wordpress_domain.png)

![success_create_route53_wordpress_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_01b_success_create_route53_wordpress_domain.png)

![create_route53_tooling_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_02a_create_route53_tooling_domain.png)

![success_create_route53_tooling_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_02b_success_create_route53_tooling_domain.png)

![all_route53_domain_records](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_03_all_route53_domain_records.png)

#### Ensure that health check passes for the target groups

Tooling Target Group Health Check

![tooling_ASG_healthy_status_route53_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_04a_tooling_ASG_healthy_status_route53_domain.png)

Wordpress Target Group Health Check

![wordpress_ASG_healthy_status_route53_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_04b_wordpress_ASG_healthy_status_route53_domain.png)

Bastion Target Group Health Check

![bation_ASG_healthy_status_route53_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_04c_bation_ASG_healthy_status_route53_domain.png)

Nginx Target Group Health Check

![nginx_ASG_healthy_status_route53_domain](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_04d_nginx_ASG_healthy_status_route53_domain.png)

![all_ec2_instances](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_07_route53_domain/Aws_cloud_solution_07_route53_domain_05_all_ec2_instances.png)

#### Confirmation of Bastion host working
- Bastion server (public: 13.60.65.64; private: 10.0.4.157)

![confirm_bastion_working](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_06a_confirm_bastion_working.png)

- tooling webserver (private: 10.0.5.170)

![confirm_bastion_working](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_06b_confirm_bastion_working.png)

**How to SSH into the Private Webserver via the Bastion Host**

To inspect `functions.php` on the private webserver, perform an SSH Agent Forwarding jump to allow tunnelling through the Bastion host.
Open your Ubuntu WSL terminal and execute these commands:
```
# 1. Start the local SSH agent inside Ubuntu
eval $(ssh-agent -s)

# 2. Add your Steghub project key file into the running memory agent
cd ~
ssh-add steghub-project-key.pem

# 3. Connect to your Bastion host using the "-A" flag (This forwards your identity keys)
ssh -A ec2-user@BASTION_PUBLIC_IP

Once you are logged into the Bastion terminal, jump straight into the private Tooling server's IP address without needing to copy the .pem file onto the Bastion itself:

# 4. Jump from Bastion directly into the Private Tooling Webserver

ssh ec2-user@PRIVATE_IP_OF_TOOLING_WEBSERVER

# 5. Navigate to the deployment directory to inspect the file
cd /var/www/html

# Use vim or cat to read the configuration block
cat functions.php
```

**Now let us access our wordpress and tooling website via a browser using our DNS name**

![tooling_swebsite_not_working](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_07a_tooling_swebsite_not_working.png)

![wordpress_swebsite_not_working](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_07b_wordpress_swebsite_not_working.png)

#### Troubleshooting actions**

1. Create `www` record in Route53

![troubleshoot_wordpress_website_create_www_record](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08a_troubleshoot_wordpress_website_create_www_record.png)

2. Try domain webpage again

![centos_server_page](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08b_centos_server_page.png)

The Centos server page showing here indicates either the wordpress page that should be rendered is misconfigured or missing.

3. Recreate `internal ALB` to be `internal-facing` not internet facing (ensure protocol is `https` on tls port 443)

![recreate_ALB_internal_facing](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08c_recreate_ALB_internal_facing.png)

4. Recreate `wordpress` and `tooling` target groups (to listen on `http:80` since apache listens on port 80) and their internal loadbalancers (to discontinue an earlier attempt at `host header routing`)

![recreate_tooling _target_group](<../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08d_recreate_tooling _target_group.png>)

![success_recreate_tooling _target_group](<../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08e_success_recreate_tooling _target_group.png>)

![recreate_wordpress _target_group](<../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08f_recreate_wordpress _target_group.png>)

![success_recreate_wordpress_target_group](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08g_success_recreate_wordpress_target_group.png)

![recreate_internal_LB_wordpress](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08h_recreate_internal_LB_wordpress.png)

![success_recreate_internal_LB_wordpress](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08i_success_recreate_internal_LB_wordpress.png)

![recreate_internal_LB_tooling](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08j_recreate_internal_LB_tooling.png)

![success_recreate_internal_LB_tooling](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_08k_success_recreate_internal_LB_tooling.png)

5. Update `tooling` and `wordpress` userdata

**Tooling**
```
#!/bin/bash
set -euxo pipefail

# Update packages
dnf update -y

# Install the web server, PHP and Git
dnf install -y httpd git php php-fpm php-mysqlnd \
    php-mbstring php-xml php-gd php-curl php-json

# Enable and start Apache and PHP-FPM
systemctl enable --now httpd
systemctl enable --now php-fpm

# Download your fork of the Tooling application
cd /tmp
rm -rf tooling
git clone --depth 1 https://github.com/samuel-systems-eng/tooling.git

# Install the application files into Apache's document root
mkdir -p /var/www/html
cp -a /tmp/tooling/html/. /var/www/html/

# Set appropriate ownership and permissions
chown -R apache:apache /var/www/html
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html -type f -exec chmod 644 {} \;

# Validate Apache configuration and restart the service
apachectl configtest
systemctl restart httpd
```
**Wordpress**
```
#!/bin/bash
set -euxo pipefail

# Update packages and install Apache, PHP and required extensions
dnf update -y
dnf install -y httpd php php-fpm php-mysqlnd php-gd \
    php-mbstring php-xml php-curl php-zip tar gzip curl

# Enable and start services
systemctl enable --now httpd
systemctl enable --now php-fpm

# Download the official WordPress archive
cd /tmp
curl -fL https://wordpress.org/latest.tar.gz -o wordpress.tar.gz
tar -xzf wordpress.tar.gz

# Install WordPress directly in Apache's document root
mkdir -p /var/www/html
cp -a /tmp/wordpress/. /var/www/html/

# Set ownership and permissions
chown -R apache:apache /var/www/html
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html -type f -exec chmod 644 {} \;

# Validate and restart Apache
apachectl configtest
systemctl restart httpd
```

6. Create new `tooling` and `wordpress` lauch templates

**Wordpress**

![updated_wordpress_launch_templates](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_09a_updated_wordpress_launch_templates.png)

**Tooling**

![updated_tooling_launch_templates](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_09b_updated_tooling_launch_templates.png)

7. Create Bastion loadbalancer

![create_bastion_LB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_10a_create_bastion_LB.png)

![success_create_bastion_LB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_10b_success_create_bastion_LB.png)

8. Ensure Nginx target servers are healthy
```
**Important Note**

There was a persistent `OOM (Out-of-memory)` situation whenever the auto scaling groups tried launching a new instance. The RAM of the AWS free tier base image allowed for the CentOS was unable to manage past the `sudo dnf update -y` command.  
This means that even when these commands are inserted into the `userdata` fields, there are not downloaded when the instance is created. It did not persist well after reboots even after I tried on a `test instance. Hence, there was a need to attach a swap file to enable necessary dependencies to be downloaded.
```
![nginx_target_swapfile](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_11b_nginx_target_swapfile.png)

![nginx_target_self-signed_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_11c_nginx_target_self-signed_certificate.png)

![nginx_target_config](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_11d_nginx_target_config.png)

![nginx_target_running](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_11e_nginx_target_running.png)

![nginx_target_healthy](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_11a_nginx_target_healthy.png)

9. SSH into webserver instance and download Wordpress
```
sudo ls -la /var/www/html/
sudo tail -n 30 /var/log/httpd/error_log
sudo ls -lh /tmp/wordpress.tar.gz
ls -lh /tmp/wordpress.tar.gz
cd /tmp && tar -xzf wordpress.tar.gz
sudo cp -a /tmp/wordpress/. /var/www/html/
sudo chown -R apache:apache /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
ls -l /var/www/html/index.php
```
![download_wordpress](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_12a_download_wordpress.png)

**TESTING WORDPRESS DID NOT STILL WORK**

- the wordpress health check path was modified

![wordpress_website_healthcheck_path_modified](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_13e_wordpress_website_healthcheck_path_modified.png)

- the wordpress configuration file was modified to listen to both `http` and `https`
```
sudo nano /etc/nginx/conf.d/samueldomain.conf
```
![wordpress_config_modified](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_13b_wordpress_config_modified.png)

![wordpress_config_modified](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_13c_wordpress_config_modified.png)

![wordpress_target_healthy](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_13a_wordpress_target_healthy.png)

![wordpress_website_redered_on_browser](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_13d_wordpress_website_redered_on_browser.png)

10. Wordpress Database access configuration

![wordpress_instance_inatall_mysql](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14b_wordpress_instance_inatall_mysql.png)

```
CREATE DATABASE wordpress;
CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
SHOW DATABASES;
```
![create_wordpress_DB](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14c_create_wordpress_DB.png)
```
Sudo setsebool -P httpd can_network_connect-1
getsebool httpd_can_network_connect 
httpd_can_network_connect --> on
```
![set_read_write_permissions](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14d_set_read_write_permissions.png)

![_wordpress_website_DB_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14a_wordpress_website_DB_details.png)

![success_wordpress_installed](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14e_success_wordpress_installed.png)

![wordpress_website_additional_DB_details](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14f_wordpress_website_additional_DB_details.png)
```
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the website, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'admin' );

/** Database password */
define( 'DB_PASSWORD', '[placeholder]' );

/** Database hostname */
define( 'DB_HOST', 'steghub-project15-rds-instance.cv60qk6k8vru.eu-north-1.rds.amazonaws.com' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         '7H?zX4<bgiW[PI{~q~wSknQ7zN9? BP.u#)-nM@3K)FDWc}Yc&2f~$CE9&r_el:w' );
define( 'SECURE_AUTH_KEY',  'N5X{1{$f})w_owU/lD4KGj=z*FZ8ny3WJ&E,u[8>alwLR_Ku@^65!&/<lc4pq$[<' );
define( 'LOGGED_IN_KEY',    'uU$=&0L<QcB~Qc~gnp0Y;xaE{+XGW`#Pyns|v/lk/lM}7`& wp*b*v_>*9=^sy_m' );
define( 'NONCE_KEY',        'US/f}qU/4w>6s%Ol;,J0y6}Dur8SbT_`R3%qy|K[y:>N}=^9Y0ivaU$Wi/D{K_cL' );
define( 'AUTH_SALT',        'zZo-5OEZ?;@e*(0X?,* &gfx~!`>CU/&q=L~WXR1 =l%A,&pU[]QA>WH.Ogq&@]3' );
define( 'SECURE_AUTH_SALT', 'vHU$Vw4MgsN{k;|66r{P>Oe#A6iQr{[,k/]W]A1Lk|e|^8[&>w7QUZ,ek`P?Jl)?' );
define( 'LOGGED_IN_SALT',   'z/o[d`i{Pvm+8Rb=|K;{}6*iYp@pFLSXKQ&m;DgpzY($tJ=T5Q=T!^G!-cIp`WD=' );
define( 'NONCE_SALT',       ')PnNE[!s69l-2/2l$F_}gn6w?nPWmijBoWAxK323!Hb&g%-$v#O4lp}:I?D@j-34' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 *
 * At the installation time, database tables are created with the specified prefix.
 * Changing this value after WordPress is installed will make your site think
 * it has not been installed.
 *
 * @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/#table-prefix
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

sudo semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html'
sudo restorecon -v /var/www/html
ls -ldZ /var/www/html
```
![wordpress_website_login](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14g_wordpress_website_login.png)

![success_wordpress_website_login](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_14h_success_wordpress_website_login.png)

#### StegHub DevOps Project 15: WordPress on AWS
**Troubleshooting Report — Problems, Root Causes and Resolutions**  
**Project:** WordPress deployment on AWS using a highly available web architecture  
**Region:** Europe (Stockholm) — eu-north-1  
**Operating system:** CentOS Stream 9  
**Database:** Amazon RDS for MySQL 8.4.11  
**Web technologies:** Apache HTTP Server, PHP-FPM, WordPress and MySQL  
**Status:** WordPress installation completed and administrator login verified
________________________________________
**Executive Summary**  

During the deployment of WordPress on AWS, several issues prevented the application from working correctly. These included a resource-constrained EC2 instance, incorrect SELinux file labels, a database connection failure originating from the web application, an SELinux restriction on outbound connections, an SELinux restriction on writing the WordPress configuration file, and an inability to reset the WordPress administrator password through email.  

The troubleshooting process relied on command-line diagnostics, application logs, SELinux audit records, direct database connection tests and verification through the browser.  

The principal lesson was that successful database connectivity from the MySQL client did not automatically guarantee that WordPress, running under PHP-FPM, could connect to the same database. 

Similarly, correct Linux ownership and permissions did not override SELinux access controls.
The WordPress setup was eventually completed, and the administrator account was recovered through a direct database password reset.

**Architecture and Environment**

The deployment used a multi-tier AWS architecture:

- External access layer: Route 53 DNS records and an external Application Load Balancer.
- Reverse proxy layer: Nginx instances behind the external load balancer.
- Application layer: An internal WordPress Application Load Balancer distributing traffic to EC2 web servers in private subnets.
- Database layer: An Amazon RDS for MySQL database in the private database subnets.
- Supporting infrastructure: Auto Scaling Groups, NAT Gateway, Amazon EFS and a bastion host.
- The WordPress EC2 instance used during troubleshooting had the private IP address `10.0.5.22`.
- The RDS endpoint was:
`steghub-project15-rds-instance.cv60qk6k8vru.eu-north-1.rds.amazonaws.com`
- The WordPress database was named `wordpress`. The database account used during installation was admin, and the WordPress table prefix was `wp_`.
- The application was accessed through:
`https://samueldomain.dpdns.org`
- The WordPress setup page was:
`https://samueldomain.dpdns.org/wp-admin/setup-config.php`

**Detailed problem discussion**

**Problem 1:**  
EC2 Instance Resource Exhaustion During   

Symptoms  
The initial provisioning process included a system package update:
```
dnf update -y
```
The command was killed before it completed.

Investigation and root cause  
The instance had approximately 717 MiB of RAM and no swap space at the time of the incident. Kernel logs confirmed that the Linux out-of-memory (OOM) mechanism had terminated a process.
This indicated a resource limitation rather than a WordPress application error.

Resolution  
A 2 GB swap file was created and configured for persistence through /etc/fstab.
The package installation and WordPress setup were subsequently completed manually.  
The installed software included Apache, PHP, PHP-FPM, PHP database drivers, supporting PHP extensions, and the WordPress application files.  

Lessons learned  
•	Check instance memory capacity before performing memory-intensive provisioning.  
•	Review kernel logs when installation commands are unexpectedly killed.  
•	Consider swap as a temporary safeguard on small-memory instances, not a substitute for adequate RAM.  
•	Keep provisioning scripts idempotent and validate each major stage.  

Verification:  
The WordPress files were installed, Apache and PHP-FPM were running, and the setup page became accessible.

**Problem 2:**  
WordPress Returned HTTP 403 Because of SELinux File Labels  

Symptoms  
The WordPress website initially returned `HTTP 403` Forbidden errors.  
Apache logs contained permission-related errors involving files such as index.php.  

Investigation and root cause  
The WordPress directory contained files whose SELinux labels were unsuitable for normal Apache web-content access. Some files had a `user_tmp_t` label.  
Although Linux file ownership and mode bits appeared reasonable, SELinux was enforcing an additional access-control policy.  
SELinux labels determine which operations a process domain may perform on files and directories. Apache being the owner of a file does not automatically grant permission under SELinux.

Resolution  
The file contexts were restored to their default policy-defined labels:
```
sudo restorecon -Rv /var/www/html
```
The WordPress directory then used the expected web-content label, ``httpd_sys_content_t`.

Verification  
The site subsequently became accessible through the browser and the WordPress setup page loaded.

Lessons learned  
•	Investigate both Unix permissions and SELinux labels when diagnosing `HTTP 403` errors.  
•	Use ``restorecon` to restore expected default labels.  
•	Avoid disabling SELinux simply to make an application work.  
•	Preserve enforcing mode and correct the specific policy or label causing the denial.  

**Problem 3:**   
WordPress Could Not Establish a Database Connection

Symptoms  
When the WordPress setup wizard submitted the database configuration, it initially returned:  
```
Error establishing a database connection.  
```
The browser showed an HTTP 500 response for the setup request.  

Database configuration  
The configuration entered in the WordPress setup wizard was:
```
Setting	Value
Database name	Wordpress
Database username	Admin
Database password	The configured RDS password
Database host	steghub-project15-rds-instance.cv60qk6k8vru.eu-north-1.rds.amazonaws.com
Port	MySQL default, 3306
Table prefix	
```
Diagnostic steps  
The following checks were performed:  
1.	The MySQL client was installed on the WordPress instance.
2.	A direct connection to the RDS endpoint was attempted.
3.	A successful MySQL session was established using the correct password.
4.	The wordpress database was selected successfully.
5.	PHP database extensions, including mysqli, mysqlnd and pdo_mysql, were confirmed to be present.
6.	A direct PHP mysqli connection test succeeded when supplied with the correct database credentials.
These tests established that RDS was reachable and that PHP could connect to it under the conditions of the direct test.  

Further investigation  
Because the direct tests succeeded while the web application still failed, the investigation moved beyond basic network connectivity and database credentials.  
Recent system journal entries and SELinux audit logs showed denials involving the php-fpm process and outbound connections to ports 80 and 443.  
This revealed an additional security-policy restriction affecting the web application.

Resolution  
The SELinux boolean governing HTTP-domain network connections was enabled:
```
sudo setsebool -P httpd_can_network_connect 1
```
The persistent setting was verified with:
```
getsebool httpd_can_network_connect
```
The reported value was:
```
httpd_can_network_connect --> on
```
After this change, the WordPress setup wizard progressed beyond the database connection stage and displayed the configuration-file creation instructions.

Important distinction  
The database issue was not solved by changing the RDS endpoint or rebuilding the database. Instead, investigation showed that application-level SELinux restrictions also needed to be considered.  
The successful direct MySQL and PHP tests were important because they helped distinguish a general database connectivity problem from a restriction affecting the web application process.  
The enabled boolean is relatively broad. For a hardened production deployment, review whether a narrower SELinux policy or a more limited application-specific configuration would be preferable.  

**Problem 4:**  
WordPress Could Not Create wp-config.php

Symptoms  
Once database connectivity succeeded, the setup wizard displayed a message explaining that it could not write the wp-config.php file and that the file could be created manually.  
This was a new failure occurring after the database connection problem had been resolved.

Investigation and root cause  
The directory permissions were inspected:
```
ls -ldZ /var/www/html
```
The directory initially showed:
```
unconfined_u:object_r:httpd_sys_content_t:s0
```
A Unix-level write test for the Apache account indicated that the directory was writable under the ordinary file permission model. Nevertheless, SELinux was enforcing and continued to deny the write.  
The audit log contained a relevant denial similar to:
```
comm="php-fpm"
avc: denied { write }
name="html"
tcontext=unconfined_u:object_r:httpd_sys_content_t:s0
tclass=dir
permissive=0
```
This was decisive evidence: PHP-FPM was being denied write access to the WordPress root directory because of its SELinux context.  

Inspection of SELinux file-context rules  
The existing rules were inspected with:
```
sudo semanage fcontext -l | grep -E '/var/www/html'
```
The output included writable-context rules for locations such as:

•	/var/www/html/wp-content  
•	/var/www/html/uploads  
•	/var/www/html/wp_backups  

However, there was no matching local rule making the root directory `/var/www/html` writable.

Resolution  
A temporary persistent file-context rule was added for the WordPress root directory:
```
sudo semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html'
sudo restorecon -v /var/www/html
```
The result confirmed that the directory was relabelled from `httpd_sys_content_t` to `httpd_sys_rw_content_t`.

The label was verified using:
```
ls -ldZ /var/www/html
```
The directory then showed:
```
unconfined_u:object_r:httpd_sys_rw_content_t:s0
```
On retrying the WordPress setup process, WordPress reported that wp-config.php already existed and provided a link to continue to the installation page.  
This confirmed that the configuration-file creation stage had progressed successfully.  

Security consideration and required follow-up  
The root-directory rule was introduced to get past the installation-time write restriction. It should not automatically be treated as the ideal permanent security configuration.
Making the WordPress root directory writable under SELinux can grant the web application more write capability than it needs during normal operation.
After confirming that WordPress is fully installed and the configuration file is working, the temporary override was removed.

**Problem 5:**
HTTPS Certificate Warning on the Apex   

Observation  
The website was accessible through HTTPS, but the browser displayed a certificate warning for:
`https://samueldomain.dpdns.org`  
The ACM certificate in use covered:
`*.samueldomain.dpdns.org`

Root cause and resolution  
A wildcard certificate for `*.samueldomain.dpdns.org` covers subdomains such as `tooling.samueldomain.dpdns.org`, but it does not cover the apex domain `samueldomain.dpdns.org`.  
Consequently, the browser could report a hostname mismatch even while the website and WordPress setup page were reachable.  

11. Check `tooling` webpage

![failed_tooling_website](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15a_failed_tooling_website.png)

12. Create `swapfile` to enable download of required dependencies for the `tooling` website

![create_tooling_swapfile](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15c_create_tooling_swapfile.png)

13. Install `php` and `git`

![install_php_and_git](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15d_install_php_and_git.png)

![install_php_and_git](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15e_install_php_and_git.png)

![complete_install_php_and_git](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15f_complete_install_php_and_git.png)

![confirmed_working_php_and_git](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15g_confirmed_working_php_and_git.png)

14. Confirm selinux settings

![check_selinux_policy](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15h_check_selinus_policy.png)

15. Confirm `tooling` target group status

![tooling_target_group_healthy](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15b_tooling_target_group_healthy.png)

16. Check tooling webpage on browser

![failed_tooling_website](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_15i_failed_tooling_website.png)
**TOOLING WEBPAGE DID NOT WORK**

17. Rechecked restarted Nginx instance for required download of dependencies and realised instance was killed during download because of out-of-memory (OOM)

![rechecked_restarted_nginx_instance_swapfile_required](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16a_rechecked_restarted_nginx_instance_swapfile_required.png)

![instances_rechecked_restarted_nginx_instance](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16aa_ec2_instances_rechecked_restarted_nginx_instance.png)

![ssh_ec2_instances](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16ab_ssh_ec2_instances.png)

![check_swapfile_status](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16ac_check_swapfile_status.png)

18. Confirm nginx installations

![confirm_nginx_installations](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16b_confirm_nginx_installations.png)

19. Confirm Nginx self-signed certificate

![created_nginx_self-signed_certificate](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16c_created_nginx_self-signed_certificate.png)

20. Confirm nginx working

![confirmed_nginx_working_tooling_down](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16d_confirmed_nginx_working_tooling_down.png)

21. Recheck selinux policy for `tooling` website

![recheck_selinux_policy_tooling](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16e_recheck_selinux_policy_tooling.png)

22. `Tooling` webpage renders on browser using dns name

![success_tooling_webpage_browser](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16f_success_tooling_webpage_browser.png)

![success_tooling_webpage_renders_on_browser](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16g_success_tooling_webpage_renders_on_browser.png)

![success_tooling_webpage_renders_on_browser](../AWS_CLOUD_SOLUTION_images/Aws_cloud_solution_08_troubleshooting/Aws_cloud_solution_08_troubleshooting_16h_success_tooling_webpage_renders_on_browser.png)

##### Additional troubleshooting actions

**StegHub DevOps Project 15: Troubleshooting the Tooling Website and Nginx Reverse Proxy**

**Objective**  
The objective of this troubleshooting exercise was to restore the Tooling EC2 instance, ensure the Tooling application was functioning correctly, resolve a 502 Bad Gateway error affecting the public Tooling website, and verify that the application could be accessed through the public domain:
https://tooling.samueldomain.dpdns.org

The troubleshooting covered the Tooling application server, internal Application Load Balancer (ALB), Nginx reverse proxy, and SELinux security configuration.

**Troubleshooting the Tooling EC2 instance**  

1. Identifying the incomplete bootstrap  
The Tooling EC2 instance (i-00e2b8ef287de6b35, private IP 10.0.2.184) had not completed its initial setup. Examination of the bootstrap process showed that the script had stopped at:
```
dnf update -y
```
The instance had approximately 717 MiB of RAM, and the update process had encountered an out-of-memory condition. This prevented the subsequent installation and configuration of the web application environment from completing.  

2. Adding swap memory  
To provide additional virtual memory and reduce the likelihood of the package-management process exhausting available RAM, a 2 GiB swap file was created and enabled.  
The resulting memory status showed:  
•	Physical memory: approximately 717 MiB.  
•	Swap capacity: 2.0 GiB.  
•	Swap usage immediately after setup: 0 bytes.  

This allowed the remaining installation and configuration tasks to proceed with additional memory headroom.

3. Installing the application dependencies  
The necessary web server, PHP components, and supporting packages were installed:
```
sudo dnf install -y \
  httpd php php-fpm php-mysqlnd \
  php-mbstring php-xml php-gd php-curl \
  php-json git
  ```
The Tooling repository was then cloned and the application files copied into the Apache document root:
```
git clone https://github.com/samuel-systems-eng/tooling.git
sudo cp -a /tmp/tooling/html/. /var/www/html/
sudo chown -R apache:apache /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```
The repository was used as the source of the Tooling application files.

4. Correcting SELinux file labels  
The application files had been copied from `/tmp`, which left them with inappropriate SELinux labels for serving web content. The labels were corrected using:
```
sudo restorecon -Rv /var/www/html
```
This restored the expected SELinux file contexts for the Apache document root.

5. Verifying the application locally  
The application was tested directly on the Tooling instance:
```
curl -I http://localhost/login.php
```
The request returned HTTP 200, confirming that the local web server could serve the Tooling login endpoint.  
The AWS tooling-TG target group also reported the Tooling target as healthy. Together, these checks established that the application server was responding correctly before the remaining public-access problem was addressed.

**Troubleshooting the Nginx reverse-proxy instance**  
1. Identifying the Nginx bootstrap failure  
The Nginx EC2 instance (10.0.4.78) also experienced an incomplete bootstrap. Its cloud-init script stopped at `dnf update -y`, with system logs showing an out-of-memory condition.  
The script was intended to install Nginx and OpenSSL, generate a self-signed TLS certificate, configure the reverse-proxy virtual hosts, and start the required services.  
The instance was recovered by adding a 2 GiB swap file, then completing the missing installation and configuration steps manually.  
The installed packages included Nginx and OpenSSL. The Nginx service was subsequently enabled and started.  
2. Restoring the Nginx HTTPS configuration  
The Nginx configuration defined separate virtual hosts for the WordPress and Tooling domains. For Tooling, the relevant configuration was:
```
server {
    listen 443 ssl;
    server_name tooling.samueldomain.dpdns.org;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    resolver 169.254.169.253 ipv6=off valid=30s;

    location / {
        proxy_pass https://internal-Internal-tooling-LB-2018188508.eu-north-1.elb.amazonaws.com;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```
The configuration forwards requests received for `tooling.samueldomain.dpdns.org` to the internal Tooling ALB over HTTPS.  
The Nginx configuration was validated and the service restarted:
```
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl restart nginx
sudo systemctl is-active nginx
```
The configuration test succeeded, and the Nginx service reported active.

**Diagnosing the 502 Bad Gateway error**  
Although the Tooling target was healthy and Nginx was running, opening the public Tooling domain initially returned:
`502 Bad Gateway`  
The investigation followed the request through the proxy and internal load balancer rather than assuming the application itself was broken.

1. Checking the internal ALB listener  
The initial Nginx configuration used HTTP when forwarding requests to the internal Tooling ALB.
A direct request to the internal ALB over HTTP port 80 failed with a connection-refused error. Inspection of the ALB configuration established that it had an HTTPS listener on port 443 only, not an HTTP listener on port 80.  
This explained why the original HTTP upstream configuration could not work.  
The Nginx proxy_pass directive was changed to use HTTPS:
```
sudo sed -i \
's|proxy_pass http://internal-Internal-tooling-LB-2018188508.eu-north-1.elb.amazonaws.com;|proxy_pass https://internal-Internal-tooling-LB-2018188508.eu-north-1.elb.amazonaws.com;|' \
/etc/nginx/conf.d/ssl.conf
```
The updated configuration was validated and reloaded:
```
sudo nginx -t
sudo systemctl reload nginx
```
A direct request from the Nginx instance to the internal Tooling ALB then returned:  
`HTTP 200`

This confirmed that the internal ALB and its application target were reachable over the correct protocol.

2. Identifying the SELinux denial 
 
Despite correcting the upstream protocol, requests passing through Nginx still returned `502 Bad Gateway`.    
The Nginx error log contained messages similar to:
`connect() to 10.0.5.60:443 failed (13: Permission denied)`while connecting to upstream.  

The Permission denied error indicated that the problem was not simply an unreachable application. SELinux was a strong candidate because it can prevent a web server process from making outbound network connections.  

To confirm the cause, the following commands were executed:
```
getenforce
sudo ausearch -m AVC -ts recent | tail -n 20
```
The first command returned:
```
Enforcing
The audit log then supplied decisive evidence:
avc: denied { name_connect }
comm="nginx"
dest=443
scontext=system_u:system_r:httpd_t:s0
tclass=tcp_socket
permissive=0
```
This record established that SELinux, in enforcing mode, was denying Nginx permission to establish a network connection to port 443. Nginx was operating in the httpd_t security domain.

3. Applying the SELinux fix  

Rather than disabling SELinux or changing the instance to permissive mode, the appropriate network-connection Boolean was enabled:
```
sudo setsebool -P httpd_can_network_connect 1
The -P option makes the setting persistent across reboots.
```
This change allows processes in the HTTP service domain to make network connections permitted by the policy. It resolved the specific restriction preventing Nginx from connecting to the internal Tooling ALB.

4. Final verification  

After applying the SELinux fix, Nginx was reloaded and the Tooling login endpoint was tested through the local Nginx reverse proxy.
The command used was:
```
sudo systemctl reload nginx

curl -k -sS -o /dev/null -w 'HTTP %{http_code}\n' \
  --resolve tooling.samueldomain.dpdns.org:443:127.0.0.1 \
  https://tooling.samueldomain.dpdns.org/login.php
The result was:
HTTP 200
```
The --resolve option directed this test request to 127.0.0.1 while preserving the requested domain name and HTTPS connection. This isolated the local Nginx proxy path for testing.  

Finally, the public URL was opened in a browser, and the Tooling website rendered successfully.

**Lessons learned**

This troubleshooting exercise demonstrated several practical DevOps principles.

1. Investigate each layer independently. A healthy application target does not automatically mean the public website will work. The application, load balancer listener, reverse proxy, network access, and operating-system security policy must all work together.
2. Use logs and targeted tests to locate the fault. The local curl test, Nginx error log, and SELinux audit records progressively narrowed the problem to the outbound proxy connection.
3. Check listener protocols before changing network rules. The internal ALB exposed HTTPS on port 443, so configuring Nginx to use HTTP was incompatible with the actual listener configuration.
4. Understand the interaction between SELinux and reverse proxies. Nginx can be running and correctly configured yet still fail to connect to an upstream server because SELinux denies the connection.
5. Account for resource constraints during automated provisioning. Both the Tooling and Nginx instances experienced out-of-memory problems during package updates. Adding swap provided additional memory headroom and allowed the recovery work to continue.
6. Validate the final outcome from the user's perspective. A successful command-line response was important, but opening the public URL in the browser provided the final confirmation that the Tooling website rendered successfully.


## CONCLUSION
This project successfully created a secured, scalable and cost-effective infrastructure to host two enterprise websites using various cloud services from AWS. At this point, our infrastructure is ready to host real websites' load.  

The WordPress deployment was recovered through systematic diagnosis of application behaviour, Linux resource usage, database connectivity, SELinux policy enforcement and WordPress account configuration.  

This project demonstrates practical skills in AWS infrastructure operations, Linux administration, web-server troubleshooting, PHP application support, MySQL connectivity, SELinux diagnostics and systematic root-cause analysis.

Moreover, the Tooling website's failure instances and nginx first required recovery from incomplete bootstrap scripts caused by memory exhaustion. The Nginx upstream configuration then needed to match the internal ALB's HTTPS listener. Finally, SELinux was found to be blocking Nginx's outbound connection to port 443.  

After correcting these issues, the Tooling application returned HTTP 200 through the Nginx proxy, and the public website rendered successfully in the browser.  

The exercise reinforced the importance of systematic troubleshooting, log-based diagnosis, correct load-balancer protocol configuration, Linux resource management, and secure SELinux policy administration in AWS-hosted web infrastructure.  