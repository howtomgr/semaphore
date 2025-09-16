# semaphore Installation Guide

semaphore is a free and open-source modern UI for Ansible. Semaphore provides a web UI for Ansible with scheduling, notifications, and access control, serving as an open-source alternative to Ansible Tower/AWX

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 1GB for application
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default semaphore port)
  - Database connection required
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install semaphore
sudo dnf install -y semaphore

# Enable and start service
sudo systemctl enable --now semaphore

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
semaphore --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install semaphore
sudo apt install -y semaphore

# Enable and start service
sudo systemctl enable --now semaphore

# Configure firewall
sudo ufw allow 3000

# Verify installation
semaphore --version
```

### Arch Linux

```bash
# Install semaphore
sudo pacman -S semaphore

# Enable and start service
sudo systemctl enable --now semaphore

# Verify installation
semaphore --version
```

### Alpine Linux

```bash
# Install semaphore
apk add --no-cache semaphore

# Enable and start service
rc-update add semaphore default
rc-service semaphore start

# Verify installation
semaphore --version
```

### openSUSE/SLES

```bash
# Install semaphore
sudo zypper install -y semaphore

# Enable and start service
sudo systemctl enable --now semaphore

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
semaphore --version
```

### macOS

```bash
# Using Homebrew
brew install semaphore

# Start service
brew services start semaphore

# Verify installation
semaphore --version
```

### FreeBSD

```bash
# Using pkg
pkg install semaphore

# Enable in rc.conf
echo 'semaphore_enable="YES"' >> /etc/rc.conf

# Start service
service semaphore start

# Verify installation
semaphore --version
```

### Windows

```bash
# Using Chocolatey
choco install semaphore

# Or using Scoop
scoop install semaphore

# Verify installation
semaphore --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/semaphore

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
semaphore --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable semaphore

# Start service
sudo systemctl start semaphore

# Stop service
sudo systemctl stop semaphore

# Restart service
sudo systemctl restart semaphore

# Check status
sudo systemctl status semaphore

# View logs
sudo journalctl -u semaphore -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add semaphore default

# Start service
rc-service semaphore start

# Stop service
rc-service semaphore stop

# Restart service
rc-service semaphore restart

# Check status
rc-service semaphore status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'semaphore_enable="YES"' >> /etc/rc.conf

# Start service
service semaphore start

# Stop service
service semaphore stop

# Restart service
service semaphore restart

# Check status
service semaphore status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start semaphore
brew services stop semaphore
brew services restart semaphore

# Check status
brew services list | grep semaphore
```

### Windows Service Manager

```powershell
# Start service
net start semaphore

# Stop service
net stop semaphore

# Using PowerShell
Start-Service semaphore
Stop-Service semaphore
Restart-Service semaphore

# Check status
Get-Service semaphore
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream semaphore_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name semaphore.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name semaphore.example.com;

    ssl_certificate /etc/ssl/certs/semaphore.example.com.crt;
    ssl_certificate_key /etc/ssl/private/semaphore.example.com.key;

    location / {
        proxy_pass http://semaphore_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName semaphore.example.com
    Redirect permanent / https://semaphore.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName semaphore.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/semaphore.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/semaphore.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend semaphore_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/semaphore.pem
    redirect scheme https if !{ ssl_fc }
    default_backend semaphore_backend

backend semaphore_backend
    balance roundrobin
    server semaphore1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R semaphore:semaphore /etc/semaphore
sudo chmod 750 /etc/semaphore

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status semaphore

# View logs
sudo journalctl -u semaphore -f

# Monitor resource usage
top -p $(pgrep semaphore)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/semaphore"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/semaphore-backup-$DATE.tar.gz" /etc/semaphore /var/lib/semaphore

echo "Backup completed: $BACKUP_DIR/semaphore-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop semaphore

# Restore from backup
tar -xzf /backup/semaphore/semaphore-backup-*.tar.gz -C /

# Start service
sudo systemctl start semaphore
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u semaphore -n 100
sudo tail -f /var/log/semaphore/semaphore.log

# Check configuration
semaphore --version

# Check permissions
ls -la /etc/semaphore
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep semaphore)

# Check disk I/O
iotop -p $(pgrep semaphore)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  semaphore:
    image: semaphore:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/semaphore
      - ./data:/var/lib/semaphore
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update semaphore

# Debian/Ubuntu
sudo apt update && sudo apt upgrade semaphore

# Arch Linux
sudo pacman -Syu semaphore

# Alpine Linux
apk update && apk upgrade semaphore

# openSUSE
sudo zypper update semaphore

# FreeBSD
pkg update && pkg upgrade semaphore

# Always backup before updates
tar -czf /backup/semaphore-pre-update-$(date +%Y%m%d).tar.gz /etc/semaphore

# Restart after updates
sudo systemctl restart semaphore
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/semaphore

# Clean old logs
find /var/log/semaphore -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/semaphore
```

## Additional Resources

- Official Documentation: https://docs.semaphore.org/
- GitHub Repository: https://github.com/semaphore/semaphore
- Community Forum: https://forum.semaphore.org/
- Best Practices Guide: https://docs.semaphore.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
