# duplicati Installation Guide

duplicati is a free and open-source backup client. Duplicati provides encrypted backups to cloud storage services

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
  - RAM: 512MB minimum
  - Storage: 10GB for cache
  - Network: Cloud storage APIs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8200 (default duplicati port)
  - None
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

# Install duplicati
sudo dnf install -y duplicati

# Enable and start service
sudo systemctl enable --now duplicati

# Configure firewall
sudo firewall-cmd --permanent --add-port=8200/tcp
sudo firewall-cmd --reload

# Verify installation
duplicati --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install duplicati
sudo apt install -y duplicati

# Enable and start service
sudo systemctl enable --now duplicati

# Configure firewall
sudo ufw allow 8200

# Verify installation
duplicati --version
```

### Arch Linux

```bash
# Install duplicati
sudo pacman -S duplicati

# Enable and start service
sudo systemctl enable --now duplicati

# Verify installation
duplicati --version
```

### Alpine Linux

```bash
# Install duplicati
apk add --no-cache duplicati

# Enable and start service
rc-update add duplicati default
rc-service duplicati start

# Verify installation
duplicati --version
```

### openSUSE/SLES

```bash
# Install duplicati
sudo zypper install -y duplicati

# Enable and start service
sudo systemctl enable --now duplicati

# Configure firewall
sudo firewall-cmd --permanent --add-port=8200/tcp
sudo firewall-cmd --reload

# Verify installation
duplicati --version
```

### macOS

```bash
# Using Homebrew
brew install duplicati

# Start service
brew services start duplicati

# Verify installation
duplicati --version
```

### FreeBSD

```bash
# Using pkg
pkg install duplicati

# Enable in rc.conf
echo 'duplicati_enable="YES"' >> /etc/rc.conf

# Start service
service duplicati start

# Verify installation
duplicati --version
```

### Windows

```bash
# Using Chocolatey
choco install duplicati

# Or using Scoop
scoop install duplicati

# Verify installation
duplicati --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/duplicati

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
duplicati --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable duplicati

# Start service
sudo systemctl start duplicati

# Stop service
sudo systemctl stop duplicati

# Restart service
sudo systemctl restart duplicati

# Check status
sudo systemctl status duplicati

# View logs
sudo journalctl -u duplicati -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add duplicati default

# Start service
rc-service duplicati start

# Stop service
rc-service duplicati stop

# Restart service
rc-service duplicati restart

# Check status
rc-service duplicati status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'duplicati_enable="YES"' >> /etc/rc.conf

# Start service
service duplicati start

# Stop service
service duplicati stop

# Restart service
service duplicati restart

# Check status
service duplicati status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start duplicati
brew services stop duplicati
brew services restart duplicati

# Check status
brew services list | grep duplicati
```

### Windows Service Manager

```powershell
# Start service
net start duplicati

# Stop service
net stop duplicati

# Using PowerShell
Start-Service duplicati
Stop-Service duplicati
Restart-Service duplicati

# Check status
Get-Service duplicati
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream duplicati_backend {
    server 127.0.0.1:8200;
}

server {
    listen 80;
    server_name duplicati.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name duplicati.example.com;

    ssl_certificate /etc/ssl/certs/duplicati.example.com.crt;
    ssl_certificate_key /etc/ssl/private/duplicati.example.com.key;

    location / {
        proxy_pass http://duplicati_backend;
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
    ServerName duplicati.example.com
    Redirect permanent / https://duplicati.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName duplicati.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/duplicati.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/duplicati.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8200/
    ProxyPassReverse / http://127.0.0.1:8200/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend duplicati_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/duplicati.pem
    redirect scheme https if !{ ssl_fc }
    default_backend duplicati_backend

backend duplicati_backend
    balance roundrobin
    server duplicati1 127.0.0.1:8200 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R duplicati:duplicati /etc/duplicati
sudo chmod 750 /etc/duplicati

# Configure firewall
sudo firewall-cmd --permanent --add-port=8200/tcp
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
sudo systemctl status duplicati

# View logs
sudo journalctl -u duplicati -f

# Monitor resource usage
top -p $(pgrep duplicati)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/duplicati"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/duplicati-backup-$DATE.tar.gz" /etc/duplicati /var/lib/duplicati

echo "Backup completed: $BACKUP_DIR/duplicati-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop duplicati

# Restore from backup
tar -xzf /backup/duplicati/duplicati-backup-*.tar.gz -C /

# Start service
sudo systemctl start duplicati
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u duplicati -n 100
sudo tail -f /var/log/duplicati/duplicati.log

# Check configuration
duplicati --version

# Check permissions
ls -la /etc/duplicati
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8200

# Test connectivity
telnet localhost 8200

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep duplicati)

# Check disk I/O
iotop -p $(pgrep duplicati)

# Check connections
ss -an | grep 8200
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  duplicati:
    image: duplicati:latest
    ports:
      - "8200:8200"
    volumes:
      - ./config:/etc/duplicati
      - ./data:/var/lib/duplicati
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update duplicati

# Debian/Ubuntu
sudo apt update && sudo apt upgrade duplicati

# Arch Linux
sudo pacman -Syu duplicati

# Alpine Linux
apk update && apk upgrade duplicati

# openSUSE
sudo zypper update duplicati

# FreeBSD
pkg update && pkg upgrade duplicati

# Always backup before updates
tar -czf /backup/duplicati-pre-update-$(date +%Y%m%d).tar.gz /etc/duplicati

# Restart after updates
sudo systemctl restart duplicati
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/duplicati

# Clean old logs
find /var/log/duplicati -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/duplicati
```

## Additional Resources

- Official Documentation: https://docs.duplicati.org/
- GitHub Repository: https://github.com/duplicati/duplicati
- Community Forum: https://forum.duplicati.org/
- Best Practices Guide: https://docs.duplicati.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
