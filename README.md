# linkerd Installation Guide

linkerd is a free and open-source service mesh. Linkerd provides ultralight, security-focused service mesh for Kubernetes

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
  - Storage: 5GB for config
  - Network: Various protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8086 (default linkerd port)
  - Admin on 9990
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

# Install linkerd
sudo dnf install -y linkerd

# Enable and start service
sudo systemctl enable --now linkerd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8086/tcp
sudo firewall-cmd --reload

# Verify installation
linkerd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install linkerd
sudo apt install -y linkerd

# Enable and start service
sudo systemctl enable --now linkerd

# Configure firewall
sudo ufw allow 8086

# Verify installation
linkerd --version
```

### Arch Linux

```bash
# Install linkerd
sudo pacman -S linkerd

# Enable and start service
sudo systemctl enable --now linkerd

# Verify installation
linkerd --version
```

### Alpine Linux

```bash
# Install linkerd
apk add --no-cache linkerd

# Enable and start service
rc-update add linkerd default
rc-service linkerd start

# Verify installation
linkerd --version
```

### openSUSE/SLES

```bash
# Install linkerd
sudo zypper install -y linkerd

# Enable and start service
sudo systemctl enable --now linkerd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8086/tcp
sudo firewall-cmd --reload

# Verify installation
linkerd --version
```

### macOS

```bash
# Using Homebrew
brew install linkerd

# Start service
brew services start linkerd

# Verify installation
linkerd --version
```

### FreeBSD

```bash
# Using pkg
pkg install linkerd

# Enable in rc.conf
echo 'linkerd_enable="YES"' >> /etc/rc.conf

# Start service
service linkerd start

# Verify installation
linkerd --version
```

### Windows

```bash
# Using Chocolatey
choco install linkerd

# Or using Scoop
scoop install linkerd

# Verify installation
linkerd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/linkerd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
linkerd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable linkerd

# Start service
sudo systemctl start linkerd

# Stop service
sudo systemctl stop linkerd

# Restart service
sudo systemctl restart linkerd

# Check status
sudo systemctl status linkerd

# View logs
sudo journalctl -u linkerd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add linkerd default

# Start service
rc-service linkerd start

# Stop service
rc-service linkerd stop

# Restart service
rc-service linkerd restart

# Check status
rc-service linkerd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'linkerd_enable="YES"' >> /etc/rc.conf

# Start service
service linkerd start

# Stop service
service linkerd stop

# Restart service
service linkerd restart

# Check status
service linkerd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start linkerd
brew services stop linkerd
brew services restart linkerd

# Check status
brew services list | grep linkerd
```

### Windows Service Manager

```powershell
# Start service
net start linkerd

# Stop service
net stop linkerd

# Using PowerShell
Start-Service linkerd
Stop-Service linkerd
Restart-Service linkerd

# Check status
Get-Service linkerd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream linkerd_backend {
    server 127.0.0.1:8086;
}

server {
    listen 80;
    server_name linkerd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name linkerd.example.com;

    ssl_certificate /etc/ssl/certs/linkerd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/linkerd.example.com.key;

    location / {
        proxy_pass http://linkerd_backend;
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
    ServerName linkerd.example.com
    Redirect permanent / https://linkerd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName linkerd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/linkerd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/linkerd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8086/
    ProxyPassReverse / http://127.0.0.1:8086/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend linkerd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/linkerd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend linkerd_backend

backend linkerd_backend
    balance roundrobin
    server linkerd1 127.0.0.1:8086 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R linkerd:linkerd /etc/linkerd
sudo chmod 750 /etc/linkerd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8086/tcp
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
sudo systemctl status linkerd

# View logs
sudo journalctl -u linkerd -f

# Monitor resource usage
top -p $(pgrep linkerd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/linkerd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/linkerd-backup-$DATE.tar.gz" /etc/linkerd /var/lib/linkerd

echo "Backup completed: $BACKUP_DIR/linkerd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop linkerd

# Restore from backup
tar -xzf /backup/linkerd/linkerd-backup-*.tar.gz -C /

# Start service
sudo systemctl start linkerd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u linkerd -n 100
sudo tail -f /var/log/linkerd/linkerd.log

# Check configuration
linkerd --version

# Check permissions
ls -la /etc/linkerd
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8086

# Test connectivity
telnet localhost 8086

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep linkerd)

# Check disk I/O
iotop -p $(pgrep linkerd)

# Check connections
ss -an | grep 8086
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  linkerd:
    image: linkerd:latest
    ports:
      - "8086:8086"
    volumes:
      - ./config:/etc/linkerd
      - ./data:/var/lib/linkerd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update linkerd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade linkerd

# Arch Linux
sudo pacman -Syu linkerd

# Alpine Linux
apk update && apk upgrade linkerd

# openSUSE
sudo zypper update linkerd

# FreeBSD
pkg update && pkg upgrade linkerd

# Always backup before updates
tar -czf /backup/linkerd-pre-update-$(date +%Y%m%d).tar.gz /etc/linkerd

# Restart after updates
sudo systemctl restart linkerd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/linkerd

# Clean old logs
find /var/log/linkerd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/linkerd
```

## Additional Resources

- Official Documentation: https://docs.linkerd.org/
- GitHub Repository: https://github.com/linkerd/linkerd
- Community Forum: https://forum.linkerd.org/
- Best Practices Guide: https://docs.linkerd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
