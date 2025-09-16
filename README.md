# nebula Installation Guide

nebula is a free and open-source scalable overlay networking tool. Developed by Slack, Nebula provides secure networks between computers anywhere, serving as a modern alternative to traditional VPNs

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
  - RAM: 128MB minimum
  - Storage: 50MB for installation
  - Network: UDP connectivity
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4242 (default nebula port)
  - Lighthouse if used
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

# Install nebula
sudo dnf install -y nebula

# Enable and start service
sudo systemctl enable --now nebula

# Configure firewall
sudo firewall-cmd --permanent --add-port=4242/tcp
sudo firewall-cmd --reload

# Verify installation
nebula --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nebula
sudo apt install -y nebula

# Enable and start service
sudo systemctl enable --now nebula

# Configure firewall
sudo ufw allow 4242

# Verify installation
nebula --version
```

### Arch Linux

```bash
# Install nebula
sudo pacman -S nebula

# Enable and start service
sudo systemctl enable --now nebula

# Verify installation
nebula --version
```

### Alpine Linux

```bash
# Install nebula
apk add --no-cache nebula

# Enable and start service
rc-update add nebula default
rc-service nebula start

# Verify installation
nebula --version
```

### openSUSE/SLES

```bash
# Install nebula
sudo zypper install -y nebula

# Enable and start service
sudo systemctl enable --now nebula

# Configure firewall
sudo firewall-cmd --permanent --add-port=4242/tcp
sudo firewall-cmd --reload

# Verify installation
nebula --version
```

### macOS

```bash
# Using Homebrew
brew install nebula

# Start service
brew services start nebula

# Verify installation
nebula --version
```

### FreeBSD

```bash
# Using pkg
pkg install nebula

# Enable in rc.conf
echo 'nebula_enable="YES"' >> /etc/rc.conf

# Start service
service nebula start

# Verify installation
nebula --version
```

### Windows

```bash
# Using Chocolatey
choco install nebula

# Or using Scoop
scoop install nebula

# Verify installation
nebula --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nebula

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nebula --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nebula

# Start service
sudo systemctl start nebula

# Stop service
sudo systemctl stop nebula

# Restart service
sudo systemctl restart nebula

# Check status
sudo systemctl status nebula

# View logs
sudo journalctl -u nebula -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nebula default

# Start service
rc-service nebula start

# Stop service
rc-service nebula stop

# Restart service
rc-service nebula restart

# Check status
rc-service nebula status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nebula_enable="YES"' >> /etc/rc.conf

# Start service
service nebula start

# Stop service
service nebula stop

# Restart service
service nebula restart

# Check status
service nebula status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nebula
brew services stop nebula
brew services restart nebula

# Check status
brew services list | grep nebula
```

### Windows Service Manager

```powershell
# Start service
net start nebula

# Stop service
net stop nebula

# Using PowerShell
Start-Service nebula
Stop-Service nebula
Restart-Service nebula

# Check status
Get-Service nebula
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nebula_backend {
    server 127.0.0.1:4242;
}

server {
    listen 80;
    server_name nebula.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nebula.example.com;

    ssl_certificate /etc/ssl/certs/nebula.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nebula.example.com.key;

    location / {
        proxy_pass http://nebula_backend;
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
    ServerName nebula.example.com
    Redirect permanent / https://nebula.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nebula.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nebula.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nebula.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4242/
    ProxyPassReverse / http://127.0.0.1:4242/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nebula_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nebula.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nebula_backend

backend nebula_backend
    balance roundrobin
    server nebula1 127.0.0.1:4242 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nebula:nebula /etc/nebula
sudo chmod 750 /etc/nebula

# Configure firewall
sudo firewall-cmd --permanent --add-port=4242/tcp
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
sudo systemctl status nebula

# View logs
sudo journalctl -u nebula -f

# Monitor resource usage
top -p $(pgrep nebula)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nebula"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nebula-backup-$DATE.tar.gz" /etc/nebula /var/lib/nebula

echo "Backup completed: $BACKUP_DIR/nebula-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nebula

# Restore from backup
tar -xzf /backup/nebula/nebula-backup-*.tar.gz -C /

# Start service
sudo systemctl start nebula
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nebula -n 100
sudo tail -f /var/log/nebula/nebula.log

# Check configuration
nebula --version

# Check permissions
ls -la /etc/nebula
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4242

# Test connectivity
telnet localhost 4242

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nebula)

# Check disk I/O
iotop -p $(pgrep nebula)

# Check connections
ss -an | grep 4242
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nebula:
    image: nebula:latest
    ports:
      - "4242:4242"
    volumes:
      - ./config:/etc/nebula
      - ./data:/var/lib/nebula
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nebula

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nebula

# Arch Linux
sudo pacman -Syu nebula

# Alpine Linux
apk update && apk upgrade nebula

# openSUSE
sudo zypper update nebula

# FreeBSD
pkg update && pkg upgrade nebula

# Always backup before updates
tar -czf /backup/nebula-pre-update-$(date +%Y%m%d).tar.gz /etc/nebula

# Restart after updates
sudo systemctl restart nebula
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nebula

# Clean old logs
find /var/log/nebula -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nebula
```

## Additional Resources

- Official Documentation: https://docs.nebula.org/
- GitHub Repository: https://github.com/nebula/nebula
- Community Forum: https://forum.nebula.org/
- Best Practices Guide: https://docs.nebula.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
