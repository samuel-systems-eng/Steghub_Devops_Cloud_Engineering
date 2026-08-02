# Steghub DevOps & Cloud Engineering Program

This repository tracks my hands-on cloud infrastructure assignments and system architecture deployments.

## Project 1: LAMP Stack Deployment on AWS EC2

A complete manual deployment of a high-availability **LAMP (Linux, Apache, MySQL, PHP)** web architecture baseline on an Amazon Web Services infrastructure.

### Technical Achievements
* **Infrastructure**: Provisioned an Ubuntu 26.04 LTS instance secured via custom RSA Key Pairs.
* **Network Firewalls**: Configured AWS Security Groups mapping precise inbound traffic rules for SSH (22), HTTP (80), and HTTPS (443).
* **Database Layer**: Deployed MySQL Server using secure authentication parameters.
* **Application Routing**: Configured an Apache Virtual Host mapping a local test domain (`www.projectlamp`) and extracted server metadata (public hostname and IP) via AWS IMDSv2.

## Project 2: LEMP Stack Deployment on AWS EC2

A hands-on implementation of a high-performance web stack (Linux, Nginx, MySQL, PHP) provisioned on an AWS EC2 instance. This project covers core web server configuration, database persistence, dynamic PHP-FPM routing, and deep troubleshooting of modern cloud security protocols.

### 📋 Task Checklist
- [x] **Task 0**: Provision and configure an Ubuntu EC2 Instance on AWS.
- [x] **Task 1**: Install and optimize the Nginx Web Server.
- [x] **Task 2**: Install and secure the MySQL Database Server.
- [x] **Task 3**: Install and configure PHP for dynamic server-side execution.
- [x] **Task 4**: Configure Nginx to route traffic to the PHP-FPM processor.
- [x] **Task 5**: Validate PHP-Nginx integration.
- [x] **Task 6**: Write server-side scripts to dynamically query data from the MySQL database.

---

### 🛠️ Advanced Troubleshooting & Engineering Deep Dives

During this implementation, two critical automation and security bottlenecks were identified and resolved regarding AWS Instance Metadata Service Version 2 (IMDSv2).

#### 1. Secure Bash Shortcut for Public IP Lookup
* **Issue**: Standard IMDSv1 calls (`curl 169.254.169.254/...`) returned blank outputs because AWS strictly enforces token-based **IMDSv2** to prevent Server-Side Request Forgery (SSRF) attacks. Additionally, long terminal configurations were failing due to SSH paste truncation bugs.
* **Resolution**: Instead of compromising infrastructure security by downgrading to IMDSv1, a highly efficient, short bash alias was automated into `.bashrc` utilizing external secure redirects.
* **Implementation**:
  ```bash
  sed -i '/alias get-ip=/d' ~/.bashrc && echo "alias get-ip='curl -sL ifconfig.me && echo'" >> ~/.bashrc && source ~/.bashrc
  ```

#### 2. Hybrid Hostname & IP Page Automation Script
* **Issue**: Automating landing page generation via terminal scripts resulted in the web application displaying a massive dump of AWS historical release dates (e.g., `2007-01-19, latest`). Line-wrap truncation cut the AWS metadata URL short, causing the terminal to call the API root directory index and dump it directly into `index.html`.
* **Resolution**: Developed a robust, short-line script that securely fetches a session token for internal hostname mapping via IMDSv2, while utilizing an ultra-short external hook for the public IP to eliminate line-wrap truncation risks.
* **Implementation**:
  ```bash
  {
    TOKEN=\$(curl -s -X PUT "169.254.169" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
    PUB_HOST=(curl -s -H "X-aws-ec2-metadata-token: TOKEN" 169.254.169)
    PUB_IP=\$(curl -sL ifconfig.me)

    echo "Hello LEMP from hostname \(PUB_HOST with public IP\)PUB_IP" | sudo tee /var/www/projectLEMP/index.html > /dev/null
  }
  ```

### 📈 Verified Output
Running a curl request against the public IP successfully validates that the dynamic stack routing, Nginx configuration, and metadata mappings are operational:
```bash
\$ curl http://54.91.39.35
Hello LEMP from hostname amazonaws.com with public IP 54.91.39.35
```

## Project 3: MERN Stack Deployment on AWS EC2

A complete deployment of a full-stack Todo application leveraging the MERN (MongoDB, Express, React, Node.js) architecture.

