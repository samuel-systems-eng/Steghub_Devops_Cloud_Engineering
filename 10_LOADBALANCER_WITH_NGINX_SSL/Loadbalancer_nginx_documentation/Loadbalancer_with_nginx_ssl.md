# Load Balancer Solution With Nginx and SSL/TLS

A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way. 

In this project, the configuration of `Nginx Load Balancer Solution` is carried out.

It is extremely important to ensure that connections to our Web Solutions are secure and information is encrypted in transit. Connection over secured HTTP (HTTPS protocol), it's purpose and what is required for implementation is covered.

**Task**  

    This project consist of two parts:

    1. Configure Nginx as a Load Balancer
   
    2. Register a new domain name and configure secure connection

The diagram below shows the architecture of the solution

![LB_nginx_P0_01_architecture_drawing](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part0_images/LB_nginx_P0_01_architecture_drawing.png)

## Part 1 - Configure Nginx as a Load Balancer

1. Create an EC2 VM based on Ubuntu Server 26.04 LTS and name it `nginx LB`  

![LB_nginx_part1_01_create_ec2_instance](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_01_create_ec2_instance.png)

![LB_nginx_part1_02_ec2_details](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_02_ec2_details.png)

Open TCP port 80 for HTTP connections and TCP port 443 for secured HTTPS connections

![LB_nginx_part1_03_security_group](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_03_security_group.png)

2. Update `/etc/hosts` file for local DNS with Web Servers' names (e.g `web1` and `web2`) and their local IP addresses
   
Access the instance

**Nginx public IP = 54.89.227.93**  
**Nginx private IP = 172.31.21.94**

```ssh -i STEG_MEAN.pem ubuntu@18.119.165.258```

![LB_nginx_part1_04_ssh.png](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_04_ssh.png)

Update the hosts file

**Webserver 1 private IP = 172.31.18.179**  
**Webserver 2 private IP = 172.31.16.202**

```sudo vi /etc/hosts```

![LB_nginx_part1_05_access_etc_hosts](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_05_access_etc_hosts.png)

![LB_nginx_part1_06_modify_etc_hosts](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_06_modify_etc_hosts.png)


3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
   
Update the instance

```sudo apt update && sudo apt upgrade -y```

![LB_nginx_part1_07a_update_upgrade_ubuntu](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_07a_update_upgrade_ubuntu.png)

![LB_nginx_part1_07b_update_upgrade_ubuntu](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_07b_update_upgrade_ubuntu.png)

Install Nginx

```sudo apt install nginx```

![LB_nginx_part1_08a_install_nginx_modify_config](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_08a_install_nginx_modify_config.png)

![LB_nginx_part1_08b_install_nginx_modify_config](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_08b_install_nginx_modify_config.png)

4. Configure Nginx LB using the Web Servers' name defined in `/etc/hosts`.  

Open the default Nginx configuration file

```sudo vi /etc/nginx/nginx.conf```

Insert the following configuration in http section

    ```upstream myproject {
       server Web1 weight=5;
       server Web2 weight=5;
    }

    server {
        listen 80;
        server_name ww.domain.com;

        location / {
            proxy_pass http://myproject;
        }
    }
    # comment out this line
    # include /ete/nginx/sites-enabled/```

![LB_nginx_part1_09a_actual_modify_nginx_config](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_09a_actual_modify_nginx_config.png)

![LB_nginx_part1_09b_actual_modify_nginx_config](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_09b_actual_modify_nginx_config.png)

Test the server configuration

```sudo nginx -t```

![LB_nginx_part1_10_test_nginx_config_ok](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_10_test_nginx_config_ok.png)

Restart Nginx and ensure the service is up and running

```sudo systemctl restart nginx```  
```sudo systemctl status nginx```

![LB_nginx_part1_11_confirm_nginx_running](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part1_images/LB_nginx_part1_11_confirm_nginx_running.png)

## Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates

In order to get a valid `SSL certificate` we need to register a new domain name, we can do it using any Domain name registrar - a company that manages reservation of domain names. The most popular ones are: *Godaddy.com*, *Domain.com*, *Bluehost.com*.

