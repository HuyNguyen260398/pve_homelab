# Proxmox VE Homelab Troubleshooting Guide

This guide covers common issues and their solutions when running Proxmox VE in a homelab environment.

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [Network Problems](#network-problems)
3. [Web Interface Issues](#web-interface-issues)
4. [VM/Container Issues](#vmcontainer-issues)
5. [Storage Problems](#storage-problems)
6. [Performance Issues](#performance-issues)
7. [Update/Upgrade Issues](#updateupgrade-issues)
8. [Backup and Restore](#backup-and-restore)

## Installation Issues

### USB Drive Won't Boot

**Symptoms:**
- Computer doesn't boot from USB
- Stuck on manufacturer logo
- Boot device not found

**Solutions:**
1. Verify boot order in BIOS/UEFI
2. Try different USB port (USB 2.0 ports often more reliable)
3. Recreate bootable USB with different tool
4. Ensure UEFI/Legacy boot mode matches your system
5. Disable Secure Boot in BIOS if enabled

### Installation Freezes or Crashes

**Symptoms:**
- Installation hangs at certain percentage
- Black screen during installation
- System reboots unexpectedly

**Solutions:**
1. Check RAM with memtest86+
2. Try basic graphics mode during boot
3. Disconnect unnecessary peripherals
4. Update BIOS/UEFI firmware
5. Check hardware compatibility

### "No Valid Subscription" Warning

**Symptoms:**
- Warning message about subscription after login
- Repository errors during updates

**Solutions:**
```bash
# Disable enterprise repository
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Add no-subscription repository
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update package lists
apt update
```

## Network Problems

### Cannot Access Web Interface

**Symptoms:**
- Unable to reach https://[IP]:8006
- Connection timeout
- Connection refused

**Diagnostic Steps:**
```bash
# Check if Proxmox web service is running
systemctl status pveproxy
systemctl status pvedaemon

# Check if port 8006 is listening
netstat -tlnp | grep 8006
# or
ss -tlnp | grep 8006

# Restart web services if needed
systemctl restart pveproxy
systemctl restart pvedaemon
```

**Solutions:**
1. Verify correct IP address:
   ```bash
   ip addr show
   ```
2. Check firewall rules:
   ```bash
   iptables -L -n
   ```
3. Verify network connectivity from client machine:
   ```bash
   ping [proxmox-ip]
   telnet [proxmox-ip] 8006
   ```
4. Try accessing from different device
5. Check browser (try different browser, clear cache)

### VMs Cannot Access Internet

**Symptoms:**
- VM can ping gateway but not internet
- DNS resolution fails
- No network connectivity

**Diagnostic Steps:**
```bash
# Check bridge configuration
ip addr show vmbr0
brctl show

# Check if IP forwarding is enabled (if using NAT)
cat /proc/sys/net/ipv4/ip_forward

# Check iptables rules
iptables -t nat -L -n -v
```

**Solutions:**
1. Verify bridge configuration:
   ```bash
   cat /etc/network/interfaces
   ```
2. Ensure VM is on correct bridge (vmbr0)
3. Check VM network settings (IP, gateway, DNS)
4. Verify DNS servers are correct
5. Test with static IP before trying DHCP

### Cannot SSH into Proxmox Host

**Symptoms:**
- SSH connection refused
- Connection timeout
- Authentication failure

**Solutions:**
```bash
# Check if SSH service is running
systemctl status ssh
systemctl start ssh
systemctl enable ssh

# Check SSH configuration
cat /etc/ssh/sshd_config

# Verify firewall isn't blocking SSH
iptables -L -n | grep 22

# Check listening ports
ss -tlnp | grep :22
```

## Web Interface Issues

### SSL Certificate Warnings

**Symptoms:**
- Browser shows security warning
- "Your connection is not private"

**Solutions:**
1. This is normal for self-signed certificates - proceed anyway
2. Generate proper SSL certificate:
   ```bash
   # Install certbot
   apt install certbot
   
   # Get Let's Encrypt certificate (requires public domain)
   certbot certonly --standalone -d your-domain.com
   ```
3. Import certificate in browser as trusted

### Web Interface Very Slow

**Symptoms:**
- Pages take long time to load
- Console unresponsive
- Timeouts

**Diagnostic Steps:**
```bash
# Check system load
uptime
top

# Check memory usage
free -h

# Check disk I/O
iostat -x 1 5

# Check Proxmox services
systemctl status pve*
```

**Solutions:**
1. Restart Proxmox services:
   ```bash
   systemctl restart pveproxy pvedaemon pvestatd
   ```
2. Clear browser cache
3. Check network latency
4. Verify system resources not exhausted

### Cannot Login to Web Interface

**Symptoms:**
- Incorrect username/password error
- Authentication failed
- Login page loops

**Solutions:**
```bash
# Reset root password from console
passwd root

# Check PAM authentication
cat /etc/pam.d/common-auth

# Verify PVE authentication service
systemctl status pvedaemon

# Check logs for authentication errors
journalctl -u pvedaemon | tail -50
```

## VM/Container Issues

### VM Won't Start

**Symptoms:**
- Start button doesn't work
- VM shows error message
- VM starts then stops immediately

**Diagnostic Steps:**
```bash
# Check VM configuration
qm config [VMID]

# Check VM logs
tail -50 /var/log/pve/tasks/active

# Try starting from command line
qm start [VMID]

# Check storage availability
pvesm status
```

**Solutions:**
1. Verify sufficient resources (CPU, RAM)
2. Check storage is accessible
3. Verify disk images exist:
   ```bash
   ls -lh /var/lib/vz/images/[VMID]/
   ```
4. Check for lock files:
   ```bash
   qm unlock [VMID]
   ```
5. Review VM configuration for errors

### Container Won't Start

**Symptoms:**
- LXC container fails to start
- Error messages about mount points

**Diagnostic Steps:**
```bash
# Check container configuration
pct config [CTID]

# Try starting from command line
pct start [CTID]

# Check container logs
journalctl -u pve-container@[CTID]
```

**Solutions:**
1. Verify container template exists
2. Check mount points configuration
3. Verify sufficient resources
4. Check for corrupt container:
   ```bash
   pct fsck [CTID]
   ```

### VNC/NoVNC Console Not Working

**Symptoms:**
- Black screen in console
- "Failed to connect to server" error
- Console doesn't respond

**Solutions:**
1. Restart VM/container
2. Check VM display settings (ensure VGA is enabled)
3. Restart web proxy:
   ```bash
   systemctl restart pveproxy
   ```
4. Try different browser
5. Check websocket connections aren't blocked

## Storage Problems

### Storage Full

**Symptoms:**
- "No space left on device"
- Cannot create new VMs
- Cannot take snapshots

**Diagnostic Steps:**
```bash
# Check disk usage
df -h

# Check storage status in Proxmox
pvesm status

# Find large files
du -sh /var/lib/vz/* | sort -hr | head -10
```

**Solutions:**
1. Remove old backups:
   ```bash
   ls -lh /var/lib/vz/dump/
   rm /var/lib/vz/dump/old-backup.vma.gz
   ```
2. Delete unused VM disks
3. Remove old snapshots
4. Clean up old container templates:
   ```bash
   ls /var/lib/vz/template/cache/
   ```
5. Extend storage or add new storage

### Storage Not Visible

**Symptoms:**
- Storage shows offline
- Cannot access storage
- Storage disappeared

**Solutions:**
```bash
# Check storage configuration
cat /etc/pve/storage.cfg

# Verify mount points
mount | grep pve

# Check storage service
systemctl status pvestatd

# Rescan storage
pvesm scan nfs [server]
pvesm scan lvm
```

## Performance Issues

### High CPU Usage

**Diagnostic Steps:**
```bash
# Check CPU usage
top
htop

# Identify high CPU processes
ps aux --sort=-%cpu | head -10

# Check VM CPU usage
qm list
```

**Solutions:**
1. Review VM CPU allocation (reduce if over-committed)
2. Use CPU limits on VMs if needed
3. Enable CPU units for fairness
4. Check for runaway processes in VMs
5. Consider upgrading hardware

### High Memory Usage

**Diagnostic Steps:**
```bash
# Check memory usage
free -h
vmstat 1 10

# Check for memory pressure
dmesg | grep -i "out of memory"
```

**Solutions:**
1. Review VM memory allocation
2. Enable memory ballooning
3. Use swap (if available)
4. Reduce VM memory allocations
5. Add more physical RAM

### Slow Disk Performance

**Diagnostic Steps:**
```bash
# Check disk I/O
iostat -x 1 5

# Check for high I/O wait
top
# Look at %wa column

# Test disk speed
hdparm -tT /dev/sda
```

**Solutions:**
1. Use VirtIO drivers in VMs
2. Enable disk cache (writethrough or writeback)
3. Consider SSD for VM storage
4. Check for failing disks:
   ```bash
   smartctl -a /dev/sda
   ```
5. Reduce simultaneous VM operations

## Update/Upgrade Issues

### Repository Errors During Update

**Symptoms:**
- "Unable to fetch" errors
- 401 Unauthorized errors
- Repository not found

**Solutions:**
```bash
# Fix enterprise repository issue
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Ensure no-subscription repo is added
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update package cache
apt update
```

### Update Breaks System

**Symptoms:**
- System won't boot after update
- Services don't start
- Errors after update

**Solutions:**
1. Boot into previous kernel (GRUB menu)
2. Check logs:
   ```bash
   journalctl -xb
   ```
3. Try to repair packages:
   ```bash
   apt --fix-broken install
   dpkg --configure -a
   ```
4. Restore from backup if available

## Backup and Restore

### Backup Fails

**Symptoms:**
- Backup job shows error
- Incomplete backups
- Storage full during backup

**Diagnostic Steps:**
```bash
# Check backup logs
tail -100 /var/log/pve/tasks/active

# Verify backup storage
pvesm status

# Check available space
df -h
```

**Solutions:**
1. Ensure sufficient storage space
2. Try different backup mode (snapshot vs. stop)
3. Check VM/container isn't corrupted
4. Verify backup storage is accessible

### Restore Fails

**Symptoms:**
- Cannot restore from backup
- Restore shows errors
- Restored VM won't start

**Solutions:**
1. Verify backup file integrity
2. Check storage has sufficient space
3. Try restoring to different storage
4. Manually extract backup:
   ```bash
   vzdump-extract-config backup.vma.gz
   ```

## Getting Help

If you cannot resolve your issue:

### Collect System Information
```bash
# Generate support report
pveversion -v
pveversion -v > /tmp/pve-version.txt

# Check system logs
journalctl -xe > /tmp/system-log.txt
dmesg > /tmp/dmesg.txt

# Check Proxmox specific logs
cat /var/log/pve/tasks/active
```

### Resources
- **Official Documentation**: https://pve.proxmox.com/pve-docs/
- **Community Forum**: https://forum.proxmox.com/
- **Reddit**: r/Proxmox
- **Wiki**: https://pve.proxmox.com/wiki/

### Before Posting for Help
1. Describe the problem clearly
2. List what you've tried
3. Include relevant error messages
4. Provide system information (pveversion -v)
5. Include relevant configuration (sanitize passwords!)
