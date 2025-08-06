Project: Secure WordPress Deployment on Ubuntu EC2This tutorial guides you through a hands-on project to deploy and secure a WordPress site on an Ubuntu EC2 instance. You will:Launch and connect to an EC2 serverConfigure a non-root admin user and lock down SSHInstall and refine the Apache + MySQL + PHP (LAMP) stackHarden the server with firewall and intrusion protection toolsCreate recoverable backups using AMIs and EBS snapshotsDay 1: Launch & Connect to EC2Launch an EC2 instanceAMI: Ubuntu Server 20.04 LTSInstance type: t2.micro (Free Tier)Security group: Open ports 22 (SSH), 80 (HTTP), 443 (HTTPS)Key pair: intern-project.pemMove and secure your key locallymv ~/Desktop/intern-project.pem ~/.ssh/
chmod 400 ~/.ssh/intern-project.pem
Connect via SSHssh -i ~/.ssh/intern-project.pem ubuntu@<EC2_PUBLIC_IP>
Verify your user and kernel:whoami
uname -a
Update the OSsudo apt update && sudo apt upgrade -y
Day 2: Create Admin User & Harden SSHAdd a non-root sudo usersudo adduser isaiah01
sudo usermod -aG sudo isaiah01
Generate an SSH key pair (local)ssh-keygen -t rsa -b 4096 -f ~/.ssh/intern-project
Install the public key on EC2ssh-copy-id -i ~/.ssh/intern-project.pub isaiah01@<EC2_PUBLIC_IP>
Test login as new userssh -i ~/.ssh/intern-project isaiah01@<EC2_PUBLIC_IP>
Disable root and password SSH logins# Edit SSH config
sudo /etc/ssh/sshd_config
→ Change “PermitRootLogin yes” to “PermitRootLogin no” then Change “PasswordAuthentication yes” to “PasswordAuthentication no”# Restart SSH to apply
sudo systemctl restart sshd
Day 3: Install & Refine ApacheInstall Apachesudo apt install -y apache2
Verify and enable on bootsudo systemctl status apache2
sudo systemctl enable apache2
Enable URL rewrites for WordPress permalinkssudo a2enmod rewrite
sudo systemctl reload apache2
Day 4: Full LAMP & WordPress DeploymentInstall MySQL and PHPsudo apt install -y mysql-server php libapache2-mod-php php-mysql
Secure MySQLsudo mysql_secure_installation
Create WordPress database and usersudo mysql -u root -p << ‘EOF’

CREATE DATABASE wordpress_db DEFAULT CHARACTER SET utf8mb4;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongPassword!';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
EOF
Download and deploy WordPress# Change to the Temporary directory
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
Configure WordPress# Navigate to WordPress Directory
cd /var/www/html

# Create WordPress Configuration File
sudo cp wp-config-sample.php wp-config.php

# Update Database Credentials
sudo sed -i "s/database_name_here/wordpress_db/; s/username_here/wpuser/; s/password_here/StrongPassword!/" wp-config.php

# Restart Web Server
sudo systemctl restart apache2
Verify siteOpen http://<EC2_PUBLIC_IP> in your browser and complete the setup wizard.Day 5: Backup & CleanupStep 5.1: Create an AMI Backup (AWS Console)Navigate to EC2 → Instances, select your instance.Click Actions → Image → Create Image.Name: wordpress-backup-day5, click Create Image.Confirm in EC2 → AMIs that the image becomes available.Step 5.2: Take an EBS Snapshot (AWS Console)Go to Elastic Block Store → Volumes, select your volume.Click Actions → Create Snapshot, name wp-volume-snapshot-day5, click Create Snapshot.Confirm in Snapshots that it’s completed.