### Technical Achievements
- **Backend API**: Configured an Express.js backend on AWS running on port 5000 with environment variables dynamically managed by dotenvx.
- **Database Layer**: Established a live, validated cloud handshake with a MongoDB Atlas cluster utilizing custom Mongoose schemas.
- **Frontend Integration**: Maintained cross-origin communications using reverse routing proxies inside package.json to seamlessly bridge browser queries.
- **Full CRUD Testing**: Successfully validated database actions (`POST`, `GET`, `DELETE`) across the AWS firewall utilizing external Postman payloads.

### 🛠️ Real-World Troubleshooting & Engineering Win
- **The Problem:** The application backend crashed on boot during initial database connection handshakes, throwing a fatal `MongoParseError` because it contained obsolete configuration properties (`useNewUrlParser` and `useUnifiedTopology`). 
- **The Root Cause:** These legacy object parameters are deprecation landmines that completely break modern versions of Mongoose (v6+) and Node.js runtimes. 
- **The Engineering Fix:** Refactored the database initialization configuration inside `index.js` via terminal-based Vim, stripping out the obsolete parameters to streamline the connection syntax and leverage native, automated Mongoose asynchronous connection drivers.
- **Result:** Successfully secured a permanent, live cloud connection to the MongoDB Atlas database cluster without compromising application stability.


## Project 4: MEAN Stack Deployment on AWS EC2

A complete deployment of a full-stack Book register application leveraging the MEAN (MongoDB, Express, Angular, Node.js) architecture.

### Technical Achievements

•	Backend API: Configured a scalable Express.js backend on an AWS EC2 instance powered by a Node.js runtime environment.

•	Database Layer: Installed and configured a secure MongoDB database natively on the Ubuntu instance using custom Mongoose schemas for data validation.

•	Frontend Integration: Maintained cross-origin communications using reverse routing configurations to seamlessly bridge Angular browser queries to backend services.

•	Full CRUD Testing: Successfully validated database state actions (POST, GET, DELETE) live on the cloud network ecosystem.


### 🛠️ Real-World Troubleshooting & Engineering Win

Building locally is easy; deploying and debugging in the cloud is where real engineering happens. I encountered and resolved three critical production bottlenecks:

1️⃣ Frontend Isolation

•	The Problem: Frontend Isolation: The frontend layout was unreachable (Cannot GET /) because the “public” and “apps” asset folder was left outside the root repository directory.

•	The Engineering Fix: File Organization: Relocated the “apps” and “public” folders into the main application root (~/Books/) using the Linux “mv” command.

2️⃣ Syntax Typos

•	The Problem: data wouldn't display because a typo (an extra .) on line 3 of script.js completely broke the initialization of the AngularJS engine.

•	The Engineering Fix: Client Patch: Sanitized the “public/script.js” code into standard format (var app = angular.module('myApp', []);), and (app.controller…….) allowing the framework to safely communicate with MongoDB.


## Project 5: Client-Server Architecture Implementation on AWS

### 📝 Project Description
This project demonstrates the practical decoupling of database services from application clients within an enterprise cloud network.  
By deploying two distinct Ubuntu 24.04 LTS EC2 instances—one dedicated as a hardened MySQL Server and the other as a lightweight MySQL Client—I established a secure, low-latency multi-tier architecture. The core focus was managing isolated network environments, overriding default local loopback restrictions, and utilizing AWS internal VPC routing for secure data persistence.

### 🚀 Technical Achievements
* **Decoupled Architecture**: Successfully separated database infrastructure from client utilities, minimizing the attack surface of the database engine.
  
* **Network Hardening & Security**: Implemented least-privilege access rules within AWS Security Groups, locking down inbound TCP Port 3306 traffic exclusively to the client's internal private IP.
  
* **Targeted Identity Access Management (IAM)**: Overrode MySQL’s default local-only restrictions by provisioning a specialized remote user identity authenticated via `mysql_native_password`.
  
* **Network Daemon Configuration**: Reconfigured the server runtime socket binds (`bind-address = 0.0.0.0`) to accept non-local network interfaces without compromising local routing integrity.

### 🛠️ Real-World Troubleshooting & Engineering Wins
* **The "Host Not Allowed" (ERROR 1130) Triumph**: Encountered and resolved the classic remote handshake block where the server rejects the incoming EC2 internal hostname. I diagnosed this as a mismatch between MySQL’s internal host table and the AWS client private IP. Fixed it by mapping exact host wildcards (`'client'@'%'`) to ensure seamless scaling within the VPC.
  
