# vernemq Installation Guide

vernemq is a free and open-source MQTT broker. VerneMQ provides scalable MQTT broker

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: MQTT protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1883 (default vernemq port)
  - MQTTS on 8883
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

# Install vernemq
sudo dnf install -y vernemq

# Enable and start service
sudo systemctl enable --now vernemq

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --reload

# Verify installation
vernemq --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install vernemq
sudo apt install -y vernemq

# Enable and start service
sudo systemctl enable --now vernemq

# Configure firewall
sudo ufw allow 1883

# Verify installation
vernemq --version
```

### Arch Linux

```bash
# Install vernemq
sudo pacman -S vernemq

# Enable and start service
sudo systemctl enable --now vernemq

# Verify installation
vernemq --version
```

### Alpine Linux

```bash
# Install vernemq
apk add --no-cache vernemq

# Enable and start service
rc-update add vernemq default
rc-service vernemq start

# Verify installation
vernemq --version
```

### openSUSE/SLES

```bash
# Install vernemq
sudo zypper install -y vernemq

# Enable and start service
sudo systemctl enable --now vernemq

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --reload

# Verify installation
vernemq --version
```

### macOS

```bash
# Using Homebrew
brew install vernemq

# Start service
brew services start vernemq

# Verify installation
vernemq --version
```

### FreeBSD

```bash
# Using pkg
pkg install vernemq

# Enable in rc.conf
echo 'vernemq_enable="YES"' >> /etc/rc.conf

# Start service
service vernemq start

# Verify installation
vernemq --version
```

### Windows

```bash
# Using Chocolatey
choco install vernemq

# Or using Scoop
scoop install vernemq

# Verify installation
vernemq --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/vernemq

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
vernemq --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable vernemq

# Start service
sudo systemctl start vernemq

# Stop service
sudo systemctl stop vernemq

# Restart service
sudo systemctl restart vernemq

# Check status
sudo systemctl status vernemq

# View logs
sudo journalctl -u vernemq -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add vernemq default

# Start service
rc-service vernemq start

# Stop service
rc-service vernemq stop

# Restart service
rc-service vernemq restart

# Check status
rc-service vernemq status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'vernemq_enable="YES"' >> /etc/rc.conf

# Start service
service vernemq start

# Stop service
service vernemq stop

# Restart service
service vernemq restart

# Check status
service vernemq status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start vernemq
brew services stop vernemq
brew services restart vernemq

# Check status
brew services list | grep vernemq
```

### Windows Service Manager

```powershell
# Start service
net start vernemq

# Stop service
net stop vernemq

# Using PowerShell
Start-Service vernemq
Stop-Service vernemq
Restart-Service vernemq

# Check status
Get-Service vernemq
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream vernemq_backend {
    server 127.0.0.1:1883;
}

server {
    listen 80;
    server_name vernemq.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name vernemq.example.com;

    ssl_certificate /etc/ssl/certs/vernemq.example.com.crt;
    ssl_certificate_key /etc/ssl/private/vernemq.example.com.key;

    location / {
        proxy_pass http://vernemq_backend;
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
    ServerName vernemq.example.com
    Redirect permanent / https://vernemq.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName vernemq.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/vernemq.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/vernemq.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1883/
    ProxyPassReverse / http://127.0.0.1:1883/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend vernemq_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/vernemq.pem
    redirect scheme https if !{ ssl_fc }
    default_backend vernemq_backend

backend vernemq_backend
    balance roundrobin
    server vernemq1 127.0.0.1:1883 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R vernemq:vernemq /etc/vernemq
sudo chmod 750 /etc/vernemq

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
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
sudo systemctl status vernemq

# View logs
sudo journalctl -u vernemq -f

# Monitor resource usage
top -p $(pgrep vernemq)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/vernemq"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/vernemq-backup-$DATE.tar.gz" /etc/vernemq /var/lib/vernemq

echo "Backup completed: $BACKUP_DIR/vernemq-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop vernemq

# Restore from backup
tar -xzf /backup/vernemq/vernemq-backup-*.tar.gz -C /

# Start service
sudo systemctl start vernemq
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u vernemq -n 100
sudo tail -f /var/log/vernemq/vernemq.log

# Check configuration
vernemq --version

# Check permissions
ls -la /etc/vernemq
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1883

# Test connectivity
telnet localhost 1883

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep vernemq)

# Check disk I/O
iotop -p $(pgrep vernemq)

# Check connections
ss -an | grep 1883
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  vernemq:
    image: vernemq:latest
    ports:
      - "1883:1883"
    volumes:
      - ./config:/etc/vernemq
      - ./data:/var/lib/vernemq
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update vernemq

# Debian/Ubuntu
sudo apt update && sudo apt upgrade vernemq

# Arch Linux
sudo pacman -Syu vernemq

# Alpine Linux
apk update && apk upgrade vernemq

# openSUSE
sudo zypper update vernemq

# FreeBSD
pkg update && pkg upgrade vernemq

# Always backup before updates
tar -czf /backup/vernemq-pre-update-$(date +%Y%m%d).tar.gz /etc/vernemq

# Restart after updates
sudo systemctl restart vernemq
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/vernemq

# Clean old logs
find /var/log/vernemq -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/vernemq
```

## Additional Resources

- Official Documentation: https://docs.vernemq.org/
- GitHub Repository: https://github.com/vernemq/vernemq
- Community Forum: https://forum.vernemq.org/
- Best Practices Guide: https://docs.vernemq.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