1. Register a new domain name with any registrar of your choice in any domain zone. (e.g .com, .net, .org, .edu, info, .xyz or any other)
   
`Cloudns.net` is the domain name registrar used for this project.

![LB_nginx_part2_01_register_for_dns](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_01_register_for_dns.png)

![LB_nginx_part2_02_create_domain](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_02_create_domain.png)

![LB_nginx_part2_03_domain_creation_success](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_03_domain_creation_success.png)

2. Assign an `Elastic IP` to our `Nginx LB` server and associate our domain name with this Elastic IP
   
This is neccessary in order to have a static IP address that does not change after reboot.

**Nginx elastic IP = 18.210.150.10**

![LB_nginx_part2_04a_allocated_elastic_IP](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_04a_allocated_elastic_IP.png)

![LB_nginx_part2_04b_allocated_elastic_IP](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_04b_allocated_elastic_IP.png)

Associate the elastic IP with `Nginx LB`

![LB_nginx_part2_05a_associated_elastic_IP](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_05a_associated_elastic_IP.png)

![LB_nginx_part2_05b_associated_elastic_IP](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_05b_associated_elastic_IP.png)

![LB_nginx_part2_05c_associated_elastic_IP](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_05c_associated_elastic_IP.png)

3. Update or create `A record` your registrar to point to Nginx LB using the elastic IP

![LB_nginx_part2_06a_add_elasticIP_to_domain_A_record](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_06a_add_elasticIP_to_domain_A_record.png)

![LB_nginx_part2_06b_add_elasticIP_to_domain_A_record](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_06b_add_elasticIP_to_domain_A_record.png)