* **Zero SSH-Tunneling Footprint**: Engineered a network path allowing direct MySQL utility client connections entirely over private AWS backplanes, removing the performance overhead and configuration complexity of SSH port-forwarding tunnels.
  
* **CRUD Integrity Validation**: Verified structural deployment from the remote client shell by building schemas (`test_db`), seeding data tables, and running complex queries across instances.


## Project 6 - Web Solution with WordPress in AWS

This repository documents the deployment of a secure, production-grade, two-tier WordPress web infrastructure on Amazon Web Services (AWS) using Red Hat Enterprise Linux (RHEL) 10. The architecture decouples the web presentation layer from the data management layer, using advanced logical volume management to optimize disk durability, isolation, and scalability.

### 🚀 Technical Achievements

*   **Multi-Tier Infrastructure Decoupling:** Implemented a highly secure, separated cluster utilizing two independent AWS EC2 instances (`t3.micro`) running RHEL 10 to segment presentation nodes from relational data stores.
*   **Advanced Linux LVM Architecture:** Provisioned six independent AWS Elastic Block Store (EBS) data volumes across both nodes, initialized modern GUID Partition Tables (GPT) utilizing native binaries, and established custom Volume Groups (`webdata-vg` and `database-vg`).
*   **Storage Segmentation & Resiliency:** Carved out dedicated, independent Logical Volumes (`apps-lv`, `db-lv`, and `logs-lv`) formatted with `ext4` filesystems to guarantee that application failures or log explosions never cascade to compromise core system data or database integrity.
*   **Secure Private Subnet Handshake:** Enforced strict firewall perimeters by modifying AWS Security Groups to accept inbound relational database network requests exclusively on Port 3306 originating from the Web Server's specific Private IP address.
*   **Automated AWS Mirror Integration:** Bypassed restrictive, mismatched entitlement network licenses by shifting package management streams (`dnf`/`yum`) to draw securely from local AWS Red Hat Update Infrastructure (RHUI) endpoints.

### 🛠️ Troubleshooting & Engineering Wins

*   **The RHEL 10 Version Pivot:** Faced a critical blockade where standard curriculum setup steps for `gdisk` and external Remi software streams failed due to RHEL 10's modern DNF5 deprecations. Successfully pivoted to modern, native `fdisk` implementations to write clean GPT schemes and leveraged RHEL 10's native upstream PHP 8.x engines to build a secure, zero-external-dependency software stack.
*   **The Emergency Mode Boot-Loop Rescue:** Encountered a system panic where a timing conflict during system boot locked the Database Server into an Emergency Mode state (2/3 AWS status checks passed) due to un-initialized LVM subsystems during mounting. Conquered this obstruction by force-stopping the server, detaching the 10GB root storage volume, and sideloading it onto the operational Web Server as a secondary data block.
*   **Configuration Hardening via Rescue Mounts:** Successfully mounted the broken root partition under a temporary `/mnt/rescue` pathway on the healthy web server, safely neutralized the blocking lines inside the raw `/etc/fstab` boot file, and re-attached the primary drive to its native slot.
*   **Immunization via Safety Flags:** Permanently immunized the entire multi-server cloud deployment against future virtualization initialization delays by appending the advanced `_netdev,nofail` operating system parameters to all persistent disk mount mappings.

## Project 7 - Devops Tooling Website Solution in AWS

This project implements a resilient, multi-node DevOps Tooling Website Solution utilizing a 3-tier architecture pattern within Amazon Web Services (AWS). 

By decoupling the web application layer, database services, and network storage targets, the design achieves high availability, eliminates deployment drift, and provides an optimized infrastructure ready for horizontal scaling behind an Application Load Balancer.

## Technical Achievements
* **Decoupled 3-Tier Layering:** Implemented an enterprise-grade separation of concerns across a 3-Node Web Server cluster, a dedicated storage layer, and an independent database instance.
* **Centralized Storage Fabric:** Architected a high-concurrency Network File System (NFS) to simultaneously serve shared application code assets (`/var/www/html`) and aggregate distributed system engine logs (`/var/log/httpd`).
* **Storage Optimization via LVM:** Configured Logical Volume Management (LVM) on raw EBS storage devices to dynamically manage partition sizing for application and system logging directories.
* **Modern DB Policy Management:** Applied secure access parameters using Subnet Mask Notation (`255.255.240.0`) to natively map network permissions within modern MySQL engines.
* **Boot-Safe System Persistence:** Hardened system mount structures against critical network drops by appending `_netdev` and `nofail` parameter rules to the Linux `/etc/fstab` filesystem configurations.

