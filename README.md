# MERN Stack VPS Deployment Guide

A complete guide for deploying MERN stack applications on a Virtual Private Server (VPS) with Nginx, Node.js, React, and MongoDB.

## Table of Contents
- [Prerequisites](#prerequisites)
- [1. Nginx Installation & Configuration](#1-nginx-installation--configuration)
- [2. Node.js, React & Next.js Setup](#2-nodejs-react--nextjs-setup)
- [3. MongoDB Installation & Configuration](#3-mongodb-installation--configuration)
- [4. SSH Key Setup for Git](#4-ssh-key-setup-for-git)
- [5. SSL Certificate Installation](#5-ssl-certificate-installation)

## Prerequisites
- VPS server with Ubuntu/Debian
- Root access or sudo privileges
- Domain name (optional)
- Basic terminal/SSH knowledge

---

## 1. Nginx Installation & Configuration

### Connect to Server
Login to your VPS using SSH:
```bash
ssh root@<server_ip_address>
```

### Check Existing Web Servers
```bash
nginx -v
apache -v
```

### Install Nginx
```bash
sudo apt update
apt install nginx
```

### Start and Configure Nginx
```bash
# Start nginx
systemctl start nginx

# Check status
systemctl status nginx

# Restart nginx
systemctl restart nginx
```

### Configure Nginx Server Blocks

1. **Delete default server files:**
```bash
rm /etc/nginx/sites-available/default
rm /etc/nginx/sites-enabled/default
```

2. **Create new server configuration:**
```bash
nano /etc/nginx/sites-available/<file_name>
```

### Nginx Configuration Examples

#### For ReactJS Applications:
```nginx
server {
    listen 80;

    location / {
        root /var/www/<project_folder_name>;
        index index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
    }
}
```

#### For NodeJS and NextJS Applications:
```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://YOUR_SERVER_IP:8800;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### Multiple Services on Single Domain:
```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        root /var/www/netflix/client;
        index index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
    }

    location /api/v1 {
        proxy_pass http://YOUR_SERVER_IP:8800;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

3. **Enable the site:**
```bash
ln -s /etc/nginx/sites-available/<file_name> /etc/nginx/sites-enabled/<file_name>
```

4. **Restart Nginx:**
```bash
systemctl restart nginx
```

### Project Setup
```bash
# Navigate to web directory
cd /var/www/html

# Create project folder
mkdir <folder_name>

# Enter project directory
cd <folder_name>
```

---

## 2. Node.js, React & Next.js Setup

### System Update
```bash
sudo apt update
```

### Install NVM Dependencies
```bash
sudo apt install curl ca-certificates bash
```

### Install Node Version Manager (NVM)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

**Note:** Close and reopen terminal to reload shell configuration.

### Verify NVM Installation
```bash
nvm --version
```

### Install Node.js
```bash
# Install Node.js version 20
nvm install 20

# Verify installation
node -v
npm -v
```

### Git Setup and Project Cloning
```bash
# Install git
apt install git

# Clone specific branch
git clone --branch <Branch_Name> https://github.com/user/repository.git .

# Pull latest code
git pull
```

### Install PM2 Process Manager
```bash
npm i -g pm2
```

### Install Project Dependencies
```bash
# Install packages
npm install

# Update packages
npm update
```

### Install Nodemon (Optional)
```bash
npm install nodemon --global
```

### Start Applications with PM2

#### For NodeJS Applications:
```bash
pm2 start --name <name> nodemon

# Restart
pm2 restart <name>
```

#### For NextJS Applications:
```bash
# Build the application
npm run build

# Start with PM2
pm2 start npm --name "<name>" -- run start

# Restart
npm run build
pm2 restart <name>
```

### PM2 Management Commands
```bash
# List all running processes
pm2 list

# View logs
pm2 log <name>

# Save PM2 configuration
pm2 save

# Setup startup script
pm2 startup
```

**Important:** After setting up startup script, save PM2 again using `pm2 save`.

---

## 3. MongoDB Installation & Configuration

### Official Installation Guide
Follow MongoDB's official documentation: https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/

### Check Existing MongoDB Installation
```bash
mongod --version
```

### Update Package List
```bash
apt update
```

### Import MongoDB GPG Key
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
```

### Create MongoDB Source List
```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

### Install MongoDB
```bash
apt update
apt install -y mongodb-org
```

### Start MongoDB Service
```bash
# Check status
systemctl status mongod

# Start if not running
systemctl start mongod
```

### Alternative Installation (if errors occur):
```bash
# For MongoDB 6.0
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/mongodb-org-6.0.gpg > /dev/null
sudo apt update
sudo apt install -y mongodb-org
```

### Database and User Setup

#### Connect to MongoDB
```bash
mongosh
```

#### Create Administrative User
```bash
# Switch to admin database
use admin

# Create super admin user
db.createUser({
    user: "superAdmin",
    pwd: "PRO4z&9Yp*B#2qV@Xm!GST",
    roles: [
        { role: "userAdminAnyDatabase", db: "admin" },
        { role: "dbAdminAnyDatabase", db: "admin" },
        { role: "readWriteAnyDatabase", db: "admin" }
    ]
});

# Exit
exit
```

#### Create Database-Specific Users
```bash
# Connect with authentication
mongosh --username superAdmin --authenticationDatabase admin

# Create new database
use database-name

# Create database user
db.createUser({
    user: "user1",
    pwd: "password1",
    roles: [
        { role: "readWrite", db: "database-name" },
        { role: "dbAdmin", db: "database-name" }
    ]
});

# Create test collection for Compass visibility
use <database_name>
show collections
db.test_collection.insertOne({ createdAt: new Date() })

exit
```

#### Test Connection
```bash
mongosh -u Your-User-Name -p 'Your-Password' --authenticationDatabase 'Your-DB-Name'
```

### Configure MongoDB for Remote Access

#### Edit MongoDB Configuration
```bash
nano /etc/mongod.conf
```

**Important:** This file is space-sensitive. Edit carefully.

#### Update Network Interfaces
```yaml
net:
    port: 27017
    bindIp: 0.0.0.0
```

#### Enable Authentication
```yaml
security:
    authorization: enabled
```

#### Restart MongoDB
```bash
sudo systemctl restart mongod
```

### MongoDB Compass Connection Strings

#### Database-specific connection:
```
mongodb://Your-User-Name:Your-Password@Your-IP:27017/Your-DB-Name
```

#### Super admin connection (access all databases):
```
mongodb://Your-SuperUser-Name:superUserPassword@Your-IP:27017/admin?authSource=admin
```

---

## 4. SSH Key Setup for Git

### Generate SSH Key on VPS
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Press Enter to save in default location (`~/.ssh/id_rsa`).

### Copy SSH Public Key
```bash
cat ~/.ssh/id_rsa.pub
```

Copy the entire output starting with `ssh-rsa`.

### Add SSH Key to Git Provider
1. Go to your Git provider (GitHub, GitLab, etc.)
2. Navigate to Settings → SSH and GPG Keys
3. Click "Add SSH Key"
4. Paste your public key

### Configure Repository for SSH
```bash
# Change remote URL to SSH
git remote set-url origin git@github.com:username/repo.git
```

### Test SSH Connection
```bash
ssh -T git@github.com
```

Expected response:
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### Adding SSH Key for Additional Repositories

Use the same SSH key for multiple repositories on the same Git provider. For different providers, add the same public key to each provider's SSH settings.

---

## 5. SSL Certificate Installation

### Update System and Install Certbot
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

### Verify Installation
```bash
certbot --version
```

### Test Nginx Configuration
```bash
sudo nginx -t
```

### Restart Nginx if Needed
```bash
sudo systemctl restart nginx
```

### Install SSL Certificate
```bash
# Automatic installation for all domains
sudo certbot --nginx
```

Certbot will automatically detect domains from your Nginx configuration.

### Setup Auto-renewal
```bash
# Test renewal process
sudo certbot renew --dry-run
```

### Add SSL for New Domains
If you add new domains after SSL installation:
```bash
sudo certbot --nginx
```

---

## Additional Notes

### Important Commands Summary
```bash
# Nginx management
systemctl restart nginx
nginx -t

# PM2 management
pm2 list
pm2 restart <app_name>
pm2 save
pm2 startup

# MongoDB management
systemctl restart mongod
systemctl status mongod

# Git operations
git pull
git clone --branch <branch> <repo_url> .
```

### Security Best Practices
- Use strong passwords for database users
- Keep your system updated: `sudo apt update && sudo apt upgrade`
- Configure firewall rules
- Regularly backup your databases
- Monitor server logs

### Troubleshooting Tips
- Check service status: `systemctl status <service_name>`
- View logs: `journalctl -u <service_name>`
- Test configurations before restarting services
- Ensure proper file permissions for web directories

This guide provides a comprehensive approach to deploying MERN stack applications on VPS servers. Follow each section carefully and test configurations before proceeding to the next step.
