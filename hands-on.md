## **COMPREHENSIVE HANDS-ON PRACTICE GUIDE FOR TURING DEBIAN ENGINEER TEST**

### **PHASE 1: INITIAL SETUP & SYSTEM PREPARATION**

#### **Step 1.1: Debian Installation**

```bash
# Installation Steps:
1. Boot from debian-13.1.0-amd64-netinst.iso
2. Choose "Graphical install" or "Install" based on preference
3. Select language, location, keyboard layout
4. Configure network (DHCP or static IP)
5. Set hostname: "debian-practice"
6. Set root password and create user account
7. Partition disk: Use "Guided - use entire disk" for simplicity
8. Choose software: Select "SSH server" and "Standard system utilities"
9. Install GRUB bootloader
10. Complete installation and reboot
```

#### **Step 1.2: Post-Installation Setup**

```bash
# Login as root first, then switch to your user

# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install essential tools
sudo apt install -y \
    vim nano \
    curl wget \
    git \
    htop \
    net-tools \
    dnsutils \
    tcpdump \
    strace \
    lsof \
    iotop \
    dstat \
    tree \
    less \
    man-db \
    build-essential \
    unattended-upgrades \
    fail2ban \
    ufw \
    openssh-server \
    rsync \
    tar gzip \
    locate \
    cron \
    logrotate

# 3. Install development tools
sudo apt install -y \
    python3 python3-pip \
    bash-completion \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg2 \
    lsb-release

# 4. Enable bash completion
echo 'source /etc/bash_completion' >> ~/.bashrc
source ~/.bashrc

# 5. Create practice directories
mkdir -p ~/practice/{scripts,logs,backups,configs,tests}
cd ~/practice
```

---

### **PHASE 2: HIGH PRIORITY TOPICS (Core Debian Skills)**

#### **MODULE 2.1: APT Package Management**

**Exercise 2.1.1: Basic Package Operations**
```bash
# 1. Update package lists
sudo apt update

# 2. Search for packages
apt search nginx
apt-cache search "web server"

# 3. Show package information
apt show nginx
apt-cache show nginx

# 4. Install packages
sudo apt install -y nginx vim

# 5. List installed packages
dpkg -l | grep nginx
dpkg -l | less

# 6. Show package files
dpkg -L nginx

# 7. Check package status
dpkg -s nginx

# 8. Remove packages
sudo apt remove nginx
sudo apt purge nginx  # Removes config files too
sudo apt autoremove   # Removes unused dependencies
```

**Exercise 2.1.2: Dependency Management**
```bash
# 1. Install a package intentionally with dependencies
sudo apt install -y apache2

# 2. Check dependencies
apt-cache depends apache2
apt-cache rdepends apache2

# 3. Simulate broken dependencies
sudo apt remove libapache2-mod-php8.1
sudo apt install -f  # Fix broken dependencies

# 4. Check for broken packages
dpkg --audit
```

**Exercise 2.1.3: Repository Management**
```bash
# 1. List current repositories
cat /etc/apt/sources.list
ls -la /etc/apt/sources.list.d/

# 2. Add a repository (example: Docker)
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Update package lists
sudo apt update

# 4. Verify GPG keys
apt-key list
gpg --list-keys --keyring /usr/share/keyrings/docker-archive-keyring.gpg

# 5. Add repository manually
sudo nano /etc/apt/sources.list.d/custom.list
# Add: deb http://example.com/debian stable main
sudo apt update
```

**Exercise 2.1.4: Package Version Management**
```bash
# 1. List available versions
apt-cache policy nginx
apt-cache madison nginx

# 2. Install specific version
sudo apt install nginx=1.18.0-6

# 3. Pin package to version
sudo apt-mark hold nginx

# 4. List held packages
apt-mark showhold

# 5. Unhold package
sudo apt-mark unhold nginx

# 6. Configure package pinning
sudo nano /etc/apt/preferences.d/nginx-pin
# Add:
# Package: nginx
# Pin: version 1.18.0*
# Pin-Priority: 1001
```

**Exercise 2.1.5: Troubleshooting Broken Packages**
```bash
# SCENARIO: Package installation fails
# 1. Check for broken packages
sudo dpkg --configure -a

# 2. Fix broken dependencies
sudo apt --fix-broken install

# 3. Clean package cache
sudo apt clean
sudo apt autoclean

# 4. Remove partial installations
sudo rm -rf /var/lib/dpkg/info/package-name.*
sudo apt remove --purge package-name

# 5. Reconfigure packages
sudo dpkg-reconfigure -a
```

