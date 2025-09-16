# pihole Installation Guide

pihole is a free and open-source network-wide ad blocker. Pi-hole acts as a DNS sinkhole to block ads and trackers network-wide, providing privacy and performance benefits

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
  - Storage: 200MB for installation
  - Network: DNS and web interface
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 53 (default pihole port)
  - Port 80/443 for admin
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

# Install pihole
sudo dnf install -y pihole

# Enable and start service
sudo systemctl enable --now pihole-FTL

# Configure firewall
sudo firewall-cmd --permanent --add-port=53/tcp
sudo firewall-cmd --reload

# Verify installation
pihole -v
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install pihole
sudo apt install -y pihole

# Enable and start service
sudo systemctl enable --now pihole-FTL

# Configure firewall
sudo ufw allow 53

# Verify installation
pihole -v
```

### Arch Linux

```bash
# Install pihole
sudo pacman -S pihole

# Enable and start service
sudo systemctl enable --now pihole-FTL

# Verify installation
pihole -v
```

### Alpine Linux

```bash
# Install pihole
apk add --no-cache pihole

# Enable and start service
rc-update add pihole-FTL default
rc-service pihole-FTL start

# Verify installation
pihole -v
```

### openSUSE/SLES

```bash
# Install pihole
sudo zypper install -y pihole

# Enable and start service
sudo systemctl enable --now pihole-FTL

# Configure firewall
sudo firewall-cmd --permanent --add-port=53/tcp
sudo firewall-cmd --reload

# Verify installation
pihole -v
```

### macOS

```bash
# Using Homebrew
brew install pihole

# Start service
brew services start pihole

# Verify installation
pihole -v
```

### FreeBSD

```bash
# Using pkg
pkg install pihole

# Enable in rc.conf
echo 'pihole-FTL_enable="YES"' >> /etc/rc.conf

# Start service
service pihole-FTL start

# Verify installation
pihole -v
```

### Windows

```bash
# Using Chocolatey
choco install pihole

# Or using Scoop
scoop install pihole

# Verify installation
pihole -v
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/pihole

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
pihole -v
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable pihole-FTL

# Start service
sudo systemctl start pihole-FTL

# Stop service
sudo systemctl stop pihole-FTL

# Restart service
sudo systemctl restart pihole-FTL

# Check status
sudo systemctl status pihole-FTL

# View logs
sudo journalctl -u pihole-FTL -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add pihole-FTL default

# Start service
rc-service pihole-FTL start

# Stop service
rc-service pihole-FTL stop

# Restart service
rc-service pihole-FTL restart

# Check status
rc-service pihole-FTL status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'pihole-FTL_enable="YES"' >> /etc/rc.conf

# Start service
service pihole-FTL start

# Stop service
service pihole-FTL stop

# Restart service
service pihole-FTL restart

# Check status
service pihole-FTL status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start pihole
brew services stop pihole
brew services restart pihole

# Check status
brew services list | grep pihole
```

### Windows Service Manager

```powershell
# Start service
net start pihole-FTL

# Stop service
net stop pihole-FTL

# Using PowerShell
Start-Service pihole-FTL
Stop-Service pihole-FTL
Restart-Service pihole-FTL

# Check status
Get-Service pihole-FTL
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream pihole_backend {
    server 127.0.0.1:53;
}

server {
    listen 80;
    server_name pihole.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pihole.example.com;

    ssl_certificate /etc/ssl/certs/pihole.example.com.crt;
    ssl_certificate_key /etc/ssl/private/pihole.example.com.key;

    location / {
        proxy_pass http://pihole_backend;
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
    ServerName pihole.example.com
    Redirect permanent / https://pihole.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName pihole.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/pihole.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/pihole.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:53/
    ProxyPassReverse / http://127.0.0.1:53/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend pihole_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/pihole.pem
    redirect scheme https if !{ ssl_fc }
    default_backend pihole_backend

backend pihole_backend
    balance roundrobin
    server pihole1 127.0.0.1:53 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R pihole:pihole /etc/pihole
sudo chmod 750 /etc/pihole

# Configure firewall
sudo firewall-cmd --permanent --add-port=53/tcp
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
sudo systemctl status pihole-FTL

# View logs
sudo journalctl -u pihole-FTL -f

# Monitor resource usage
top -p $(pgrep pihole)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/pihole"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/pihole-backup-$DATE.tar.gz" /etc/pihole /var/lib/pihole

echo "Backup completed: $BACKUP_DIR/pihole-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop pihole-FTL

# Restore from backup
tar -xzf /backup/pihole/pihole-backup-*.tar.gz -C /

# Start service
sudo systemctl start pihole-FTL
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u pihole-FTL -n 100
sudo tail -f /var/log/pihole/pihole.log

# Check configuration
pihole -v

# Check permissions
ls -la /etc/pihole
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 53

# Test connectivity
telnet localhost 53

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep pihole)

# Check disk I/O
iotop -p $(pgrep pihole)

# Check connections
ss -an | grep 53
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  pihole:
    image: pihole:latest
    ports:
      - "53:53"
    volumes:
      - ./config:/etc/pihole
      - ./data:/var/lib/pihole
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update pihole

# Debian/Ubuntu
sudo apt update && sudo apt upgrade pihole

# Arch Linux
sudo pacman -Syu pihole

# Alpine Linux
apk update && apk upgrade pihole

# openSUSE
sudo zypper update pihole

# FreeBSD
pkg update && pkg upgrade pihole

# Always backup before updates
tar -czf /backup/pihole-pre-update-$(date +%Y%m%d).tar.gz /etc/pihole

# Restart after updates
sudo systemctl restart pihole-FTL
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/pihole

# Clean old logs
find /var/log/pihole -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/pihole
```

## Additional Resources

- Official Documentation: https://docs.pihole.org/
- GitHub Repository: https://github.com/pihole/pihole
- Community Forum: https://forum.pihole.org/
- Best Practices Guide: https://docs.pihole.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
