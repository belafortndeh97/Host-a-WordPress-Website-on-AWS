![Alt text](/2._Host_a_WordPress_Website_on_AWS.png)

Host a WordPress Website on AWS

## Overview

This project involves hosting a WordPress website on AWS using various services to ensure high availability, scalability, fault tolerance, and security. The infrastructure is designed across multiple Availability Zones (AZs) in a Virtual Private Cloud (VPC) to enhance reliability. The project utilizes the following AWS resources:

1. **Virtual Private Cloud (VPC):**
   - Configured a VPC with both public and private subnets across two different availability zones for enhanced availability and fault tolerance.

2. **Internet Gateway:**
   - Deployed an Internet Gateway to facilitate connectivity between VPC instances and the wider Internet.

3. **Security Groups:**
   - Established Security Groups as a network firewall mechanism to control inbound and outbound traffic.

4. **Availability Zones:**
   - Leveraged two Availability Zones to enhance system reliability and fault tolerance.

5. **Public Subnets:**
   - Utilized Public Subnets for infrastructure components like the NAT Gateway and Application Load Balancer.

6. **EC2 Instance Connect Endpoint:**
   - Implemented EC2 Instance Connect Endpoint for secure connections to assets within both public and private subnets.

7. **Private Subnets:**
   - Positioned web servers (EC2 instances) within Private Subnets for enhanced security.

8. **NAT Gateway:**
   - Enabled instances in both the private Application and Data subnets to access the Internet via the NAT Gateway.

9. **Website Hosting on EC2 Instances:**
   - Hosted the WordPress website on EC2 Instances.

10. **Application Load Balancer (ALB) and Auto Scaling:**
    - Employed an Application Load Balancer and a target group for evenly distributing web traffic to an Auto Scaling Group of EC2 instances across multiple Availability Zones.

11. **Auto Scaling Group:**
    - Utilized an Auto Scaling Group to automatically manage EC2 instances, ensuring website availability, scalability, fault tolerance, and elasticity.

12. **Version Control with GitHub:**
    - Stored web files on GitHub for version control and collaboration.

13. **Certificate Manager:**
    - Secured application communications using AWS Certificate Manager for SSL/TLS certificates.

14. **Simple Notification Service (SNS):**
    - Configured Simple Notification Service (SNS) to alert about activities within the Auto Scaling Group.

15. **Domain Name and Route 53:**
    - Registered the domain name and set up a DNS record using Route 53.

16. **Elastic File System (EFS):**
    - Used Amazon EFS for a shared file system, ensuring consistent file access across multiple instances.

17. **Relational Database Service (RDS):**
    - Utilized Amazon RDS for hosting the database.

## Deployment Scripts

Script 1: WordPress Installation

#!/bin/bash

# Update the software packages on the EC2 instance
sudo yum update -y

# Create the HTML directory
sudo mkdir -p /var/www/html

# Environment variable for EFS DNS name
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-2.amazonaws.com

# Mount the EFS to the HTML directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server and start it
sudo yum install -y httpd

sudo systemctl enable httpd

sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y \

php \

php-cli \

php-cgi \

php-curl \

php-mbstring \

php-gd \

php-mysqlnd \

php-gettext \

php-json \

php-xml \

php-fpm \

php-intl \

php-zip \

php-bcmath \

php-ctype \

php-fileinfo \

php-openssl \

php-pdo \

php-tokenizer

# Install MySQL 8 repository and server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm

sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm

sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023

sudo dnf repolist enabled | grep "mysql.*-community.*"

sudo dnf install -y mysql-community-server

# Start and enable the MySQL server
sudo systemctl start mysqld

sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user

sudo chown -R ec2-user:apache /var/www

sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;

sudo find /var/www -type f -exec sudo chmod 0664 {} \;

sudo chown apache:apache -R /var/www/html

# Download WordPress files
wget https://wordpress.org/latest.tar.gz

tar -xzf latest.tar.gz

sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# Restart the webserver
sudo systemctl restart httpd


Script 2: EC2 Instance Configuration for Autoscaling Group Launch Template

#!/bin/bash

# Update the software packages on the EC2 instance
sudo yum update -y

# Install Apache web server and start it
sudo yum install -y httpd

sudo systemctl enable httpd

sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y \

php \

php-cli \

php-cgi \

php-curl \

php-mbstring \

php-gd \

php-mysqlnd \

php-gettext \

php-json \

php-xml \

php-fpm \

php-intl \

php-zip \

php-bcmath \

php-ctype \

php-fileinfo \

php-openssl \

php-pdo \

php-tokenizer

# Install MySQL 8 repository and server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm

sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm

sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023

sudo dnf repolist enabled | grep "mysql.*-community.*"

sudo dnf install -y mysql-community-server

# Start and enable the MySQL server
sudo systemctl start mysqld

sudo systemctl enable mysqld

# Environment variable for EFS DNS name
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-2.amazonaws.com

# Mount the EFS to the HTML directory
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart the webserver
sudo systemctl restart httpd

