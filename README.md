# Deploying a Two-Tier WordPress Application on Ubuntu 22 using AWS EC2 and RDS

This guide will walk you through the steps to deploy a two-tier WordPress application on Ubuntu 22 using AWS EC2 for the web server and AWS RDS for the database.

## Prerequisites

1. An AWS account with appropriate permissions.
2. Basic familiarity with AWS services.
3. A domain pointing to the ec2's ip

## Step 1: Launch the EC2 Instance

1. Log in to your AWS Management Console.
2. Navigate to EC2 Dashboard.
3. Click "Launch Instance" and choose an Ubuntu 22 AMI.
4. Select an instance type suitable for your workload.
5. Configure instance details (VPC, subnet, security group, etc.).
6. Add storage as needed.
7. Configure the security group to allow HTTP (port 80), HTTPS (port 443) and SSH (port 22) traffic.
8. Review and launch the instance, selecting your SSH key pair.
9. Connect to the instance using SSH: `ssh -i /path/to/your/key.pem ubuntu@your-instance-ip`.

## Step 2: Launch the RDS Instance

1. Navigate to the RDS Dashboard in AWS.
2. Click "Create database" and choose MySQL.
3. Configure DB settings (instance type, storage, username, password, etc.).
4. Configure advanced settings, including VPC, subnet, and security group.
5. Allow the rds security group to receive inbound request for mysql from ec2 instance's security group.
6. Create the database instance.
7. Copy the credentials for further use.


## Step 3: Set Up LAMP Stack (EC2)

1. Update and upgrade packages: `sudo apt update && sudo apt upgrade -y`.
2. Install Apache: `sudo apt install apache2 -y`.
3. Install php and required packages: `sudo apt install -y php libapache2-mod-php php-mysql  php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip`.
4. Create Virtual Host for website:
  - `sudo mkdir /var/www/your_domain`
  - `sudo chown -R $USER:$USER /var/www/your_domain`
  - Create the configuration file for your domain `sudo nano /etc/apache2/sites-available/your_domain.conf`.
  - Paste this configuration, and modify the required values
  ```
  <VirtualHost *:80>
      ServerName your_domain
      ServerAlias www.your_domain 
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/your_domain
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
      <Directory /var/www/your_domain>
        AllowOverride All
      </Directory>
  </VirtualHost>
```
  - Press `Ctrl+O` and `Enter` to save, and `Ctrl+X` to exit.
  - Enable our site using `sudo a2ensite your_domain`.
  - (optional) disable the default apache site using `sudo a2dissite 000-default`.
  - Enable the rewrite module to be able to use Wordpress permalink feature `sudo a2enmod rewrite`.
  - Make sure the configuration is correct `sudo apache2ctl configtest`.
  - Reload apache `sudo systemctl reload apache2`.

## Step 4: Check MySQL connection and create required database

1. Install MySQL Client: `sudo apt install mysql-client -y`.
2. Login to mysql using credentials you got from RDS `mysql -h <hostname-from-rds> -u <rds-username> -p`
3. Enter the password. Check if the connection is successful.
4. Create database `create database wordpress;`
5. Exit mysql using `exit`.

## Step 5: Download Wordpress

1. Move to tmp directory `cd /tmp`.
2. Download the latest version of wordpress `curl -O https://wordpress.org/latest.tar.gz`.
3. Extract the compressed file `tar xzvf latest.tar.gz`.
4. Create an empty `.htaccess` file to be used by wordpress and apache `touch /tmp/wordpress/.htaccess`.
5. Create the `upgrade` directory so that wordpress can easily update on it's own `mkdir /tmp/wordpress/wp-content/upgrade`.
6. Copy the content of wordpress to your document root `sudo cp -a /tmp/wordpress/. /var/www/your_domain`.
7. Adjust the ownership of the directory and their folders and files
```
sudo chown -R www-data:www-data /var/www/your_domain
sudo find /var/www/your_domain/ -type d -exec chmod 750 {} \;
sudo find /var/www/your_domain/ -type f -exec chmod 640 {} \;
```
8. Now your website should be available on specified domain

## Step 6: Setup SSL Certificate for HTTPS

1. Install Certbot `sudo apt install certbot python3-certbot-apache`.
2. Run certbot and provide all the required options `sudo certbot --apache`.

> Congratulations! You've successfully deployed a two-tier WordPress application on Ubuntu 22 using AWS EC2 and RDS.