Use [dns checker](https://dnschecker.org/#A/www.toolingsolutions.abrdns.com) to verify the DNS record

![LB_nginx_part2_07_verify_dns_record](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_07_verify_dns_record.png)

4. Configure Nginx to recognize your new domain name
   
Update your `nginx.conf` with server_name `www.<your-domain-name.com` instead of server_name `www.domain.com`

In this project, the server_name is **www.toolingsolutions.abrdns.com**

```sudo vi /etc/nginx/nginx.conf```

![LB_nginx_part2_08b_modify_nginx_config_with_dns](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_08b_modify_nginx_config_with_dns.png)

![LB_nginx_part2_08a_modify_nginx_config_with_dns](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_08a_modify_nginx_config_with_dns.png)

Restart Nginx

```sudo systemctl restart nginx```

![LB_nginx_part2_09_restart_confirm_nginx_running](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_09_restart_confirm_nginx_running.png)

Check that the Web Server can be reach from a browser with the new domain name using HTTP protocol.

http://<your-domain-name.com> 
`http://www.toolingsolutions.abrdns.com`

![LB_nginx_part2_prob1a_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1a_browser_cannot_reach_webservers.png)

**DevOps Troubleshooting Report: Nginx Load Balancer Deployment**

**1. Problem Description**  
After configuring the Nginx Load Balancer to distribute traffic to the backend web nodes (Web1 and Web2) using the domain name http://www.toolingsolutions.abrdns.com, the browser returned a 404 Not Found error page.   
Furthermore, attempts to connect to the backend servers (webserver one – private IP-172.31.18.179) from the load balancer terminal via curl failed completely with connection errors.

![LB_nginx_part2_prob1b_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1b_browser_cannot_reach_webservers.png)

**2. Root Cause Analysis**  
The issue was caused by three separate configuration misalignments across the infrastructure:  

•	Nginx Configuration Typos: The server_name directive inside /etc/nginx/nginx.conf contained some typos (ww. instead of www.). This caused Nginx to ignore the custom virtual host block and serve a default local 404 page.  

•	Apache Service Naming: The backend web servers were running Red Hat Enterprise Linux (RHEL). The Apache service was inactive because it was being managed under the Debian/Ubuntu service name (apache2) instead of the Red Hat service name (httpd).  

•	Unmounted Network Storage (NFS): The Apache service (httpd) on the backend nodes was crashing immediately upon startup because the required network shared directory (/var/www/html) was unmounted, causing the DocumentRoot safety validation check to fail.

**From webserver-1**

![LB_nginx_part2_prob1c_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1c_browser_cannot_reach_webservers.png)

**3. Implemented Solutions**  

**Step 1:** Fixed the Nginx Load Balancer Configurations

•	Corrected the server name line inside `/etc/nginx/nginx.conf` to match the exact training domain layout without protocol prefixes:

    nginx

    server name toolingsolutions.abrdns.com www.toolingsolutions.abrdns.com;

![LB_nginx_part2_prob1d_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1d_browser_cannot_reach_webservers.png)

![LB_nginx_part2_prob1e_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1e_browser_cannot_reach_webservers.png)

•	Validated the new settings using `sudo nginx -t` and applied the clean configuration with `sudo systemctl reload nginx`.

![LB_nginx_part2_prob1f_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1f_browser_cannot_reach_webservers.png)

**Step 2:** Restored Network Mounts on Backend Web Servers

•	Logged into the backend nodes and manually remounted the shared applications folder from the central NFS server to the local Apache web root:

    bash
    sudo mkdir -p /var/www/html
    sudo mount -t nfs -o rw,nosuid <NFS_SERVER_PRIVATE_IP>:/mnt/apps /var/www/html

    NFS server IP = 172.31.21.78  

For webserver one (private IP = **172.31.18.179**)

![LB_nginx_part2_prob1g_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1g_browser_cannot_reach_webservers.png)

Although `setenforce 0` was disabled, in a production environment, it will be set to a level of security permitted by the organization.  

For webserver two (private IP = **172.31.16.202**)

![LB_nginx_part2_prob1h_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1h_browser_cannot_reach_webservers.png)

**Step 3:** Started and Enabled Web Server Components

•	Started the PHP processing engine (php-fpm) and the Red Hat Apache web server (httpd) services:

    bash
    sudo systemctl start php-fpm httpd
    sudo systemctl enable php-fpm httpd

**For webserver -1**

![LB_nginx_part2_prob1i_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1i_browser_cannot_reach_webservers.png)

**For webserver 2**

![LB_nginx_part2_prob1j_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1j_browser_cannot_reach_webservers.png)

4. Verification and Results
   
•	A terminal check from the Nginx Load Balancer using `curl -I http://Web1/login.php` returned a successful `HTTP/1.1 200 OK` response from the Red Hat Apache backend.

![LB_nginx_part2_prob1k_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1k_browser_cannot_reach_webservers.png)

•	Accessing http://www.toolingsolutions.abrdns.com via an external web browser loaded the core tooling application login page successfully.

![LB_nginx_part2_prob1l_browser_cannot_reach_webservers](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob1l_browser_cannot_reach_webservers.png)

5. Install certbot and request for an SSL/TLS certificate

Ensure snapd service is active and running

```sudo systemctl status snapd```

![LB_nginx_part2_22_confirm_snapd_running](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_22_confirm_snapd_running.png)

Install certbot

```sudo snap install --classic certbot```

![LB_nginx_part2_23_install_certbot](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_23_install_certbot.png)

Request SSL/TLS Certificate  

Create a Symlink in /usr/bin for Certbot: Place a symbolic link in this PATH to make it easier to run certbot from the command line without needing to specify its full path.

```sudo ln -s /snap/bin/certbot /usr/bin/certbot```

Follow the certbot instructions you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so ensure you have updated it on step 4.

```sudo certbot --nginx```

CERTBOT FAILED TO PROVIDE CERTIFICATE

![alt text](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob2a_certbot_failed_ssl_certificate.png)

**DevOps Troubleshooting Report: Certbot SSL/TLS Authentication Failure**

**1. Problem Description** 
   
When executing sudo certbot --nginx to acquire an SSL/TLS certificate for both the root domain (`toolingsolutions.abrdns.com`) and the subdomain (`www.toolingsolutions.abrdns.com`), the Let's Encrypt Certificate Authority failed to authenticate the domains, terminating with the error:

*no valid A records found for toolingsolutions.abrdns.com.*

**2. Root Cause Analysis**

The domain verification process failed because of a partial DNS infrastructure configuration [DNS Record Types and Uses]:

•	While a valid A Record had been created for the www subdomain at the DNS provider (ClouDNS.net), the root domain (toolingsolutions.abrdns.com) lacked an autonomous mapping.

•	Because Certbot was explicitly instructed to register both entries, Let's Encrypt attempted a public challenge handshake against both names. The absence of a root A record prevented the automated public server from resolving the hostname to the Nginx Elastic IP address, failing the verification challenge [DNS Record Types and Uses].

**3. Implemented Solution**

•	Logged into the ClouDNS.net management console.

•	Created a new A Record to handle root domain routing:

o   Host/Name: Left blank (representing the apex/root domain @).
o	Points to: The AWS Elastic IP address assigned to the Nginx Load Balancer instance.
•	Allowed a brief window for DNS propagation, then successfully re-executed the certificate configuration command:`sudo certbot --nginx`

![LB_nginx_part2_prob2b_certbot_failed_ssl_certificate](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob2b_certbot_failed_ssl_certificate.docx.png)

4. Verification and Results
   
The `Let's Encrypt` ACME server successfully validated both records. Certbot generated the cryptographic keys, safely stored the certificates locally, and automatically injected the necessary secure listen 443 ssl blocks into the `/etc/nginx/nginx.conf` orchestration layer. 

![LB_nginx_part2_prob2c_certbot_failed_ssl_certificate](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_prob2c_certbot_failed_ssl_certificate.docx.png)

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>.

You shall be able to access your wesite using HTTPS protocol (Uses TCP port 443) and see a padlock image in your browsers' search string. Click on the padlock icon and you can see the detail of the certificate issued for the website.

![LB_nginx_part2_27a_confirm_secure_https_browser_access](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_27a_confirm_secure_https_browser_access.png)

![LB_nginx_part2_27b_confirm_secure_https_browser_access](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_27b_confirm_secure_https_browser_access.png)

![LB_nginx_part2_27c_confirm_secure_https_browser_access](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_27c_confirm_secure_https_browser_access.png)

6. Set up periodical renewal of your SSL/TLS certificate
By default, `LetsEncrypt` certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

Test the renewal command in dry-run mode

```sudo certbot renew --dry-run```

![LB_nginx_part2_28_check_certbot_renewal_command](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_28_check_certbot_renewal_command.png)

Best pracice is to have a scheduled job that runs renew command periodically. Configure a cronjob to run the command twice a day

Edit the crontab file

**crontab -e**

![LB_nginx_part2_29_enable_cronjob](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_29_enable_cronjob.png)

Add the following line to scheduled a job that runs renew command twice daily

```* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1```

![LB_nginx_part2_30_cronjob_frequency_setting](../Loadbalancer_nginx_images/Loadbalancer_nginx_Part2_images/LB_nginx_part2_30_cronjob_frequency_setting.png)

You can always change the interval of the cronjob if twice a day is too often by adjusting the schedule expression.

## Conclusion

This task successfully demonstrated the deployment of a highly available, scalable, and secure web infrastructure using an Nginx Layer 7 Load Balancer sitting in front of Red Hat Enterprise Linux backend web nodes [Nginx Load Balancing Methods and Features Supported by Nginx]. 

By decoupling traffic routing from backend execution, resolving network-attached NFS storage paths, and automating production-grade SSL/TLS termination via Let's Encrypt, the environment was engineered to meet modern enterprise reliability and data-in-transit security standards [Nginx Load Balancing Methods and Features Supported by Nginx, DevOps Troubleshooting Report: Certbot SSL/TLS Authentication Failure]. 

The final architecture validates critical DevOps core competencies in web server optimization, real-world network debugging, and public DNS management [Nginx Load Balancing Methods and Features Supported by Nginx, DevOps Troubleshooting Report: Certbot SSL/TLS Authentication Failure].

## References on cron configuration:

1. [Job Scheduling (cronjob/crontab) on Linux CentOS 8](https://www.youtube.com/watch?v=4g1i0ylvx3A)

2. [Online cron expression editor](https://crontab.guru/)