**Practice Scenario from Q&A:**
```bash
# Scenario: User installed custom packages using dpkg, broken dependencies
# Solution:
sudo apt-get update
sudo apt-get install -f
sudo dpkg --configure -a
dpkg -l | grep -i broken
sudo apt-get purge problematic-package
sudo apt-get install package-name
```

---

#### **MODULE 2.2: Systemd Service Management**

**Exercise 2.2.1: Basic Service Operations**
```bash
# 1. List all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running

# 2. Check service status
systemctl status ssh
systemctl status nginx

# 3. Start/Stop/Restart services
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx  # Without downtime

# 4. Enable/Disable services
sudo systemctl enable nginx
sudo systemctl disable nginx
sudo systemctl is-enabled nginx

# 5. Mask/Unmask services (prevent starting)
sudo systemctl mask nginx
sudo systemctl unmask nginx
```

**Exercise 2.2.2: View Service Logs**
```bash
# 1. View journal logs
journalctl -u nginx
journalctl -u nginx -f  # Follow logs
journalctl -u nginx --since today
journalctl -u nginx --since "2024-01-01 00:00:00" --until "2024-01-02 00:00:00"

# 2. View boot logs
journalctl -b
journalctl -b -1  # Previous boot

# 3. View system logs
journalctl -p err  # Errors only
journalctl -p warning -p err

# 4. Analyze boot time
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
```

**Exercise 2.2.3: Create Custom Systemd Service**
```bash
# Create a simple service
sudo nano /etc/systemd/system/my-custom-service.service

# Add:
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
User=your-username
ExecStart=/usr/bin/python3 /home/your-username/practice/scripts/simple_service.py
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target

# Create the script
mkdir -p ~/practice/scripts
cat > ~/practice/scripts/simple_service.py << 'EOF'
#!/usr/bin/env python3
import time
import sys

while True:
    with open('/tmp/service-log.txt', 'a') as f:
        f.write(f"Service running at {time.ctime()}\n")
    time.sleep(60)
EOF

chmod +x ~/practice/scripts/simple_service.py

# Reload systemd and start service
sudo systemctl daemon-reload
sudo systemctl enable my-custom-service
sudo systemctl start my-custom-service
sudo systemctl status my-custom-service
```

**Exercise 2.2.4: Service Dependencies**
```bash
# 1. View service dependencies
systemctl list-dependencies nginx
systemctl list-dependencies nginx --reverse

# 2. Create service with dependencies
sudo nano /etc/systemd/system/dependent-service.service

# Add:
[Unit]
Description=Dependent Service
After=network.target nginx.service
Requires=nginx.service

[Service]
Type=oneshot
ExecStart=/bin/echo "Service started after nginx"
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
```

**Exercise 2.2.5: Timer Units (Scheduling)**
```bash
# Create a systemd timer
sudo nano /etc/systemd/system/my-backup.timer

# Add:
[Unit]
Description=Run backup daily
Requires=my-backup.service

[Timer]
OnCalendar=daily
OnCalendar=Mon..Fri 02:00
Persistent=true

[Install]
WantedBy=timers.target

sudo nano /etc/systemd/system/my-backup.service

# Add:
[Unit]
Description=Backup Service

[Service]
Type=oneshot
ExecStart=/home/your-username/practice/scripts/backup.sh

# Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable my-backup.timer
sudo systemctl start my-backup.timer
sudo systemctl status my-backup.timer
```

**Practice Scenario from Q&A:**
```bash
# Scenario: Slow boot process - optimize systemd
systemd-analyze blame
# Identify slow services
sudo systemctl disable slow-service-name
sudo systemctl mask unnecessary-service
journalctl -xb | grep -i error
# Review and optimize service configurations
```

---

#### **MODULE 2.3: Shell Scripting & Automation**

