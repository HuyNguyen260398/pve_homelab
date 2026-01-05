# LXC Container Configuration Examples

This document provides examples for creating and configuring LXC containers in Proxmox VE.

## What are LXC Containers?

LXC (Linux Containers) are lightweight alternatives to full VMs:
- **Faster startup** - Boot in seconds
- **Less overhead** - Share host kernel
- **Better performance** - Near-native speed
- **Lower resource usage** - Use less RAM and disk

**When to use LXC:**
- Linux services (web servers, databases)
- Development environments
- Network services (DNS, DHCP)
- Monitoring tools

**When to use VMs instead:**
- Windows or non-Linux OS
- Kernel modules required
- Complete isolation needed
- Different kernel version needed

## Basic Ubuntu Container

### Specifications
- **Template**: Ubuntu 22.04
- **CPU**: 2 cores
- **RAM**: 1024 MB
- **Disk**: 8 GB
- **Network**: Bridge (vmbr0)

### Download Template
```bash
# Update template list
pveam update

# List available templates
pveam available

# Download Ubuntu 22.04 template
pveam download local ubuntu-22.04-standard_22.04-1_amd64.tar.zst
```

### Create Container
```bash
# Create unprivileged container
pct create 100 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname ubuntu-ct \
  --memory 1024 \
  --cores 2 \
  --rootfs local-lvm:8 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --password \
  --unprivileged 1

# Start container
pct start 100
```

### Via Web UI
1. Click "Create CT"
2. General:
   - CT ID: 100
   - Hostname: ubuntu-ct
   - Password: (set password)
   - Unprivileged: Yes
3. Template:
   - Storage: local
   - Template: ubuntu-22.04-standard
4. Root Disk:
   - Storage: local-lvm
   - Size: 8 GB
5. CPU:
   - Cores: 2
6. Memory:
   - Memory: 1024 MB
   - Swap: 512 MB
7. Network:
   - Bridge: vmbr0
   - IPv4: DHCP
8. DNS:
   - Use host settings
9. Confirm and Create

## Docker Host Container

### Specifications
- **Template**: Ubuntu 22.04
- **CPU**: 4 cores
- **RAM**: 4096 MB
- **Disk**: 20 GB
- **Network**: Bridge (vmbr0)
- **Features**: Nesting, keyctl

### Create Container
```bash
pct create 101 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname docker-host \
  --memory 4096 \
  --cores 4 \
  --rootfs local-lvm:20 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --password \
  --unprivileged 1 \
  --features nesting=1,keyctl=1

pct start 101
```

### Install Docker
```bash
# Enter container
pct enter 101

# Update system
apt update && apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Install Docker Compose
apt install docker-compose -y

# Test
docker run hello-world
```

### Important Notes
- `nesting=1` enables running containers inside container
- `keyctl=1` required for some Docker features
- Use privileged container if issues occur

## Pi-hole Container

### Specifications
- **Template**: Debian 12
- **CPU**: 1 core
- **RAM**: 512 MB
- **Disk**: 4 GB
- **Network**: Static IP

### Create Container
```bash
pct create 102 \
  local:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst \
  --hostname pihole \
  --memory 512 \
  --cores 1 \
  --rootfs local-lvm:4 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.53/24,gw=192.168.1.1 \
  --nameserver 8.8.8.8 \
  --password \
  --unprivileged 1

pct start 102
```

### Install Pi-hole
```bash
# Enter container
pct enter 102

# Update system
apt update && apt upgrade -y

# Install Pi-hole
curl -sSL https://install.pi-hole.net | bash

# Note the admin password shown at end of installation
```

### Configure Clients
- Set DNS server to 192.168.1.53
- Access web interface: http://192.168.1.53/admin

## Nginx Web Server Container

### Specifications
- **Template**: Ubuntu 22.04
- **CPU**: 2 cores
- **RAM**: 1024 MB
- **Disk**: 8 GB
- **Network**: Static IP

### Create Container
```bash
pct create 103 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname webserver \
  --memory 1024 \
  --cores 2 \
  --rootfs local-lvm:8 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.80/24,gw=192.168.1.1 \
  --nameserver 8.8.8.8 \
  --password \
  --unprivileged 1

pct start 103
```

