# sales-engineer-portfolio

Introduction
This tutorial guides you through a hands-on project to deploy and secure a WordPress site on an Ubuntu EC2 instance. You will:
Launch and connect to an EC2 server


Configure a non‑root admin user and lock down SSH


Install and refine the Apache + MySQL + PHP (LAMP) stack


Harden the server with firewall and intrusion protection tools


Create recoverable backups using AMIs and EBS snapshots
Day 1: Launch & Connect to EC2
Launch an EC2 instance


AMI: Ubuntu Server 20.04 LTS


Instance type: t2.micro (Free Tier)


Security group: Open ports 22 (SSH), 80 (HTTP), 443 (HTTPS)


Key pair: intern-project.pem


Move and secure your key locally

mv ~/Desktop/intern-project.pem ~/.ssh/
chmod 400 ~/.ssh/intern-project.pem

Connect via SSH

ssh -i ~/.ssh/intern-project.pem ubuntu@<EC2_PUBLIC_IP>

Verify your user and kernel:

whoami
uname -a

Update the OS

 sudo apt update && sudo apt upgrade -y
Day 2: Create Admin User & Harden SSH
Add a non‑root sudo user

sudo adduser isaiah01
sudo usermod -aG sudo isaiah01

Generate an SSH key pair (local)

ssh-keygen -t rsa -b 4096 -f ~/.ssh/intern-project

Install the public key on EC2

ssh-copy-id -i ~/.ssh/intern-project.pub isaiah01@<EC2_PUBLIC_IP>

Test login as new user

ssh -i ~/.ssh/intern-project isaiah01@<EC2_PUBLIC_IP>

Disable root and password SSH logins

# Edit SSH config
sudo /etc/ssh/sshd_config → Change “PermitRootLogin yes” to “PermitRootLogin no” then Change “PasswordAuthentication yes” to “PasswordAuthentication no”

#Restart SSH to apply
sudo systemctl restart sshd

Day 3: Install & Refine Apache
Install Apache

sudo apt install -y apache2

Verify and enable on boot

sudo systemctl status apache2
sudo systemctl enable apache2

Enable URL rewrites for WordPress permalinks

sudo a2enmod rewrite
sudo systemctl reload apache2
Day 4: Full LAMP & WordPress Deployment
Install MySQL and PHP

sudo apt install -y mysql-server php libapache2-mod-php php-mysql

Secure MySQL

sudo mysql_secure_installation

Create WordPress database and user

sudo mysql -u root -p << ‘EOF’

CREATE DATABASE wordpress_db DEFAULT CHARACTER SET utf8mb4;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongPassword!';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
EOF


Download and deploy WordPress


# Change to the Temporary directory
cd /tmp

# Download the latest WordPress compressed file
wget https://wordpress.org/latest.tar.gz

# Extract the compressed WordPress files
tar xzvf latest.tar.gz

#Move Files to Web Server Root
sudo mv wordpress/* /var/www/html

# Change ownership of WordPress files to the web server’s public directory
sudo chown -R www-data:www-data /var/www/html

# Set directory Permissions
sudo find /var/www/html -type d -exec chmod 755 {} \;

# Set File Permissions
sudo find /var/www/html -type f -exec chmod 644 {} \;

Configure WordPress


# Navigate to WordPress Directory
cd /var/www/html

# Create WordPress Configuration File
sudo cp wp-config-sample.php wp-config.php

# Update Database Credentials
sudo sed -i "s/database_name_here/wordpress_db/; s/username_here/wpuser/; s/password_here/StrongPassword!/" wp-config.php

# Restart Web Server
sudo systemctl restart apache2
Verify site
Open http://<EC2_PUBLIC_IP> in your browser and complete the setup wizard.
Day 5: Backup & Cleanup
Step 5.1: Create an AMI Backup (AWS Console)
Navigate to EC2 → Instances, select your instance.


Click Actions → Image → Create Image.


Name: wordpress-backup-day5, click Create Image.


Confirm in EC2 → AMIs that the image becomes available.


Step 5.2: Take an EBS Snapshot (AWS Console)
Go to Elastic Block Store → Volumes, select your volume.


Click Actions → Create Snapshot, name wp-volume-snapshot-day5, click Create Snapshot.


Confirm in Snapshots that it’s completed.