**Exercise 2.3.1: Basic Bash Scripting**
```bash
# Create practice script directory
mkdir -p ~/practice/scripts

# Script 1: Basic variables and conditionals
cat > ~/practice/scripts/script1.sh << 'EOF'
#!/bin/bash
# Script to check disk usage

THRESHOLD=85
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

if [ $DISK_USAGE -gt $THRESHOLD ]; then
    echo "WARNING: Disk usage is ${DISK_USAGE}%"
    exit 1
else
    echo "Disk usage is OK: ${DISK_USAGE}%"
    exit 0
fi
EOF

chmod +x ~/practice/scripts/script1.sh
~/practice/scripts/script1.sh
```

**Exercise 2.3.2: Functions and Error Handling**
```bash
# Script 2: Functions with error handling
cat > ~/practice/scripts/script2.sh << 'EOF'
#!/bin/bash
set -e  # Exit on error
set -u  # Exit on undefined variable

LOG_FILE="/tmp/script.log"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

backup_file() {
    local source_file=$1
    local backup_dir=$2
    
    if [ ! -f "$source_file" ]; then
        log_message "ERROR: File $source_file does not exist"
        return 1
    fi
    
    mkdir -p "$backup_dir"
    cp "$source_file" "$backup_dir/"
    log_message "Backed up $source_file to $backup_dir"
}

# Usage
backup_file "/etc/hosts" "/tmp/backups"
EOF

chmod +x ~/practice/scripts/script2.sh
```

**Exercise 2.3.3: Log Parsing Script**

```bash
# Script 3: Parse logs and generate report
cat > ~/practice/scripts/log_parser.sh << 'EOF'
#!/bin/bash
# Parse system logs and generate report

LOG_FILE="/var/log/syslog"
OUTPUT_FILE="/tmp/log_report.txt"

echo "=== Log Analysis Report ===" > "$OUTPUT_FILE"
echo "Generated: $(date)" >> "$OUTPUT_FILE"
echo "" >> "$OUTPUT_FILE"

echo "Top 10 error messages:" >> "$OUTPUT_FILE"
grep -i error "$LOG_FILE" | tail -10 >> "$OUTPUT_FILE"
echo "" >> "$OUTPUT_FILE"

echo "SSH connection attempts:" >> "$OUTPUT_FILE"
grep ssh "$LOG_FILE" | wc -l >> "$OUTPUT_FILE"
echo "" >> "$OUTPUT_FILE"

echo "Failed login attempts:" >> "$OUTPUT_FILE"
grep "Failed password" "$LOG_FILE" | wc -l >> "$OUTPUT_FILE"

cat "$OUTPUT_FILE"
EOF

chmod +x ~/practice/scripts/log_parser.sh
sudo ~/practice/scripts/log_parser.sh  # Needs sudo for /var/log/syslog
```

**Exercise 2.3.4: Deployment Script**
```bash
# Script 4: Deployment automation
cat > ~/practice/scripts/deploy.sh << 'EOF'
#!/bin/bash
# Deployment script with rollback capability

APP_DIR="/var/www/myapp"
BACKUP_DIR="/tmp/backups"
DEPLOY_DIR="/tmp/deployment"

# Create backup
backup_app() {
    echo "Creating backup..."
    mkdir -p "$BACKUP_DIR"
    tar -czf "$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S).tar.gz" "$APP_DIR"
    echo "Backup created"
}

# Deploy application
deploy_app() {
    echo "Deploying application..."
    
    if [ ! -d "$DEPLOY_DIR" ]; then
        echo "ERROR: Deployment directory not found"
        exit 1
    fi
    
    backup_app
    
    # Stop service
    sudo systemctl stop myapp || true
    
    # Copy files
    cp -r "$DEPLOY_DIR"/* "$APP_DIR/"
    
    # Start service
    sudo systemctl start myapp
    
    # Check status
    if systemctl is-active --quiet myapp; then
        echo "Deployment successful"
    else
        echo "ERROR: Deployment failed, rolling back..."
        rollback_app
        exit 1
    fi
}

# Rollback function
rollback_app() {
    echo "Rolling back..."
    LATEST_BACKUP=$(ls -t "$BACKUP_DIR"/*.tar.gz | head -1)
    sudo systemctl stop myapp
    tar -xzf "$LATEST_BACKUP" -C /
    sudo systemctl start myapp
    echo "Rollback complete"
}

# Main
case "$1" in
    deploy)
        deploy_app
        ;;
    rollback)
        rollback_app
        ;;
    *)
        echo "Usage: $0 {deploy|rollback}"
        exit 1
        ;;
esac
EOF

chmod +x ~/practice/scripts/deploy.sh
```

