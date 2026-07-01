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


---

## Repository Structure
* **[/Prerequisites](./00_Prerequisites)**: Foundational environment assets, self-study modules, and baseline environment setups.
* **[/LAMP_STACK](./01_LAMP_Stack)**: Configuration LAMP profiles, virtual host site rules, and deployment code scripts.
* **[/LEMP_STACK](./02_LEMP_Stack)**: Configuration LEMP profiles, server blocks and site rules, and deployment code scripts.
* **[/MERN_Stack](./03_MERN_Stack)**: Configuration MERN profiles, Mongoose schemas and database models, and deployment application code scripts.
* **[/MEAN_Stack](./04_MEAN_Stack)**: Configuration of MEAN profiles, deploy Mongoose schemas and database models, and sanitize code scripts.