## Troubleshooting and Engineering Wins
* **Overcoming Modern Database Constraints:** Resolved an `Error 1524 (Plugin not loaded)` failure caused by modern MySQL versions deprecating native password extensions. Remediated the block by transitioning to the secure `caching_sha2_password` mechanism while replacing unsupported CIDR formatting with explicit network subnet masks.
* **Bypassing Network Mount Device Locks:** Identified and cleared `Device or resource busy` locks during log directory migrations on multi-node instances by orchestrating deliberate service teardowns, forcing clean network unmounts, and resetting folder structures with precise `apache:apache` context permissions.
* **Mitigating SELinux Silent Failures:** Addressed an Apache `403 Forbidden` response and an initial service startup crash triggered by default RHEL SELinux policies blocking remote network stream writes. Safely transitioned the security module into a permanent `permissive` mode, satisfying data access needs while preserving underlying OS stability.

## Project 8 - "Load Balancer Solution with Apache"

This project demonstrates the implementation of a highly available, fault-tolerant web infrastructure utilizing an Apache HTTP Server as a Layer 7 Application Load Balancer. 

The system routes inbound client traffic across a scalable cluster of Red Hat Enterprise Linux backend web workers, leveraging a centralized Network File System (NFS) for shared data storage and local DNS mappings for streamlined cluster communications.

### Technical Achievements

**Layer 7 Load Balancing**: Implemented an Ubuntu-based gateway utilizing mod_proxy_balancer to handle advanced content-based request routing.

**Stateless Web Architecture**: Deployed a cluster of RHEL web workers serving identical application code assets simultaneously from a single central NFS mount.

**Localized Log Isolation**: Configured Apache logging pipelines to write execution streams locally to individual server drives, preserving network storage bandwidth.

**Local Name Resolution**: Established custom internal private DNS naming mappings within the /etc/hosts subsystem to eliminate dependency on raw IP address records.

### Troubleshooting and Engineering Wins

**Resolved Zombie Network Mounts**: Isolated and cleared a hidden duplicate mount gridlock spanning /`var/www` and `/var/www/html` by executing localized lazy unmount instructions (`umount -l`).

**Fixed App Layer Processing Handshake**: Remediated a 502 Upstream Proxy Error by synchronizing the RHEL php-fpm processing daemon with Apache's configuration match structures.Sanatized Configuration 

**Typographical Errors**: Mitigated server crash behaviors by targeting and replacing rich-text smart quote configurations with compliant straight ASCII formatting characters.


---

## Repository Structure
* **[/Prerequisites](./00_Prerequisites)**: Foundational environment assets, self-study modules, and baseline environment setups.
* **[/LAMP_STACK](./01_LAMP_Stack)**: Configuration LAMP profiles, virtual host site rules, and deployment code scripts.
* **[/LEMP_STACK](./02_LEMP_Stack)**: Configuration LEMP profiles, server blocks and site rules, and deployment code scripts.
* **[/MERN_Stack](./03_MERN_Stack)**: Configuration MERN profiles, Mongoose schemas and database models, and deployment application code scripts.
* **[/MEAN_Stack](./04_MEAN_Stack)**: Configuration of MEAN profiles, deploy Mongoose schemas and database models, and sanitize code scripts.
* **[/CLIENT_SERVER_ARCHITECTURE](./05_CLIENT_SERVER_ARCHITECTURE)**: Configuration of decoupled multi-tier AWS EC2 environments, deployment of secure MySQL network daemons, and troubleshooting of host-based authentication blocks.
* **[/WEB_SOLUTION_WITH_WORDPRESS](./06_WEB_SOLUTION_WITH_WORDPRESS)**: Implementation of web solution with wordpress, detailed implementation files, networking parameters, and core storage mapping scripts for this web deployment can be found in the active project directory.
* **[/DEVOPS_TOOLING_WEBSITE_SOLUTION](./07_DEVOPS_TOOLING_WEBSITE_SOLUTION)**: 
The complete codebase, database schemas, and configuration scripts can be explored directly in the main project tree.
* **[/LOADBALANCER_SOLUTION_WITH_APACHE](./08_LOADBALANCER_WITH_APACHE)**: 
The complete folder hierarchy, configuration scripts, and documentation files can be accessed directly inside the StegHub DevOps Cloud Engineering Repository.