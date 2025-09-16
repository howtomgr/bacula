# bacula Installation Guide

bacula is a free and open-source backup system. Bacula provides enterprise-ready network backup solution

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
  - Storage: 100GB for backups
  - Network: Network backup
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9101 (default bacula port)
  - Director 9101-9103
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

# Install bacula
sudo dnf install -y bacula

# Enable and start service
sudo systemctl enable --now bacula

# Configure firewall
sudo firewall-cmd --permanent --add-port=9101/tcp
sudo firewall-cmd --reload

# Verify installation
bacula --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install bacula
sudo apt install -y bacula

# Enable and start service
sudo systemctl enable --now bacula

# Configure firewall
sudo ufw allow 9101

# Verify installation
bacula --version
```

### Arch Linux

```bash
# Install bacula
sudo pacman -S bacula

# Enable and start service
sudo systemctl enable --now bacula

# Verify installation
bacula --version
```

### Alpine Linux

```bash
# Install bacula
apk add --no-cache bacula

# Enable and start service
rc-update add bacula default
rc-service bacula start

# Verify installation
bacula --version
```

### openSUSE/SLES

```bash
# Install bacula
sudo zypper install -y bacula

# Enable and start service
sudo systemctl enable --now bacula

# Configure firewall
sudo firewall-cmd --permanent --add-port=9101/tcp
sudo firewall-cmd --reload

# Verify installation
bacula --version
```

### macOS

```bash
# Using Homebrew
brew install bacula

# Start service
brew services start bacula

# Verify installation
bacula --version
```

### FreeBSD

```bash
# Using pkg
pkg install bacula

# Enable in rc.conf
echo 'bacula_enable="YES"' >> /etc/rc.conf

# Start service
service bacula start

# Verify installation
bacula --version
```

### Windows

```bash
# Using Chocolatey
choco install bacula

# Or using Scoop
scoop install bacula

# Verify installation
bacula --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/bacula

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
bacula --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable bacula

# Start service
sudo systemctl start bacula

# Stop service
sudo systemctl stop bacula

# Restart service
sudo systemctl restart bacula

# Check status
sudo systemctl status bacula

# View logs
sudo journalctl -u bacula -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add bacula default

# Start service
rc-service bacula start

# Stop service
rc-service bacula stop

# Restart service
rc-service bacula restart

# Check status
rc-service bacula status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'bacula_enable="YES"' >> /etc/rc.conf

# Start service
service bacula start

# Stop service
service bacula stop

# Restart service
service bacula restart

# Check status
service bacula status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start bacula
brew services stop bacula
brew services restart bacula

# Check status
brew services list | grep bacula
```

### Windows Service Manager

```powershell
# Start service
net start bacula

# Stop service
net stop bacula

# Using PowerShell
Start-Service bacula
Stop-Service bacula
Restart-Service bacula

# Check status
Get-Service bacula
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream bacula_backend {
    server 127.0.0.1:9101;
}

server {
    listen 80;
    server_name bacula.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name bacula.example.com;

    ssl_certificate /etc/ssl/certs/bacula.example.com.crt;
    ssl_certificate_key /etc/ssl/private/bacula.example.com.key;

    location / {
        proxy_pass http://bacula_backend;
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
    ServerName bacula.example.com
    Redirect permanent / https://bacula.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName bacula.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/bacula.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/bacula.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9101/
    ProxyPassReverse / http://127.0.0.1:9101/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend bacula_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/bacula.pem
    redirect scheme https if !{ ssl_fc }
    default_backend bacula_backend

backend bacula_backend
    balance roundrobin
    server bacula1 127.0.0.1:9101 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R bacula:bacula /etc/bacula
sudo chmod 750 /etc/bacula

# Configure firewall
sudo firewall-cmd --permanent --add-port=9101/tcp
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
sudo systemctl status bacula

# View logs
sudo journalctl -u bacula -f

# Monitor resource usage
top -p $(pgrep bacula)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/bacula"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/bacula-backup-$DATE.tar.gz" /etc/bacula /var/lib/bacula

echo "Backup completed: $BACKUP_DIR/bacula-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop bacula

# Restore from backup
tar -xzf /backup/bacula/bacula-backup-*.tar.gz -C /

# Start service
sudo systemctl start bacula
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u bacula -n 100
sudo tail -f /var/log/bacula/bacula.log

# Check configuration
bacula --version

# Check permissions
ls -la /etc/bacula
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9101

# Test connectivity
telnet localhost 9101

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep bacula)

# Check disk I/O
iotop -p $(pgrep bacula)

# Check connections
ss -an | grep 9101
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  bacula:
    image: bacula:latest
    ports:
      - "9101:9101"
    volumes:
      - ./config:/etc/bacula
      - ./data:/var/lib/bacula
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update bacula

# Debian/Ubuntu
sudo apt update && sudo apt upgrade bacula

# Arch Linux
sudo pacman -Syu bacula

# Alpine Linux
apk update && apk upgrade bacula

# openSUSE
sudo zypper update bacula

# FreeBSD
pkg update && pkg upgrade bacula

# Always backup before updates
tar -czf /backup/bacula-pre-update-$(date +%Y%m%d).tar.gz /etc/bacula

# Restart after updates
sudo systemctl restart bacula
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/bacula

# Clean old logs
find /var/log/bacula -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/bacula
```

## Additional Resources

- Official Documentation: https://docs.bacula.org/
- GitHub Repository: https://github.com/bacula/bacula
- Community Forum: https://forum.bacula.org/
- Best Practices Guide: https://docs.bacula.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