**Exercise 2.3.5: Cron Job Management**
```bash
# 1. List current crontabs
crontab -l

# 2. Edit crontab
crontab -e

# Add examples:
# Run every minute
# * * * * * /path/to/script.sh

# Run every Sunday at 2 AM
# 0 2 * * 0 /path/to/backup.sh

# Run every weekday at 9 AM
# 0 9 * * 1-5 /path/to/script.sh

# Run every 30 minutes
# */30 * * * * /path/to/script.sh

# 3. Create cron job script
cat > ~/practice/scripts/daily_backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/tmp/backups"
SOURCE_DIR="/etc"
DATE=$(date +%Y%m%d)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/config-backup-$DATE.tar.gz" "$SOURCE_DIR"

# Keep only last 7 backups
find "$BACKUP_DIR" -name "config-backup-*.tar.gz" -mtime +7 -delete
EOF

chmod +x ~/practice/scripts/daily_backup.sh

# 4. Schedule cron job
crontab -e
# Add: 0 2 * * 0 /home/your-username/practice/scripts/daily_backup.sh

# 5. Monitor cron logs
grep CRON /var/log/syslog
journalctl -u cron
```

**Practice Scenario from Q&A:**
```bash
# Scenario: Create cron job for Sunday backup at 2 AM
crontab -e
# Add: 0 2 * * 0 /path/to/backup_script.sh
chmod +x /path/to/backup_script.sh
# Verify
crontab -l
```

---

#### **MODULE 2.4: Networking & Connectivity**

**Exercise 2.4.1: Network Configuration**
```bash
# 1. Check network interfaces
ip addr show
ifconfig  # If net-tools installed
ip link show

# 2. Check routing table
ip route show
route -n

# 3. Configure static IP (using Netplan on newer Debian or /etc/network/interfaces)
sudo nano /etc/network/interfaces

# Add:
# auto eth0
# iface eth0 inet static
#     address 192.168.1.100
#     netmask 255.255.255.0
#     gateway 192.168.1.1
#     dns-nameservers 8.8.8.8 8.8.4.4

# Or use NetworkManager
sudo nmcli connection modify "Wired connection 1" ipv4.addresses 192.168.1.100/24
sudo nmcli connection modify "Wired connection 1" ipv4.gateway 192.168.1.1
sudo nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli connection modify "Wired connection 1" ipv4.method manual
sudo nmcli connection up "Wired connection 1"

# 4. Test connectivity
ping -c 4 8.8.8.8
ping -c 4 google.com
traceroute google.com
```

**Exercise 2.4.2: DNS Configuration**
```bash
# 1. Check DNS configuration
cat /etc/resolv.conf

# 2. Configure DNS (systemd-resolved)
sudo nano /etc/systemd/resolved.conf

# Uncomment and modify:
# [Resolve]
# DNS=8.8.8.8 8.8.4.4
# FallbackDNS=1.1.1.1

sudo systemctl restart systemd-resolved

# 3. Test DNS resolution
nslookup google.com
dig google.com
host google.com

# 4. Check DNS cache
systemd-resolve --status
```

**Exercise 2.4.3: Firewall Configuration (UFW)**
```bash
# 1. Check UFW status
sudo ufw status verbose

# 2. Enable UFW
sudo ufw enable

# 3. Allow specific ports
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS
sudo ufw allow from 192.168.1.0/24  # Allow from subnet

# 4. Deny specific ports
sudo ufw deny 8080/tcp

# 5. Delete rules
sudo ufw status numbered
sudo ufw delete 2

# 6. Reset firewall
sudo ufw reset

# 7. Check logs
sudo tail -f /var/log/ufw.log
```

**Exercise 2.4.4: Firewall Configuration (iptables)**
```bash
# 1. List current rules
sudo iptables -L -n -v
sudo iptables -L INPUT -n -v

# 2. Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 3. Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 4. Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# 5. Block IP
sudo iptables -A INPUT -s 192.168.1.200 -j DROP

# 6. Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# 7. Save rules
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

**Exercise 2.4.5: Network Troubleshooting**
```bash
# 1. Check listening ports
sudo ss -tulpn
sudo netstat -tulpn

