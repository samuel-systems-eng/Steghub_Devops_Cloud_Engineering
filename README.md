# Steghub DevOps & Cloud Engineering Program

This repository tracks my hands-on cloud infrastructure assignments and system architecture deployments.

## Project 1: LAMP Stack Deployment on AWS EC2

A complete manual deployment of a high-availability **LAMP (Linux, Apache, MySQL, PHP)** web architecture baseline on an Amazon Web Services infrastructure.

### Technical Achievements
* **Infrastructure**: Provisioned an Ubuntu 26.04 LTS instance secured via custom RSA Key Pairs.
* **Network Firewalls**: Configured AWS Security Groups mapping precise inbound traffic rules for SSH (22), HTTP (80), and HTTPS (443).
* **Database Layer**: Deployed MySQL Server using secure authentication parameters.
* **Application Routing**: Configured an Apache Virtual Host mapping a local test domain (`www.projectlamp`) and extracted server metadata (public hostname and IP) via AWS IMDSv2.

---

## Repository Structure
* **[/Prerequisites](./Prerequisites)**: Foundational environment assets, self-study modules, and baseline environment setups.
* **`/LAMP_STACK`**: Configuration profiles, virtual host site rules, and deployment code scripts.