### Install Nginx
```bash
# Enter container
pct enter 103

# Update and install
apt update
apt install nginx -y

# Start and enable
systemctl start nginx
systemctl enable nginx

# Test
curl localhost
```

## Database Server Container

### PostgreSQL

```bash
pct create 104 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname postgres \
  --memory 2048 \
  --cores 2 \
  --rootfs local-lvm:16 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.54/24,gw=192.168.1.1 \
  --nameserver 8.8.8.8 \
  --password \
  --unprivileged 1

pct start 104
pct enter 104

# Install PostgreSQL
apt update
apt install postgresql postgresql-contrib -y

# Configure
sudo -u postgres psql
# CREATE DATABASE mydb;
# CREATE USER myuser WITH PASSWORD 'mypass';
# GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```

### MySQL/MariaDB

```bash
pct create 105 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname mysql \
  --memory 2048 \
  --cores 2 \
  --rootfs local-lvm:16 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.55/24,gw=192.168.1.1 \
  --nameserver 8.8.8.8 \
  --password \
  --unprivileged 1

pct start 105
pct enter 105

# Install MariaDB
apt update
apt install mariadb-server -y

# Secure installation
mysql_secure_installation
```

## Privileged vs Unprivileged Containers

### Unprivileged (Recommended)
- More secure
- UID/GID mapping
- Limited access to host
- Cannot load kernel modules

```bash
# Create unprivileged
pct create 106 ... --unprivileged 1
```

### Privileged (Use only when necessary)
- Full root access
- Can load kernel modules
- Direct device access
- Security risk

```bash
# Create privileged
pct create 107 ... --unprivileged 0
```

## Container Features

### Enable Features
```bash
# Nesting (Docker/LXC inside container)
pct set 100 --features nesting=1

# Keyctl (systemd features)
pct set 100 --features keyctl=1

# FUSE (filesystem in userspace)
pct set 100 --features fuse=1

# NFS mounting
pct set 100 --features nfs=1

# CIFS mounting
pct set 100 --features cifs=1

# Multiple features
pct set 100 --features nesting=1,keyctl=1,fuse=1
```

## Mount Points

### Add Bind Mount
```bash
# Mount host directory into container
pct set 100 --mp0 /mnt/hostdata,mp=/mnt/data

# With size limit
pct set 100 --mp0 local-lvm:8,mp=/mnt/data

# Read-only mount
pct set 100 --mp0 /mnt/hostdata,mp=/mnt/data,ro=1
```

### Backup Bind Mounts
```bash
# Include in backups
pct set 100 --mp0 /mnt/hostdata,mp=/mnt/data,backup=1
```

## Resource Limits

### CPU Limits
```bash
# Set CPU cores
pct set 100 --cores 2

# Set CPU limit (percentage)
pct set 100 --cpulimit 0.5  # 50% of one core

# Set CPU units (weight)
pct set 100 --cpuunits 512
```

### Memory Limits
```bash
# Set memory
pct set 100 --memory 1024

# Set swap
pct set 100 --swap 512
```

### Disk I/O Limits
```bash
# Limit read/write speed (MB/s)
pct set 100 --rootfs local-lvm:8,mbps_rd=100,mbps_wr=50

# Limit IOPS
pct set 100 --rootfs local-lvm:8,iops_rd=1000,iops_wr=500
```

## Networking

### Static IP
```bash
pct set 100 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.100/24,gw=192.168.1.1
```

### DHCP
```bash
pct set 100 --net0 name=eth0,bridge=vmbr0,ip=dhcp
```

### Multiple Networks
```bash
# Add second network interface
pct set 100 --net1 name=eth1,bridge=vmbr1,ip=10.0.0.100/24
```

### VLAN Tagging
```bash
pct set 100 --net0 name=eth0,bridge=vmbr0,tag=10,ip=dhcp
```

## Storage

### Resize Disk
```bash
# Increase container disk size
pct resize 100 rootfs +8G
```

