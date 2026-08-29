# AWS RDS and EC2 MySQL Integration Lab

This repository documents the deployment and automated integration of a private Amazon RDS MySQL instance with an Amazon EC2 web application host using Infrastructure as Code principles and secure connectivity practices.

---

## 🛠 Project Architecture Overview

* **Database Tier:** Private Amazon RDS MySQL instance (`nautilus-rds`).
* **Compute Tier:** Amazon EC2 instance (`nautilus-ec2`) serving a PHP application.
* **Management Host:** Jump/Client host (`aws-client`) for SSH access and script deployments.

---

## 🚀 Key Tasks & Implementation Details

### 1. Database Provisioning & Network Security
* **RDS Configuration:** Provisioned a MySQL v8.4.5 instance (`db.t3.micro`, `gp2` storage, 5 GiB) named `nautilus-rds` with initial database `nautilus_db`.
* **Private Isolation:** Set RDS to non-publicly accessible.
* **Security Group Rules:**
  * **RDS Inbound:** Restricted MySQL port `3306` to accept connections exclusively from the EC2 Security Group.
  * **EC2 Inbound:** Opened HTTP port `80` to public traffic (`0.0.0.0/0`) and SSH port `22` for host administration.

### 2. Passwordless SSH Key Deployment
* Generated an RSA keypair on the `aws-client` host (`/root/.ssh/id_rsa`).
* Configured the `authorized_keys` file on `nautilus-ec2` via AWS EC2 Instance Connect to enable seamless, passwordless SSH and SCP workflows.

### 3. Application Deployment & Database Connection
* Transferred `index.php` from `aws-client` to `/var/www/html/` on the EC2 instance via SCP.
* Configured PDO/MySQL connection strings with the RDS endpoint credentials:
  * **Host:** `nautilus-rds.<id>.<region>.rds.amazonaws.com`
  * **User:** `nautilus_admin`
  * **Database:** `nautilus_db`

### 4. Apache Web Server Optimization
* Cleared default Apache landing pages (`index.html`) to ensure `index.php` serves as the primary root index.
* Restarted `httpd`/`apache2` web services to handle incoming HTTP requests properly.

---

## 🎯 Verification

Accessing the public IP of the EC2 instance directly in the browser yields:

```text
Connected successfully
