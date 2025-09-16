# graylog Installation Guide

graylog is a free and open-source centralized log management platform. Graylog provides log collection, processing, and analysis capabilities, serving as an open-source alternative to Splunk or Datadog

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
  - CPU: 4+ cores recommended
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 50GB+ for logs
  - Network: Syslog and GELF inputs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9000 (default graylog port)
  - Ports 514, 12201 for log inputs
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

# Install graylog
sudo dnf install -y graylog-server

# Enable and start service
sudo systemctl enable --now graylog-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
graylogctl status
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install graylog
sudo apt install -y graylog-server

# Enable and start service
sudo systemctl enable --now graylog-server

# Configure firewall
sudo ufw allow 9000

# Verify installation
graylogctl status
```

### Arch Linux

```bash
# Install graylog
sudo pacman -S graylog-server

# Enable and start service
sudo systemctl enable --now graylog-server

# Verify installation
graylogctl status
```

### Alpine Linux

```bash
# Install graylog
apk add --no-cache graylog-server

# Enable and start service
rc-update add graylog-server default
rc-service graylog-server start

# Verify installation
graylogctl status
```

### openSUSE/SLES

```bash
# Install graylog
sudo zypper install -y graylog-server

# Enable and start service
sudo systemctl enable --now graylog-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
graylogctl status
```

### macOS

```bash
# Using Homebrew
brew install graylog-server

# Start service
brew services start graylog-server

# Verify installation
graylogctl status
```

### FreeBSD

```bash
# Using pkg
pkg install graylog-server

# Enable in rc.conf
echo 'graylog-server_enable="YES"' >> /etc/rc.conf

# Start service
service graylog-server start

# Verify installation
graylogctl status
```

### Windows

```bash
# Using Chocolatey
choco install graylog-server

# Or using Scoop
scoop install graylog-server

# Verify installation
graylogctl status
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/graylog-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
graylogctl status
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable graylog-server

# Start service
sudo systemctl start graylog-server

# Stop service
sudo systemctl stop graylog-server

# Restart service
sudo systemctl restart graylog-server

# Check status
sudo systemctl status graylog-server

# View logs
sudo journalctl -u graylog-server -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add graylog-server default

# Start service
rc-service graylog-server start

# Stop service
rc-service graylog-server stop

# Restart service
rc-service graylog-server restart

# Check status
rc-service graylog-server status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'graylog-server_enable="YES"' >> /etc/rc.conf

# Start service
service graylog-server start

# Stop service
service graylog-server stop

# Restart service
service graylog-server restart

# Check status
service graylog-server status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start graylog-server
brew services stop graylog-server
brew services restart graylog-server

# Check status
brew services list | grep graylog-server
```

### Windows Service Manager

```powershell
# Start service
net start graylog-server

# Stop service
net stop graylog-server

# Using PowerShell
Start-Service graylog-server
Stop-Service graylog-server
Restart-Service graylog-server

# Check status
Get-Service graylog-server
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream graylog-server_backend {
    server 127.0.0.1:9000;
}

server {
    listen 80;
    server_name graylog-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name graylog-server.example.com;

    ssl_certificate /etc/ssl/certs/graylog-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/graylog-server.example.com.key;

    location / {
        proxy_pass http://graylog-server_backend;
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
    ServerName graylog-server.example.com
    Redirect permanent / https://graylog-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName graylog-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/graylog-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/graylog-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend graylog-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/graylog-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend graylog-server_backend

backend graylog-server_backend
    balance roundrobin
    server graylog-server1 127.0.0.1:9000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R graylog-server:graylog-server /etc/graylog-server
sudo chmod 750 /etc/graylog-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
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
sudo systemctl status graylog-server

# View logs
sudo journalctl -u graylog-server -f

# Monitor resource usage
top -p $(pgrep graylog-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/graylog-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/graylog-server-backup-$DATE.tar.gz" /etc/graylog-server /var/lib/graylog-server

echo "Backup completed: $BACKUP_DIR/graylog-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop graylog-server

# Restore from backup
tar -xzf /backup/graylog-server/graylog-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start graylog-server
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u graylog-server -n 100
sudo tail -f /var/log/graylog-server/graylog-server.log

# Check configuration
graylogctl status

# Check permissions
ls -la /etc/graylog-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000

# Test connectivity
telnet localhost 9000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep graylog-server)

# Check disk I/O
iotop -p $(pgrep graylog-server)

# Check connections
ss -an | grep 9000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  graylog-server:
    image: graylog-server:latest
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/graylog-server
      - ./data:/var/lib/graylog-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update graylog-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade graylog-server

# Arch Linux
sudo pacman -Syu graylog-server

# Alpine Linux
apk update && apk upgrade graylog-server

# openSUSE
sudo zypper update graylog-server

# FreeBSD
pkg update && pkg upgrade graylog-server

# Always backup before updates
tar -czf /backup/graylog-server-pre-update-$(date +%Y%m%d).tar.gz /etc/graylog-server

# Restart after updates
sudo systemctl restart graylog-server
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/graylog-server

# Clean old logs
find /var/log/graylog-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/graylog-server
```

## Additional Resources

- Official Documentation: https://docs.graylog-server.org/
- GitHub Repository: https://github.com/graylog-server/graylog-server
- Community Forum: https://forum.graylog-server.org/
- Best Practices Guide: https://docs.graylog-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