### Add Additional Disk
```bash
# Add mount point
pct set 100 --mp0 local-lvm:8,mp=/mnt/data
```

## Backup and Restore

### Manual Backup
```bash
# Backup container
vzdump 100 --storage local --mode snapshot --compress zstd

# Backup with specific name
vzdump 100 --storage local --dumpdir /var/lib/vz/dump --compress zstd
```

### Restore Container
```bash
# List backups
pct restore

# Restore from backup
pct restore 100 /var/lib/vz/dump/vzdump-lxc-100-*.tar.zst

# Restore to new ID
pct restore 200 /var/lib/vz/dump/vzdump-lxc-100-*.tar.zst
```

## Snapshots

```bash
# Create snapshot
pct snapshot 100 before_update

# List snapshots
pct listsnapshot 100

# Rollback to snapshot
pct rollback 100 before_update

# Delete snapshot
pct delsnapshot 100 before_update
```

## Container Management

### Basic Commands
```bash
# List containers
pct list

# Start container
pct start 100

# Shutdown container
pct shutdown 100

# Stop container (force)
pct stop 100

# Restart container
pct reboot 100

# Delete container
pct destroy 100
```

### Access Container
```bash
# Enter container console
pct enter 100

# Execute command in container
pct exec 100 -- ls -la /

# Run command as specific user
pct exec 100 --user www-data -- whoami
```

### Container Status
```bash
# Show status
pct status 100

# Show configuration
pct config 100

# Show resource usage
pct df 100
```

## Templates

### Create Template
```bash
# Convert container to template
pct template 100
```

### Clone from Template
```bash
# Full clone
pct clone 100 200 --hostname new-container --full

# Linked clone
pct clone 100 201 --hostname linked-container
```

## Common Use Cases

### Development Environment
```bash
pct create 110 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname dev-env \
  --memory 4096 \
  --cores 4 \
  --rootfs local-lvm:32 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1,keyctl=1 \
  --unprivileged 1

# Install development tools
pct enter 110
apt update && apt upgrade -y
apt install build-essential git curl wget vim -y
```

### Media Server (Jellyfin)
```bash
pct create 111 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname jellyfin \
  --memory 2048 \
  --cores 2 \
  --rootfs local-lvm:16 \
  --mp0 /mnt/media,mp=/media \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features fuse=1 \
  --unprivileged 1
```

### Monitoring (Prometheus + Grafana)
```bash
pct create 112 \
  local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname monitoring \
  --memory 2048 \
  --cores 2 \
  --rootfs local-lvm:16 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.60/24,gw=192.168.1.1 \
  --unprivileged 1
```

## Troubleshooting

### Container Won't Start
```bash
# Check status
pct status 100

# Check logs
journalctl -u pve-container@100

# Check configuration
pct config 100

# Try manual start
pct start 100
```

### Network Not Working
```bash
# Inside container
ip addr show
ping 8.8.8.8
ping google.com

# Check from host
pct exec 100 -- ip addr show
```

### Storage Issues
```bash
# Check disk usage
pct df 100

# Check filesystem
pct fsck 100

# Resize if needed
pct resize 100 rootfs +4G
```

## Best Practices

1. **Use unprivileged containers** when possible for security
2. **Enable features only when needed** (nesting, keyctl, etc.)
3. **Use templates** for faster deployment
4. **Configure backups** before making changes
5. **Use static IPs** for services
6. **Monitor resource usage** to avoid overcommitment
7. **Keep containers updated** regularly
8. **Separate services** into different containers
9. **Use bind mounts** for shared data
10. **Document configurations** for easy recovery

## Performance Tips

1. Use LXC instead of VMs when possible
2. Limit CPU and memory appropriately
3. Use local storage for better I/O
4. Enable disk cache when appropriate
5. Monitor resource usage regularly
6. Avoid running too many services in one container
7. Use swap cautiously

## Security

1. Use unprivileged containers by default
2. Don't expose unnecessary services
3. Keep containers updated
4. Use firewall rules
5. Limit resource usage
6. Use strong passwords
7. Regular backups
8. Monitor logs for suspicious activity
