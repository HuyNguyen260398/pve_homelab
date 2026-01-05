# Proxmox VE Homelab Installation Guide

This guide will walk you through installing and configuring a Proxmox VE homelab on your personal computer.

## Table of Contents
1. [Hardware Requirements](#hardware-requirements)
2. [Preparing Installation Media](#preparing-installation-media)
3. [Installing Proxmox VE](#installing-proxmox-ve)
4. [Initial Configuration](#initial-configuration)
5. [Network Setup](#network-setup)
6. [Post-Installation Steps](#post-installation-steps)
7. [Next Steps](#next-steps)

## Hardware Requirements

### Minimum Requirements
- **CPU**: 64-bit processor with virtualization support (Intel VT-x or AMD-V)
- **RAM**: 4 GB (8 GB or more recommended)
- **Storage**: 32 GB available disk space (SSD recommended)
- **Network**: Ethernet adapter (Gigabit recommended)

### Recommended Hardware
- **CPU**: Multi-core processor (4+ cores)
- **RAM**: 16 GB or more
- **Storage**: 
  - 128 GB SSD for OS and system
  - Additional storage for VMs and containers
- **Network**: Gigabit Ethernet

### Checking CPU Virtualization Support
Before installation, verify your CPU supports virtualization:

**On Linux:**
```bash
# Check for Intel VT-x
grep -E 'vmx' /proc/cpuinfo

# Check for AMD-V
grep -E 'svm' /proc/cpuinfo
```

**On Windows:**
- Open Task Manager → Performance → CPU
- Look for "Virtualization: Enabled"

## Preparing Installation Media

### Download Proxmox VE
1. Visit the official Proxmox website: https://www.proxmox.com/en/downloads
2. Download the latest Proxmox VE ISO installer
3. Verify the checksum (recommended)

### Create Bootable USB
**Using Rufus (Windows):**
1. Download Rufus from https://rufus.ie/
2. Insert USB drive (8 GB minimum)
3. Select the Proxmox VE ISO
4. Use DD mode when prompted
5. Click Start

**Using Etcher (Windows/macOS/Linux):**
1. Download balenaEtcher from https://www.balena.io/etcher/
2. Select the Proxmox VE ISO
3. Select your USB drive
4. Click Flash

**Using dd (Linux):**
```bash
# Find your USB device (be careful!)
lsblk

# Write ISO to USB (replace sdX with your device)
sudo dd if=proxmox-ve_*.iso of=/dev/sdX bs=1M status=progress
sudo sync
```

## Installing Proxmox VE

### Step 1: Boot from USB
1. Insert the bootable USB drive
2. Restart your computer
3. Enter BIOS/UEFI settings (usually F2, F12, DEL, or ESC)
4. Enable virtualization support:
   - Intel: Enable "Intel VT-x" or "Virtualization Technology"
   - AMD: Enable "AMD-V" or "SVM Mode"
5. Set USB drive as first boot device
6. Save and exit

### Step 2: Start Installation
1. Boot from USB
2. Select "Install Proxmox VE" from the menu
3. Accept the EULA

### Step 3: Target Disk Selection
1. Select the target hard disk for installation
2. Click "Options" if you want to customize:
   - Filesystem: ext4, xfs, or zfs
   - Disk setup: For ZFS, you can configure RAID
3. Click "Next"

### Step 4: Location and Time Zone
1. Select your country
2. Select your time zone
3. Select keyboard layout
4. Click "Next"

### Step 5: Administration Password and Email
1. Enter a strong root password
2. Confirm the password
3. Enter an email address for system notifications
4. Click "Next"

### Step 6: Network Configuration
1. Select management network interface
2. Enter hostname (e.g., `pve.local` or `proxmox.home`)
3. Enter IP address (static recommended):
   - Example: `192.168.1.100/24`
4. Enter Gateway (your router's IP):
   - Example: `192.168.1.1`
5. Enter DNS server:
   - Example: `8.8.8.8` or `192.168.1.1`
6. Click "Next"

### Step 7: Confirm and Install
1. Review all settings
2. Click "Install"
3. Wait for installation to complete (5-10 minutes)
4. Remove USB drive when prompted
5. Reboot

## Initial Configuration

### Accessing the Web Interface
1. After reboot, note the access URL shown on the console:
   - Example: `https://192.168.1.100:8006`
2. Open a web browser on another computer in the same network
3. Navigate to the URL (ignore SSL certificate warning)
4. Login with:
   - Username: `root`
   - Password: (password you set during installation)
   - Realm: `Linux PAM standard authentication`

### Update System
After first login, update your Proxmox VE installation:

```bash
# SSH into your Proxmox server or use the web console
ssh root@192.168.1.100

# Update package lists
apt update

# Upgrade packages
apt full-upgrade -y

# Reboot if kernel was updated
reboot
```

### Remove Enterprise Repository (Optional)
If you don't have a Proxmox subscription, disable the enterprise repository:

```bash
# Comment out enterprise repository
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Add no-subscription repository
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update package lists
apt update
```

## Network Setup

### Basic Network Configuration
Proxmox VE uses a Linux bridge for networking. The default configuration is usually sufficient for most homelabs.

View current network configuration:
```bash
cat /etc/network/interfaces
```

### Example Network Configuration
```
auto lo
iface lo inet loopback

iface enp0s3 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports enp0s3
        bridge-stp off
        bridge-fd 0
```

### NAT Configuration (Optional)
If you want VMs to access the internet through NAT:

```bash
# Enable IP forwarding
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

# Configure NAT (replace vmbr0 with your WAN interface)
cat >> /etc/network/interfaces << EOF

auto vmbr1
iface vmbr1 inet static
        address 10.10.10.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
EOF
```

## Post-Installation Steps

### 1. Configure Storage
Add additional storage locations:
1. Go to Datacenter → Storage
2. Click "Add" and select storage type:
   - **Directory**: Local directory storage
   - **LVM**: Logical Volume Manager
   - **ZFS**: ZFS pool
3. Configure and add storage

### 2. Upload ISO Images
1. Select your node → local (storage)
2. Click "ISO Images"
3. Click "Upload" or "Download from URL"
4. Upload operating system ISOs for your VMs

### 3. Create Your First VM
1. Click "Create VM" button
2. Follow the wizard:
   - General: Set VM ID and name
   - OS: Select ISO image
   - System: Default settings usually work
   - Disks: Set disk size
   - CPU: Allocate cores
   - Memory: Allocate RAM
   - Network: Select bridge (vmbr0)
3. Click "Finish"
4. Start the VM and access console

### 4. Create Your First Container (LXC)
1. Download container templates:
   - Select your node → local → CT Templates
   - Click "Templates" and download desired templates
2. Click "Create CT" button
3. Follow the wizard:
   - General: Set CT ID, hostname, password
   - Template: Select downloaded template
   - Root Disk: Set disk size
   - CPU: Allocate cores
   - Memory: Allocate RAM
   - Network: Configure network
4. Click "Finish"
5. Start the container

### 5. Configure Backups
1. Go to Datacenter → Backup
2. Click "Add" to create backup job
3. Configure:
   - Schedule: When to run backups
   - Selection: Which VMs/containers to backup
   - Storage: Where to store backups
   - Mode: Snapshot, Suspend, or Stop
4. Click "Create"

### 6. Setup Firewall (Optional)
1. Go to Datacenter → Firewall
2. Enable firewall
3. Configure rules as needed
4. Apply to VMs/containers

## Next Steps

### Learn More
- **Proxmox VE Documentation**: https://pve.proxmox.com/pve-docs/
- **Proxmox VE Forum**: https://forum.proxmox.com/
- **Community Wiki**: https://pve.proxmox.com/wiki/

### Recommended VMs/Services to Deploy
- **Pi-hole**: Network-wide ad blocking
- **Home Assistant**: Home automation platform
- **Nextcloud**: Private cloud storage
- **Plex/Jellyfin**: Media server
- **Docker host**: Container platform
- **pfSense/OPNsense**: Firewall/router
- **GitLab/Gitea**: Git repository hosting
- **Wiki.js/BookStack**: Documentation platform

### Security Best Practices
1. Change default SSH port
2. Setup firewall rules
3. Use strong passwords
4. Enable two-factor authentication
5. Regular backups
6. Keep system updated
7. Disable root SSH login (use sudo)
8. Setup monitoring and alerts

### Maintenance Tasks
- **Weekly**: Review logs and system health
- **Monthly**: Update packages and firmware
- **Quarterly**: Test backup restoration
- **As needed**: Clean up unused VMs/containers

## Troubleshooting

### Cannot access web interface
- Verify network connectivity: `ping 192.168.1.100`
- Check if web service is running: `systemctl status pveproxy`
- Verify firewall isn't blocking port 8006

### VM won't start
- Check CPU/RAM allocation
- Verify storage is available
- Review VM logs in web interface

### No internet in VMs
- Verify bridge configuration
- Check VM network settings
- Verify gateway and DNS settings

### Performance issues
- Monitor resource usage in web interface
- Consider adding more RAM or CPU cores
- Optimize VM settings
- Use VirtIO drivers for better performance

## Support
For issues and questions:
- Official Documentation: https://pve.proxmox.com/pve-docs/
- Community Forum: https://forum.proxmox.com/
- Reddit: r/Proxmox