# 2. Check process using port
sudo lsof -i :80
sudo fuser 80/tcp

# 3. Monitor network traffic
sudo tcpdump -i eth0
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 host 192.168.1.100

# 4. Check network statistics
ss -s
netstat -s

# 5. Test bandwidth
sudo apt install iperf3
# On server: iperf3 -s
# On client: iperf3 -c server-ip
```

**Practice Scenario from Q&A:**
```bash
# Scenario: Configure static IP
sudo nano /etc/network/interfaces
# Add configuration
sudo systemctl restart networking
ip addr show
ping -c 4 8.8.8.8
```

---

### **PHASE 3: MEDIUM PRIORITY TOPICS (DevOps Skills)**

#### **MODULE 3.1: Security & Hardening**

**Exercise 3.1.1: SSH Hardening**
```bash
# 1. Backup SSH config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# 2. Edit SSH config
sudo nano /etc/ssh/sshd_config

# Key settings:
# Port 2222  # Change default port
# PermitRootLogin no
# PasswordAuthentication no
# PubkeyAuthentication yes
# AllowUsers username1 username2
# MaxAuthTries 3
# ClientAliveInterval 300
# ClientAliveCountMax 2

# 3. Generate SSH keys
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 4. Copy public key to server
ssh-copy-id username@server-ip

# 5. Test SSH config
sudo sshd -t

# 6. Restart SSH service
sudo systemctl restart sshd
```

**Exercise 3.1.2: Fail2ban Setup**
```bash
# 1. Install fail2ban
sudo apt install -y fail2ban

# 2. Configure fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Configure:
# [sshd]
# enabled = true
# port = ssh
# filter = sshd
# logpath = /var/log/auth.log
# maxretry = 3
# bantime = 3600

# 3. Start and enable fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 4. Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# 5. Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

**Exercise 3.1.3: Security Updates**
```bash
# 1. Configure automatic security updates
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# 2. Configure unattended-upgrades
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

# Uncomment:
# "origin=Debian,codename=${distro_codename},label=Debian-Security";

# 3. Test updates
sudo unattended-upgrades --dry-run --debug

# 4. Manual security updates
sudo apt update
sudo apt upgrade
sudo apt list --upgradable
```

**Exercise 3.1.4: File Permissions & ACL**
```bash
# 1. Check current permissions
ls -l /etc/passwd
stat /etc/passwd

# 2. Set permissions
chmod 644 file.txt
chmod u+x script.sh
chmod -R 755 directory/

# 3. Change ownership
sudo chown user:group file.txt
sudo chown -R user:group directory/

# 4. Install ACL
sudo apt install -y acl

# 5. Set ACL
sudo setfacl -m u:username:rwx /path/to/file
sudo setfacl -m g:groupname:r-x /path/to/directory

# 6. View ACL
getfacl /path/to/file

# 7. Remove ACL
sudo setfacl -x u:username /path/to/file
```

---

#### **MODULE 3.2: Docker Containerization**

**Exercise 3.2.1: Docker Installation & Basics**
```bash
# 1. Install Docker
sudo apt update
sudo apt install -y docker.io docker-compose

# 2. Start Docker service
sudo systemctl enable docker
sudo systemctl start docker

# 3. Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# 4. Test Docker
docker --version
docker run hello-world

# 5. Basic Docker commands
docker ps
docker ps -a
docker images
docker info
```

**Exercise 3.2.2: Dockerfile Creation**
```bash
# Create Dockerfile
mkdir -p ~/practice/docker
cat > ~/practice/docker/Dockerfile << 'EOF'
FROM debian:stable-slim

RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY index.html /var/www/html/
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
EOF

# Create index.html
echo "<h1>Hello from Docker!</h1>" > ~/practice/docker/index.html

# Build image
cd ~/practice/docker
docker build -t my-nginx:latest .

# Run container
docker run -d -p 8080:80 --name my-nginx my-nginx:latest

# Test
curl http://localhost:8080
```

**Exercise 3.2.3: Docker Compose**
```bash
# Create docker-compose.yml
cat > ~/practice/docker/docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped
  
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  db_data:
EOF

# Create html directory
mkdir -p ~/practice/docker/html
echo "<h1>Docker Compose!</h1>" > ~/practice/docker/html/index.html

# Run compose
cd ~/practice/docker
docker-compose up -d
docker-compose ps
docker-compose logs
docker-compose down
```

