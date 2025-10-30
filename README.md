# PHP Website Deployment Guide with Apache, MySQL & phpMyAdmin

![Author](https://img.shields.io/badge/Author-Jaydip%20Khunt-blue)
![Date](https://img.shields.io/badge/Date-October%202025-green)
![Guide](https://img.shields.io/badge/Type-Deployment%20Guide-orange)

![PHP](https://img.shields.io/badge/PHP-8.2+-777BB4?style=for-the-badge&logo=php&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-2.4-D22128?style=for-the-badge&logo=apache&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![phpMyAdmin](https://img.shields.io/badge/phpMyAdmin-5.2-6C78AF?style=for-the-badge&logo=phpmyadmin&logoColor=white)

A comprehensive guide for deploying PHP websites on a Virtual Private Server (VPS) using the LAMP stack (Linux, Apache, MySQL, PHP) with phpMyAdmin for database management.

## Table of Contents
- [Prerequisites](#prerequisites)
- [System Preparation](#system-preparation)
- [Apache Web Server Installation](#apache-web-server-installation)
- [MySQL Database Server Installation](#mysql-database-server-installation)
- [PHP Installation & Configuration](#php-installation--configuration)
- [phpMyAdmin Installation & Setup](#phpmyadmin-installation--setup)
- [Website Deployment](#website-deployment)
- [Virtual Hosts Configuration](#virtual-hosts-configuration)
- [SSL Certificate Installation](#ssl-certificate-installation)
- [Security Hardening](#security-hardening)
- [Performance Optimization](#performance-optimization)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### System Requirements
- **Operating System**: Ubuntu 18.04+ or Debian 10+
- **RAM**: Minimum 1GB (2GB+ recommended)
- **Storage**: At least 10GB available space
- **Network**: Public IP address
- **Access**: Root or sudo privileges

### Before You Begin
- VPS server with Ubuntu/Debian
- SSH access to your server
- Domain name (optional but recommended)
- Basic command line knowledge

---

## System Preparation

### Initial Server Setup

1. **Connect to your VPS**
```bash
ssh root@YOUR_SERVER_IP
```

2. **Update system packages**
```bash
sudo apt update && sudo apt upgrade -y
```

3. **Create a non-root user (recommended)**
```bash
# Create user
sudo adduser lampuser

# Add to sudo group
sudo usermod -aG sudo lampuser

# Switch to new user
su - lampuser
```

### Configure Firewall
```bash
# Install UFW firewall
sudo apt install ufw

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (important!)
sudo ufw allow ssh
sudo ufw allow 22

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

---

## Apache Web Server Installation

### Install Apache
```bash
# Install Apache web server
sudo apt install apache2 -y

# Start and enable Apache
sudo systemctl start apache2
sudo systemctl enable apache2

# Check Apache status
sudo systemctl status apache2
```

### Configure Firewall for Apache
```bash
# Allow Apache through firewall
sudo ufw allow 'Apache Full'

# Or allow specific ports
sudo ufw allow 80    # HTTP
sudo ufw allow 443   # HTTPS

# Reload firewall
sudo ufw reload
```

### Verify Apache Installation
Open your web browser and navigate to:
```
http://YOUR_SERVER_IP
```
You should see the Apache default page.

### Apache Directory Structure
```bash
# Main configuration file
/etc/apache2/apache2.conf

# Sites available
/etc/apache2/sites-available/

# Sites enabled
/etc/apache2/sites-enabled/

# Document root
/var/www/html/

# Log files
/var/log/apache2/
```

### Basic Apache Commands
```bash
# Start Apache
sudo systemctl start apache2

# Stop Apache
sudo systemctl stop apache2

# Restart Apache
sudo systemctl restart apache2

# Reload configuration
sudo systemctl reload apache2

# Check configuration syntax
sudo apache2ctl configtest
```

---

## MySQL Database Server Installation

### Install MySQL Server
```bash
# Install MySQL
sudo apt install mysql-server -y

# Start and enable MySQL
sudo systemctl start mysql
sudo systemctl enable mysql

# Check MySQL status
sudo systemctl status mysql
```

### Secure MySQL Installation
```bash
# Run security script
sudo mysql_secure_installation
```

**Follow the prompts:**
1. **Set root password**: Choose a strong password
2. **Remove anonymous users**: Y
3. **Disallow root login remotely**: Y
4. **Remove test database**: Y
5. **Reload privilege tables**: Y

### Configure MySQL
```bash
# Login to MySQL
sudo mysql -u root -p

# Create database
CREATE DATABASE your_website_db;

# Create user
CREATE USER 'website_user'@'localhost' IDENTIFIED BY 'strong_password';

# Grant privileges
GRANT ALL PRIVILEGES ON your_website_db.* TO 'website_user'@'localhost';

# Flush privileges
FLUSH PRIVILEGES;

# Exit MySQL
EXIT;
```

### MySQL Configuration File
```bash
# Edit MySQL configuration
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

**Basic optimizations:**
```ini
[mysqld]
# Basic settings
max_connections = 100
innodb_buffer_pool_size = 256M
query_cache_type = 1
query_cache_size = 64M

# Security
bind-address = 127.0.0.1
skip-networking = false
```

Restart MySQL after changes:
```bash
sudo systemctl restart mysql
```

---

## PHP Installation & Configuration

### Install PHP and Extensions
```bash
# Install PHP and common extensions
sudo apt install php php-mysql php-cli php-common php-curl php-gd php-mbstring php-xml php-zip php-json php-xmlrpc php-soap php-intl php-bcmath -y

# Install additional modules for web development
sudo apt install libapache2-mod-php php-opcache php-readline php-imagick -y
```

### Verify PHP Installation
```bash
# Check PHP version
php -v

# Check installed modules
php -m
```

### Configure PHP
```bash
# Edit PHP configuration
sudo nano /etc/php/8.1/apache2/php.ini
```

**Important PHP settings:**
```ini
# Memory and execution limits
memory_limit = 256M
max_execution_time = 300
max_input_time = 300
post_max_size = 64M
upload_max_filesize = 64M

# Error reporting (disable in production)
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log

# Security settings
allow_url_fopen = Off
allow_url_include = Off
expose_php = Off

# Session settings
session.cookie_httponly = 1
session.cookie_secure = 1
session.use_strict_mode = 1
```

### Test PHP
Create a test file:
```bash
sudo nano /var/www/html/info.php
```

Add PHP info code:
```php
<?php
phpinfo();
?>
```

Visit `http://YOUR_SERVER_IP/info.php` to verify PHP is working.

**Important:** Remove this file after testing:
```bash
sudo rm /var/www/html/info.php
```

### Restart Apache
```bash
sudo systemctl restart apache2
```

---

## phpMyAdmin Installation & Setup

### Install phpMyAdmin
```bash
# Install phpMyAdmin and dependencies
sudo apt install phpmyadmin php-mbstring php-zip php-gd php-json php-curl -y
```

### Installation Configuration
During installation, you'll be prompted to:

1. **Select web server**: Choose **Apache2** (use spacebar to select)
2. **Configure database**: Select **Yes** to configure with dbconfig-common
3. **Set password**: Create a strong password for phpMyAdmin database user

### Enable Apache Modules
```bash
# Enable required modules
sudo phpenmod mbstring

# Restart Apache
sudo systemctl restart apache2
```

### Access phpMyAdmin
Open your browser and navigate to:
```
http://YOUR_SERVER_IP/phpmyadmin
```

### Create MySQL User for phpMyAdmin
```bash
# Login to MySQL as root
sudo mysql -u root -p
```

```sql
-- Create administrative user
CREATE USER 'phpadmin'@'localhost' IDENTIFIED BY 'strong_admin_password';

-- Grant all privileges
GRANT ALL PRIVILEGES ON *.* TO 'phpadmin'@'localhost' WITH GRANT OPTION;

-- Flush privileges
FLUSH PRIVILEGES;

-- Exit
EXIT;
```

### Secure phpMyAdmin Installation

1. **Change access URL** (optional but recommended):
```bash
# Edit Apache configuration
sudo nano /etc/apache2/conf-available/phpmyadmin.conf
```

Add an alias:
```apache
Alias /secure-admin /usr/share/phpmyadmin
```

2. **Restrict access by IP**:
```apache
<Directory /usr/share/phpmyadmin>
    Options SymLinksIfOwnerMatch
    DirectoryIndex index.php

    # Restrict access to specific IP
    <RequireAll>
        Require ip YOUR_IP_ADDRESS
        Require ip 127.0.0.1
    </RequireAll>
</Directory>
```

3. **Enable the configuration**:
```bash
sudo a2enconf phpmyadmin
sudo systemctl restart apache2
```

---

## Website Deployment

### Prepare Website Directory
```bash
# Create website directory
sudo mkdir -p /var/www/your-website.com

# Set ownership
sudo chown -R $USER:$USER /var/www/your-website.com

# Set permissions
sudo chmod -R 755 /var/www/your-website.com
```

### Upload Website Files

**Method 1: Using Git**
```bash
cd /var/www/your-website.com
git clone https://github.com/yourusername/your-php-website.git .
```

**Method 2: Using SCP/SFTP**
```bash
# From local machine
scp -r /path/to/your/website/* user@YOUR_SERVER_IP:/var/www/your-website.com/
```

**Method 3: Using wget/curl**
```bash
# Download and extract
cd /var/www/your-website.com
wget https://example.com/your-website.zip
unzip your-website.zip
rm your-website.zip
```

### Database Import
```bash
# Import SQL file
mysql -u website_user -p your_website_db < database_backup.sql

# Or use phpMyAdmin web interface
# Go to http://YOUR_SERVER_IP/phpmyadmin
# Select database and use Import tab
```

### Configure Website Database Connection
Edit your website's config file (usually `config.php` or `wp-config.php`):
```php
<?php
// Database configuration
define('DB_HOST', 'localhost');
define('DB_NAME', 'your_website_db');
define('DB_USER', 'website_user');
define('DB_PASS', 'strong_password');

// Database connection
$mysqli = new mysqli(DB_HOST, DB_USER, DB_PASS, DB_NAME);

// Check connection
if ($mysqli->connect_error) {
    die('Connection failed: ' . $mysqli->connect_error);
}
?>
```

---

## Virtual Hosts Configuration

### Create Virtual Host Configuration
```bash
# Create configuration file
sudo nano /etc/apache2/sites-available/your-website.com.conf
```

**Basic Virtual Host:**
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@your-website.com
    ServerName your-website.com
    ServerAlias www.your-website.com
    DocumentRoot /var/www/your-website.com

    # Logging
    ErrorLog ${APACHE_LOG_DIR}/your-website.com_error.log
    CustomLog ${APACHE_LOG_DIR}/your-website.com_access.log combined

    # Directory permissions
    <Directory /var/www/your-website.com>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Enable Virtual Host
```bash
# Enable site
sudo a2ensite your-website.com.conf

# Disable default site (optional)
sudo a2dissite 000-default.conf

# Test configuration
sudo apache2ctl configtest

# Restart Apache
sudo systemctl restart apache2
```

### Multiple Websites Configuration
For hosting multiple websites:

```apache
# Site 1
<VirtualHost *:80>
    ServerName site1.com
    DocumentRoot /var/www/site1.com
    # ... other configurations
</VirtualHost>

# Site 2
<VirtualHost *:80>
    ServerName site2.com
    DocumentRoot /var/www/site2.com
    # ... other configurations
</VirtualHost>
```

---

## SSL Certificate Installation

### Install Certbot
```bash
# Install Certbot for Apache
sudo apt install certbot python3-certbot-apache -y
```

### Obtain SSL Certificate
```bash
# Get SSL certificate for your domain
sudo certbot --apache -d your-website.com -d www.your-website.com
```

### Auto-renewal Setup
```bash
# Test renewal
sudo certbot renew --dry-run

# Check renewal service
sudo systemctl status certbot.timer

# Enable auto-renewal (if not already enabled)
sudo systemctl enable certbot.timer
```

### Manual SSL Configuration (if needed)
```apache
<VirtualHost *:443>
    ServerName your-website.com
    DocumentRoot /var/www/your-website.com

    # SSL Configuration
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/your-website.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/your-website.com/privkey.pem

    # Security headers
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
    Header always set X-Frame-Options DENY
    Header always set X-Content-Type-Options nosniff
</VirtualHost>
```

---

## Security Hardening

### Apache Security
```bash
# Hide Apache version
sudo nano /etc/apache2/conf-available/security.conf
```

```apache
# Hide server information
ServerTokens Prod
ServerSignature Off

# Security headers
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
Header always set X-XSS-Protection "1; mode=block"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
```

### Enable Security Module
```bash
# Enable headers module
sudo a2enmod headers

# Enable security configuration
sudo a2enconf security

# Restart Apache
sudo systemctl restart apache2
```

### MySQL Security
```bash
# Additional MySQL security
sudo mysql -u root -p
```

```sql
-- Remove test database
DROP DATABASE IF EXISTS test;

-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Remove remote root access
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');

-- Flush privileges
FLUSH PRIVILEGES;
```

### File Permissions
```bash
# Set proper ownership
sudo chown -R www-data:www-data /var/www/your-website.com

# Set directory permissions
find /var/www/your-website.com -type d -exec chmod 755 {} \;

# Set file permissions
find /var/www/your-website.com -type f -exec chmod 644 {} \;

# Protect sensitive files
chmod 600 /var/www/your-website.com/config.php
```

### Fail2Ban Installation
```bash
# Install Fail2Ban
sudo apt install fail2ban -y

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local
```

**Basic Fail2Ban configuration:**
```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[apache-auth]
enabled = true

[apache-badbots]
enabled = true

[apache-noscript]
enabled = true

[apache-overflows]
enabled = true
```

---

## Performance Optimization

### Apache Optimization
```bash
# Edit Apache configuration
sudo nano /etc/apache2/apache2.conf
```

**Performance settings:**
```apache
# Worker configuration
<IfModule mpm_prefork_module>
    StartServers          4
    MinSpareServers       20
    MaxSpareServers       40
    MaxRequestWorkers     200
    MaxConnectionsPerChild 4500
</IfModule>

# Keep alive settings
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

### Enable Compression
```bash
# Enable compression modules
sudo a2enmod deflate
sudo a2enmod expires
sudo a2enmod headers

# Create compression configuration
sudo nano /etc/apache2/conf-available/compression.conf
```

```apache
<IfModule mod_deflate.c>
    # Compress HTML, CSS, JavaScript, Text, XML and fonts
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/rss+xml
    AddOutputFilterByType DEFLATE application/vnd.ms-fontobject
    AddOutputFilterByType DEFLATE application/x-font
    AddOutputFilterByType DEFLATE application/x-font-opentype
    AddOutputFilterByType DEFLATE application/x-font-otf
    AddOutputFilterByType DEFLATE application/x-font-truetype
    AddOutputFilterByType DEFLATE application/x-font-ttf
    AddOutputFilterByType DEFLATE application/x-javascript
    AddOutputFilterByType DEFLATE application/xhtml+xml
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE font/opentype
    AddOutputFilterByType DEFLATE font/otf
    AddOutputFilterByType DEFLATE font/ttf
    AddOutputFilterByType DEFLATE image/svg+xml
    AddOutputFilterByType DEFLATE image/x-icon
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/javascript
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE text/xml
</IfModule>
```

### Enable Caching
```apache
<IfModule mod_expires.c>
    ExpiresActive On

    # Images
    ExpiresByType image/jpg "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/gif "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"

    # CSS and JavaScript
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/pdf "access plus 1 month"
    ExpiresByType text/javascript "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"

    # Icon
    ExpiresByType image/x-icon "access plus 1 year"
</IfModule>
```

Enable configurations:
```bash
sudo a2enconf compression
sudo systemctl restart apache2
```

### PHP Optimization
Enable OPcache:
```bash
sudo nano /etc/php/8.1/apache2/php.ini
```

```ini
# OPcache settings
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
```

### MySQL Optimization
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

```ini
[mysqld]
# InnoDB settings
innodb_buffer_pool_size = 256M
innodb_log_file_size = 64M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT

# Query cache
query_cache_type = 1
query_cache_size = 64M
query_cache_limit = 2M

# Connection settings
max_connections = 100
wait_timeout = 600
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Apache Won't Start
**Check error logs:**
```bash
sudo journalctl -u apache2.service
sudo tail -f /var/log/apache2/error.log
```

**Common fixes:**
```bash
# Check configuration syntax
sudo apache2ctl configtest

# Check port conflicts
sudo netstat -tulpn | grep :80

# Fix permissions
sudo chown -R www-data:www-data /var/www/
```

#### 2. PHP Not Working
**Verify PHP module:**
```bash
# Check if PHP module is loaded
apache2ctl -M | grep php

# Enable PHP module
sudo a2enmod php8.1

# Restart Apache
sudo systemctl restart apache2
```

#### 3. Database Connection Errors
**Check MySQL status:**
```bash
sudo systemctl status mysql

# Check MySQL logs
sudo tail -f /var/log/mysql/error.log

# Test connection
mysql -u username -p -h localhost database_name
```

#### 4. phpMyAdmin Access Issues
**Check configuration:**
```bash
# Verify phpMyAdmin configuration
sudo apache2ctl configtest

# Check if configuration is enabled
ls -la /etc/apache2/conf-enabled/ | grep phpmyadmin

# Enable phpMyAdmin
sudo a2enconf phpmyadmin
```

#### 5. SSL Certificate Problems
```bash
# Check certificate status
sudo certbot certificates

# Renew certificate manually
sudo certbot renew

# Check SSL configuration
openssl s_client -connect your-domain.com:443
```

### Useful Commands

#### System Monitoring
```bash
# Check system resources
htop
free -m
df -h

# Check active connections
ss -tuln

# Monitor Apache processes
ps aux | grep apache

# Check MySQL processes
ps aux | grep mysql
```

#### Log Monitoring
```bash
# Apache access log
sudo tail -f /var/log/apache2/access.log

# Apache error log
sudo tail -f /var/log/apache2/error.log

# MySQL error log
sudo tail -f /var/log/mysql/error.log

# PHP error log
sudo tail -f /var/log/php_errors.log
```

#### Service Management
```bash
# Apache
sudo systemctl {start|stop|restart|reload|status} apache2

# MySQL
sudo systemctl {start|stop|restart|status} mysql

# Check all services
sudo systemctl list-units --type=service --state=active
```

### Performance Monitoring
```bash
# Install monitoring tools
sudo apt install htop iotop nethogs

# Monitor Apache performance
sudo apachectl status

# Check MySQL performance
sudo mysql -u root -p -e "SHOW PROCESSLIST;"
sudo mysql -u root -p -e "SHOW STATUS LIKE 'Slow_queries';"
```

---

## Maintenance & Backup

### Regular Maintenance Tasks

#### System Updates
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Update PHP packages
sudo apt update && sudo apt install php

# Clean package cache
sudo apt autoremove && sudo apt autoclean
```

#### Database Backup
```bash
# Create backup script
nano ~/backup_database.sh
```

```bash
#!/bin/bash
# Database backup script

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/mysql"
DB_USER="backup_user"
DB_PASS="backup_password"
DB_NAME="your_website_db"

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
mysqldump -u$DB_USER -p$DB_PASS $DB_NAME > $BACKUP_DIR/db_backup_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/db_backup_$DATE.sql

# Remove backups older than 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
```

```bash
# Make script executable
chmod +x ~/backup_database.sh

# Add to crontab for daily backup
crontab -e

# Add this line for daily backup at 2 AM
0 2 * * * /home/username/backup_database.sh
```

#### Website Files Backup
```bash
# Backup website files
tar -czf /var/backups/website_$(date +%Y%m%d).tar.gz /var/www/your-website.com

# Backup with rsync
rsync -av /var/www/your-website.com/ /backup/location/
```

### Security Monitoring
```bash
# Check failed login attempts
sudo grep "Failed" /var/log/auth.log

# Monitor Apache access for suspicious activity
sudo grep "404" /var/log/apache2/access.log | tail -20

# Check for malware (install ClamAV)
sudo apt install clamav clamav-daemon
sudo freshclam
sudo clamscan -r /var/www/your-website.com
```

---

## Best Practices

### Security Best Practices
1. **Keep everything updated** regularly
2. **Use strong passwords** for all accounts
3. **Implement proper file permissions**
4. **Enable firewall** with minimal required ports
5. **Use SSL certificates** for all websites
6. **Regular security audits** and monitoring
7. **Backup data** regularly and test restores
8. **Monitor logs** for suspicious activities

### Performance Best Practices
1. **Enable compression** for text-based files
2. **Use caching** mechanisms (browser, server-side)
3. **Optimize images** and media files
4. **Monitor resource usage** regularly
5. **Database optimization** with proper indexing
6. **Use CDN** for static content delivery
7. **Regular performance audits**

### Development Best Practices
1. **Use version control** (Git) for code management
2. **Separate environments** (development, staging, production)
3. **Error logging** and monitoring
4. **Code documentation** and comments
5. **Regular testing** and quality assurance
6. **Database migrations** and schema management

---

## Conclusion

This comprehensive guide provides everything needed to successfully deploy PHP websites on a VPS using the LAMP stack. The configuration includes security hardening, performance optimization, and maintenance procedures to ensure a reliable and secure hosting environment.

### Key Takeaways
- **LAMP stack** provides a robust foundation for PHP applications
- **Security hardening** is crucial for production environments
- **Regular maintenance** ensures optimal performance and security
- **Monitoring and logging** help identify and resolve issues quickly
- **Backup strategies** protect against data loss

### Next Steps
- Set up automated backups
- Implement monitoring and alerting
- Configure additional security measures
- Optimize for your specific application needs
- Plan for scaling as your website grows

For additional support, refer to the official documentation of each component and consider professional security audits for production environments.

---

**Remember**: Always test configurations in a staging environment before applying to production servers.
