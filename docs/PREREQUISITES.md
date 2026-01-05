# Prerequisites Checklist

Before installing Proxmox VE for your homelab, ensure you meet the following requirements:

## Hardware Checklist

- [ ] **Compatible CPU**
  - [ ] 64-bit x86 processor
  - [ ] Virtualization support (Intel VT-x or AMD-V)
  - [ ] Virtualization enabled in BIOS/UEFI
  - [ ] Multi-core recommended (4+ cores ideal)

- [ ] **Sufficient RAM**
  - [ ] Minimum: 4 GB
  - [ ] Recommended: 16 GB or more
  - [ ] Note: More RAM = more VMs/containers

- [ ] **Storage**
  - [ ] Minimum: 32 GB free disk space
  - [ ] Recommended: 128 GB+ SSD
  - [ ] Additional storage for VMs (optional but recommended)
  - [ ] Consider RAID configuration for data redundancy

- [ ] **Network Connectivity**
  - [ ] Ethernet network interface card (NIC)
  - [ ] Gigabit Ethernet recommended
  - [ ] Access to router for network configuration
  - [ ] Static IP address available (recommended)

## Software and Media

- [ ] **Installation Media**
  - [ ] Downloaded Proxmox VE ISO from official website
  - [ ] Verified ISO checksum (recommended)
  - [ ] USB drive (8 GB minimum) for bootable media
  - [ ] USB creation tool (Rufus, Etcher, or dd)

- [ ] **Operating System ISOs**
  - [ ] Downloaded OS ISOs for VMs you plan to create
  - [ ] Stored in accessible location for upload

## Network Information

Gather the following network information before installation:

- [ ] **IP Address Configuration**
  - [ ] Static IP address for Proxmox host: `_________________`
  - [ ] Subnet mask (e.g., 255.255.255.0 or /24): `_________________`
  - [ ] Gateway address (router IP): `_________________`
  - [ ] DNS server addresses: `_________________`

- [ ] **Hostname**
  - [ ] Chosen hostname (e.g., pve.local): `_________________`
  - [ ] Fully Qualified Domain Name (FQDN): `_________________`

## Access Requirements

- [ ] **Administration**
  - [ ] Strong root password planned
  - [ ] Email address for notifications
  - [ ] Keyboard layout known

- [ ] **Remote Access**
  - [ ] Another computer for web interface access
  - [ ] Web browser (Firefox, Chrome, or Edge)
  - [ ] Same network connectivity for management

## Knowledge Requirements

- [ ] **Basic Linux Knowledge**
  - [ ] Comfortable with command line
  - [ ] Understanding of basic networking concepts
  - [ ] Familiarity with SSH (helpful)

- [ ] **Virtualization Concepts**
  - [ ] Understanding of VMs vs. containers
  - [ ] Resource allocation concepts (CPU, RAM, disk)
  - [ ] Basic networking (bridge, NAT)

## Preparation Steps

- [ ] **Backup Current Data**
  - [ ] Backed up all important data from target PC
  - [ ] Confirmed backup integrity
  - [ ] Stored backup in safe location

- [ ] **BIOS/UEFI Access**
  - [ ] Know how to enter BIOS/UEFI (F2, F12, DEL, ESC)
  - [ ] Know how to change boot order
  - [ ] Located virtualization settings

- [ ] **Network Planning**
  - [ ] Decided on IP addressing scheme
  - [ ] Planned VM/container network layout
  - [ ] Identified needed network bridges

## Optional but Recommended

- [ ] **Additional Equipment**
  - [ ] KVM switch or extra monitor
  - [ ] Extra keyboard/mouse
  - [ ] UPS (Uninterruptible Power Supply)
  - [ ] Network switch for multiple NICs

- [ ] **Documentation**
  - [ ] Current network diagram
  - [ ] Password manager for credentials
  - [ ] Notebook for configuration notes

- [ ] **Testing Plan**
  - [ ] Plan for first VM/container to create
  - [ ] List of services to deploy
  - [ ] Backup and recovery strategy

## Compatibility Check

### Verify CPU Virtualization Support

**On Current Linux System:**
```bash
# Check for Intel VT-x
lscpu | grep -i virtualization
grep -E 'vmx' /proc/cpuinfo

# Check for AMD-V
grep -E 'svm' /proc/cpuinfo
```

**On Current Windows System:**
1. Open Task Manager (Ctrl+Shift+Esc)
2. Click "Performance" tab
3. Select "CPU"
4. Look for "Virtualization: Enabled"

**Alternative - Check BIOS/UEFI:**
1. Restart computer
2. Enter BIOS/UEFI
3. Look for settings:
   - Intel: "Intel VT-x", "Virtualization Technology", "Intel Virtualization"
   - AMD: "AMD-V", "SVM Mode", "Secure Virtual Machine"

### Test Network Connectivity
```bash
# Verify internet connectivity
ping -c 4 8.8.8.8

# Check DNS resolution
ping -c 4 www.proxmox.com

# Verify local network access
ping -c 4 [your-router-ip]
```

## Decision Points

Before proceeding, decide on:

- [ ] **Storage Configuration**
  - [ ] Single disk or RAID?
  - [ ] Filesystem: ext4, xfs, or ZFS?
  - [ ] Separate storage for VMs?

- [ ] **Network Architecture**
  - [ ] Single network interface or multiple?
  - [ ] Bridged networking or NAT?
  - [ ] VLANs needed?

- [ ] **Subscription Model**
  - [ ] Enterprise subscription (paid support)?
  - [ ] No-subscription (community support)?

- [ ] **Backup Strategy**
  - [ ] Local backups only?
  - [ ] Remote backup location?
  - [ ] Backup schedule?

## Ready to Install?

Once all items above are checked:
- [ ] All hardware requirements met
- [ ] Installation media prepared
- [ ] Network information gathered
- [ ] Current data backed up
- [ ] Installation plan documented

**If all boxes are checked, you're ready to proceed with installation!**

→ Continue to [INSTALLATION.md](INSTALLATION.md) for step-by-step installation guide.

## Additional Resources

- **Proxmox VE System Requirements**: https://www.proxmox.com/en/proxmox-ve/requirements
- **Proxmox VE Documentation**: https://pve.proxmox.com/pve-docs/
- **Community Forum**: https://forum.proxmox.com/
- **Hardware Compatibility**: https://forum.proxmox.com/forums/proxmox-ve-hardware-compatibility.25/
