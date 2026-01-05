# Quick Start Guide

Get your Proxmox VE homelab up and running in under 30 minutes!

## Prerequisites

Before you begin, ensure you have:
- [ ] Compatible PC with virtualization support (see [Prerequisites](PREREQUISITES.md))
- [ ] 8 GB USB drive
- [ ] Downloaded Proxmox VE ISO
- [ ] Network information (IP, gateway, DNS)

## Installation Steps

### 1. Create Bootable USB (5 minutes)

**Windows:**
```
1. Download Rufus from https://rufus.ie/
2. Select Proxmox VE ISO
3. Click Start (use DD mode)
```

**Linux:**
```bash
sudo dd if=proxmox-ve_*.iso of=/dev/sdX bs=1M status=progress
```

### 2. Boot and Install (10 minutes)

1. **Boot from USB**
   - Insert USB and restart PC
   - Enter BIOS (F2/F12/DEL)
   - Enable virtualization
   - Boot from USB

2. **Install Proxmox VE**
   - Select "Install Proxmox VE"
   - Accept EULA
   - Select target disk
   - Choose timezone and keyboard
   - Set root password and email
   - Configure network:
     ```
     Hostname: pve.local
     IP:       192.168.1.100/24
     Gateway:  192.168.1.1
     DNS:      8.8.8.8
     ```
   - Click Install
   - Reboot when finished

### 3. Access Web Interface (2 minutes)

1. Open browser on another computer
2. Navigate to: `https://192.168.1.100:8006`
3. Accept security warning
4. Login:
   - Username: `root`
   - Password: (your password)
   - Realm: `Linux PAM standard authentication`

### 4. Initial Setup (5 minutes)

**Remove subscription notice:**
```bash
# SSH into Proxmox
ssh root@192.168.1.100

# Disable enterprise repo
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Add no-subscription repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update system
apt update && apt full-upgrade -y
```

### 5. Create Your First VM (10 minutes)

1. **Upload ISO**
   - Select: local (pve) → ISO Images
   - Click "Upload" or use URL
   - Upload OS ISO (e.g., Ubuntu)

2. **Create VM**
   - Click "Create VM"
   - General: Name your VM (e.g., "ubuntu-test")
   - OS: Select uploaded ISO
   - System: Keep defaults
   - Disks: 32 GB
   - CPU: 2 cores
   - Memory: 2048 MB
   - Network: vmbr0
   - Click "Finish"

3. **Start VM**
   - Select VM
   - Click "Start"
   - Click "Console"
   - Install OS as normal

## Next Steps

### Add More Services

**Container (LXC) - Faster alternative to VMs:**
```bash
# Download template
pveam update
pveam available
pveam download local ubuntu-22.04-standard_22.04-1_amd64.tar.zst

# Create container via web UI
```

### Configure Backups

1. Datacenter → Backup
2. Add backup job
3. Schedule: Daily at 2 AM
4. Select VMs/containers
5. Storage: local

### Common Services to Deploy

| Service | Type | Use Case |
|---------|------|----------|
| Pi-hole | LXC | Ad blocking |
| Home Assistant | VM | Home automation |
| Nextcloud | VM | Cloud storage |
| Plex/Jellyfin | VM | Media server |
| Docker | LXC/VM | Container platform |

## Common Commands

```bash
# List VMs
qm list

# Start VM
qm start 100

# Stop VM
qm stop 100

# List containers
pct list

# Start container
pct start 100

# Update Proxmox
apt update && apt full-upgrade
```

## Troubleshooting

### Can't access web interface
```bash
# Check service status
systemctl status pveproxy

# Restart if needed
systemctl restart pveproxy
```

### VM won't start
- Check resources (CPU/RAM not overcommitted)
- Verify storage is available
- Check VM logs in web interface

### No internet in VM
- Verify VM network is on vmbr0
- Check gateway and DNS settings in VM
- Test: `ping 8.8.8.8`

## Learn More

- **Full Installation Guide**: [INSTALLATION.md](INSTALLATION.md)
- **Network Configuration**: [NETWORK_EXAMPLES.md](NETWORK_EXAMPLES.md)
- **Troubleshooting**: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- **Official Docs**: https://pve.proxmox.com/pve-docs/

## Community Resources

- **Forum**: https://forum.proxmox.com/
- **Reddit**: r/Proxmox
- **YouTube**: Search "Proxmox Homelab"
- **Discord**: Various homelab communities

## Tips

- **Always** take snapshots before major changes
- **Setup** regular backups early
- **Use** containers (LXC) when possible - lighter than VMs
- **Monitor** resource usage to avoid overcommitment
- **Keep** system updated monthly
- **Document** your configuration

## Security Checklist

- [ ] Change default SSH port
- [ ] Setup firewall rules
- [ ] Use strong passwords
- [ ] Disable root SSH (use keys)
- [ ] Enable automatic updates
- [ ] Regular backup testing

---

**Questions?** Check the [full documentation](INSTALLATION.md) or visit the [Proxmox forum](https://forum.proxmox.com/).