---

#### **MODULE 3.3: CI/CD Integration**

**Exercise 3.3.1: Git Setup**
```bash
# 1. Install Git
sudo apt install -y git

# 2. Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 3. Create repository
mkdir -p ~/practice/git-repo
cd ~/practice/git-repo
git init
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# 4. Create branches
git checkout -b development
git checkout -b feature/new-feature
git branch

# 5. Merge branches
git checkout main
git merge feature/new-feature

# 6. Handle conflicts (create conflict scenario)
echo "Conflict line" > conflict.txt
git add conflict.txt
git commit -m "Add conflict file"
git checkout -b conflict-branch
echo "Different line" > conflict.txt
git commit -am "Change conflict file"
git checkout main
git merge conflict-branch
# Resolve conflict
git add conflict.txt
git commit -m "Resolve conflict"
```

**Exercise 3.3.2: GitHub Actions Workflow**
```bash
# Create GitHub Actions workflow
mkdir -p ~/practice/git-repo/.github/workflows
cat > ~/practice/git-repo/.github/workflows/deploy.yml << 'EOF'
name: Deploy Application

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        python -m pytest tests/
    
    - name: Build Docker image
      run: |
        docker build -t myapp:latest .
    
    - name: Deploy
      run: |
        echo "Deploying application..."
EOF

git add .github/workflows/deploy.yml
git commit -m "Add CI/CD workflow"
```

---

### **PHASE 4: PRACTICE SCENARIOS FROM Q&A**

#### **Scenario Practice 1: Troubleshoot Slow Boot**
```bash
# 1. Analyze boot time
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain

# 2. Disable unnecessary services
sudo systemctl list-unit-files --type=service | grep enabled
sudo systemctl disable service-name

# 3. Check logs
journalctl -xb | grep -i error
journalctl -u slow-service-name

# 4. Optimize service dependencies
sudo systemctl edit service-name
```

#### **Scenario Practice 2: Disk Space Management**
```bash
# 1. Check disk usage
df -h
du -h --max-depth=1 /

# 2. Find large files
find / -type f -size +100M -exec ls -lh {} \; | awk '{print $5, $9}'
find / -type f -size +100M

# 3. Find large directories
du -h / | sort -rh | head -20

# 4. Clean package cache
sudo apt clean
sudo apt autoclean
sudo apt autoremove

# 5. Clean logs
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=500M
```

#### **Scenario Practice 3: Service Failure Recovery**
```bash
# 1. Simulate service failure
sudo systemctl stop nginx
sudo systemctl status nginx

# 2. Check logs
journalctl -u nginx -n 50
journalctl -u nginx --since "1 hour ago"

# 3. Check configuration
sudo nginx -t

# 4. Restart service
sudo systemctl restart nginx

# 5. Enable auto-restart
sudo systemctl edit nginx
# Add:
# [Service]
# Restart=on-failure
# RestartSec=10
```

---

### **DAILY PRACTICE SCHEDULE**

**Day 1-2: Installation & Basics**
- Complete Debian installation
- Post-installation setup
- Module 2.1 (APT Package Management)

**Day 3-4: System Management**
- Module 2.2 (Systemd)
- Module 2.4 (Networking)

**Day 5-6: Scripting & Automation**
- Module 2.3 (Shell Scripting)
- Practice scenarios

**Day 7-8: Security & DevOps**
- Module 3.1 (Security)
- Module 3.2 (Docker)

**Day 9-10: Integration & Practice**
- Module 3.3 (CI/CD)
- Review all scenarios
- Mock test practice

---

### **TEST DAY CHECKLIST**

Before the test:
- [ ] Debian system running and accessible
- [ ] SSH access configured
- [ ] Basic tools installed
- [ ] Practice directories created
- [ ] Documentation ready
- [ ] Backup system available

During the test:
- [ ] Read questions carefully
- [ ] Show your thought process
- [ ] Use proper commands
- [ ] Handle edge cases
- [ ] Document your work

---

This guide covers the topics needed for the Turing test. Practice each module hands-on, and work through the scenarios from the Q&A document. Focus on understanding why each command works, not just memorizing commands.

Should I create a specific practice file for any module, or do you want more detail on any section?