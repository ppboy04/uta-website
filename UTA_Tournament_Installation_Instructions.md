# UTA Tournament Website - Installation Instructions

**Quick Setup Guide for AWS Deployment**

---

## 📋 Overview

This guide will help you install the UTA Tournament Management System on your existing AWS account. The installation includes:
- ✅ EC2 server setup with the website
- ✅ RDS MySQL database configuration
- ✅ SSL certificate setup
- ✅ Complete working tournament system

**Total Installation Time: 30-45 minutes**

---

## 📦 Prerequisites

Before starting, ensure you have:
- ✅ AWS account with EC2 and RDS access
- ✅ **UTA Tournament project zip file** (provided by developer)
- ✅ SSH key pair for EC2 access
- ✅ Basic command line knowledge

---

## 🚀 Step 1: Launch EC2 Instance

### 1.1 Create EC2 Instance
1. **Login to AWS Console** → Go to EC2 Dashboard
2. **Click "Launch Instance"**
3. **Configure as follows:**

\`\`\`
Name: UTA-Tournament-Server
AMI: Ubuntu Server 22.04 LTS (Free tier eligible)
Instance Type: t3.medium (2 vCPU, 4 GB RAM)
Key Pair: Create new or select existing
Storage: 20 GB gp3 SSD
\`\`\`

### 1.2 Security Group Settings
**Create new security group: "UTA-Web-Security"**

\`\`\`
Inbound Rules:
- SSH (22) - Your IP only
- HTTP (80) - Anywhere (0.0.0.0/0)
- HTTPS (443) - Anywhere (0.0.0.0/0)
- Custom TCP (3000) - Anywhere (0.0.0.0/0)

Outbound Rules:
- All traffic - Anywhere (0.0.0.0/0)
\`\`\`

### 1.3 Launch Instance
- Click "Launch Instance"
- Wait for instance to be "Running"
- Note down the **Public IP address**

---

## 🗄️ Step 2: Create RDS Database

### 2.1 Launch RDS Instance
1. **Go to RDS Dashboard** → Click "Create database"
2. **Configure as follows:**

\`\`\`
Engine: MySQL
Version: MySQL 8.0
Template: Free tier (if eligible) or Production
DB Instance Identifier: uta-tournament-db
Master Username: admin
Master Password: [Create strong password - save this!]
DB Instance Class: db.t3.micro
Storage: 20 GB gp2
\`\`\`

### 2.2 Database Security Settings
\`\`\`
VPC: Default VPC
Public Access: No
VPC Security Group: Create new "UTA-DB-Security"
Database Name: uta_tournament
\`\`\`

### 2.3 Security Group for Database
**Create "UTA-DB-Security" group:**

\`\`\`
Inbound Rules:
- MySQL/Aurora (3306) - Source: UTA-Web-Security group
- MySQL/Aurora (3306) - Source: Your IP (for management)

Outbound Rules:
- All traffic - Anywhere
\`\`\`

### 2.4 Launch Database
- Click "Create database"
- Wait 5-10 minutes for database to be "Available"
- Note down the **Endpoint URL**

---

## 💻 Step 3: Connect to Server and Install Software

### 3.1 Connect to EC2 Instance
\`\`\`bash
# Replace with your key file and public IP
ssh -i your-key.pem ubuntu@YOUR-EC2-PUBLIC-IP
\`\`\`

### 3.2 Update System and Install Node.js
\`\`\`bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install additional packages
sudo apt install -y git nginx mysql-client-core-8.0 unzip

# Install PM2 globally
sudo npm install -g pm2

# Verify installations
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x
\`\`\`

---

## 📁 Step 4: Upload and Setup Website Files

### 4.1 Create Application Directory
\`\`\`bash
# Create directory
sudo mkdir -p /var/www/uta-tournament
sudo chown ubuntu:ubuntu /var/www/uta-tournament

# Navigate to directory
cd /var/www/uta-tournament
\`\`\`

### 4.2 Upload Project Files

**Option A: Using SCP (from your local machine)**
\`\`\`bash
# From your local machine (where you have the zip file)
scp -i your-key.pem uta-tournament.zip ubuntu@YOUR-EC2-PUBLIC-IP:/home/ubuntu/
\`\`\`

**Option B: Using SFTP**
1. Use an SFTP client like FileZilla or WinSCP
2. Connect to your EC2 instance using your key pair
3. Upload `uta-tournament.zip` to `/home/ubuntu/` directory

**Option C: Using wget (if zip is hosted online)**
\`\`\`bash
# If the zip file is available via URL
wget https://your-download-link/uta-tournament.zip
\`\`\`

### 4.3 Extract and Setup Files
\`\`\`bash
# Navigate to home directory
cd /home/ubuntu

# Extract the zip file
unzip uta-tournament.zip

# Move files to web directory
sudo mv uta-tournament/* /var/www/uta-tournament/
# OR if files are in a subfolder:
# sudo mv uta-tournament-main/* /var/www/uta-tournament/

# Change ownership
sudo chown -R ubuntu:ubuntu /var/www/uta-tournament

# Navigate to application directory
cd /var/www/uta-tournament

# Verify files are present
ls -la
# You should see: package.json, app/, components/, etc.
\`\`\`

### 4.4 Install Dependencies
\`\`\`bash
# Install Node.js dependencies
npm install

# Create logs directory
sudo mkdir -p /var/log/uta-tournament
sudo chown ubuntu:ubuntu /var/log/uta-tournament
\`\`\`

---

## ⚙️ Step 5: Configure Environment Variables

### 5.1 Create Environment File
\`\`\`bash
# Navigate to application directory
cd /var/www/uta-tournament

# Create environment configuration
nano .env.local
\`\`\`

### 5.2 Add Configuration (Replace with your actual values)
\`\`\`env
# Database Configuration
DATABASE_URL="mysql://admin:YOUR_DB_PASSWORD@YOUR_RDS_ENDPOINT:3306/uta_tournament"
DB_HOST="YOUR_RDS_ENDPOINT"
DB_USER="admin"
DB_PASSWORD="YOUR_DB_PASSWORD"
DB_NAME="uta_tournament"
DB_PORT="3306"

# Application Configuration
NEXTAUTH_URL="http://YOUR_EC2_PUBLIC_IP:3000"
NEXTAUTH_SECRET="your-random-secret-key-here-make-it-long-and-random"
NODE_ENV="production"
PORT="3000"

# Security
JWT_SECRET="another-random-secret-key-for-jwt-tokens"
BCRYPT_ROUNDS="12"
\`\`\`

**Important:** Replace these values:
- `YOUR_DB_PASSWORD` - The password you set for RDS
- `YOUR_RDS_ENDPOINT` - The RDS endpoint URL (e.g., uta-tournament-db.xxxxx.us-east-1.rds.amazonaws.com)
- `YOUR_EC2_PUBLIC_IP` - Your EC2 instance public IP

**Example:**
\`\`\`env
DATABASE_URL="mysql://admin:MySecurePass123@uta-tournament-db.c9akl5xxxxxx.us-east-1.rds.amazonaws.com:3306/uta_tournament"
DB_HOST="uta-tournament-db.c9akl5xxxxxx.us-east-1.rds.amazonaws.com"
DB_USER="admin"
DB_PASSWORD="MySecurePass123"
DB_NAME="uta_tournament"
DB_PORT="3306"
NEXTAUTH_URL="http://54.123.45.67:3000"
NEXTAUTH_SECRET="super-long-random-secret-key-for-nextauth-security"
NODE_ENV="production"
PORT="3000"
JWT_SECRET="another-super-long-random-secret-for-jwt-tokens"
BCRYPT_ROUNDS="12"
\`\`\`

---

## 🗃️ Step 6: Setup Database Tables

### 6.1 Connect to Database
\`\`\`bash
# Test database connection (replace with your RDS endpoint and password)
mysql -h YOUR_RDS_ENDPOINT -u admin -p
# Enter your database password when prompted
\`\`\`

### 6.2 Create Database Schema
\`\`\`sql
-- Connect to the database
USE uta_tournament;

-- Create events table
CREATE TABLE IF NOT EXISTS tbl_eventName (
    id VARCHAR(10) PRIMARY KEY,
    event_name VARCHAR(100) NOT NULL,
    description TEXT,
    max_teams INT DEFAULT 32,
    entry_fee DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Create players table
CREATE TABLE IF NOT EXISTS tbl_players (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    whatsapp_number VARCHAR(15) NOT NULL UNIQUE,
    dob DATE NOT NULL,
    city VARCHAR(100) NOT NULL,
    shirt_size ENUM('S', 'M', 'L', 'XL', 'XXL') NOT NULL,
    short_size ENUM('S', 'M', 'L', 'XL', 'XXL') NOT NULL,
    food_pref ENUM('veg', 'nonveg') NOT NULL DEFAULT 'veg',
    stay BOOLEAN NOT NULL DEFAULT FALSE,
    fee_paid BOOLEAN NOT NULL DEFAULT FALSE,
    registration_id VARCHAR(20) UNIQUE,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_whatsapp (whatsapp_number),
    INDEX idx_registration (registration_id),
    INDEX idx_status (status)
);

-- Create partners table
CREATE TABLE IF NOT EXISTS tbl_partners (
    id INT AUTO_INCREMENT PRIMARY KEY,
    event_name VARCHAR(10) NOT NULL,
    user_id INT NOT NULL,
    partner_id INT,
    ranking INT DEFAULT 0,
    status ENUM('registered', 'confirmed', 'withdrawn') DEFAULT 'registered',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (event_name) REFERENCES tbl_eventName(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES tbl_players(id) ON DELETE CASCADE,
    FOREIGN KEY (partner_id) REFERENCES tbl_players(id) ON DELETE SET NULL,
    
    UNIQUE KEY unique_user_event (user_id, event_name),
    INDEX idx_event (event_name),
    INDEX idx_ranking (ranking)
);

-- Create admin users table
CREATE TABLE IF NOT EXISTS tbl_admin_users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    role ENUM('admin', 'super_admin') DEFAULT 'admin',
    last_login TIMESTAMP NULL,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_username (username),
    INDEX idx_status (status)
);
\`\`\`

### 6.3 Insert Initial Data
\`\`\`sql
-- Insert event categories
INSERT INTO tbl_eventName (id, event_name, description, entry_fee) VALUES
('a', 'Category A (Open)', 'Open category for all eligible players. Coaches allowed.', 3000.00),
('b', 'Category B (90+ combined)', 'For pairs with combined age of 90+ years', 3000.00),
('c', 'Category C (105+ combined)', 'For pairs with combined age of 105+ years', 3000.00),
('d', 'Category D (120+ combined)', 'For pairs with combined age of 120+ years', 3000.00),
('lucky', 'Lucky Doubles', 'For players who lose both matches in round 1', 0.00);

-- Insert default admin user (username: admin, password: admin123)
INSERT INTO tbl_admin_users (username, password_hash, email, role) VALUES
('admin', '$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewdBPj3L3jzjvQSG', 'admin@utatournament.com', 'super_admin');

-- Insert sample players for testing
INSERT INTO tbl_players (name, whatsapp_number, dob, city, shirt_size, short_size, food_pref, stay, fee_paid, registration_id) VALUES
('Rahul Sharma', '9876543210', '1988-06-15', 'Dehradun', 'L', 'M', 'veg', TRUE, TRUE, 'UTA-2024-001'),
('Vikram Singh', '9876543211', '1985-08-22', 'Delhi', 'XL', 'L', 'nonveg', FALSE, TRUE, 'UTA-2024-002'),
('Amit Patel', '9876543212', '1991-03-10', 'Mumbai', 'M', 'M', 'veg', TRUE, TRUE, 'UTA-2024-003');

-- Verify data insertion
SELECT 'Events' as table_name, COUNT(*) as count FROM tbl_eventName
UNION ALL
SELECT 'Admin Users', COUNT(*) FROM tbl_admin_users
UNION ALL
SELECT 'Sample Players', COUNT(*) FROM tbl_players;

-- Exit MySQL
EXIT;
\`\`\`

---

## 🚀 Step 7: Build and Start the Application

### 7.1 Build the Application
\`\`\`bash
# Navigate to application directory
cd /var/www/uta-tournament

# Install production dependencies
npm install --production

# Build the Next.js application
npm run build

# Verify build completed successfully
ls -la .next/
\`\`\`

### 7.2 Create PM2 Configuration
\`\`\`bash
# Create PM2 ecosystem file
nano ecosystem.config.js
\`\`\`

\`\`\`javascript
module.exports = {
  apps: [{
    name: 'uta-tournament',
    script: 'npm',
    args: 'start',
    cwd: '/var/www/uta-tournament',
    instances: 1,
    exec_mode: 'fork',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: '/var/log/uta-tournament/err.log',
    out_file: '/var/log/uta-tournament/out.log',
    log_file: '/var/log/uta-tournament/combined.log',
    time: true,
    max_memory_restart: '1G',
    restart_delay: 4000,
    max_restarts: 10,
    min_uptime: '10s'
  }]
};
\`\`\`

### 7.3 Start the Application
\`\`\`bash
# Start application with PM2
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 to start on boot
pm2 startup
# Follow the command it shows you (copy and paste the sudo command)

# Check application status
pm2 status
pm2 logs uta-tournament --lines 50
\`\`\`

---

## 🌐 Step 8: Configure Nginx (Optional but Recommended)

### 8.1 Configure Nginx
\`\`\`bash
# Create Nginx configuration
sudo nano /etc/nginx/sites-available/uta-tournament
\`\`\`

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_IP;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/json;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;
    }

    # Static files caching
    location /_next/static {
        proxy_pass http://localhost:3000;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }
}
