# VM Configuration Examples

This directory contains example configurations for common virtual machines in your Proxmox homelab.

## Ubuntu Server VM

### Specifications
- **OS**: Ubuntu Server 22.04 LTS
- **CPU**: 2 cores
- **RAM**: 2048 MB
- **Disk**: 32 GB
- **Network**: Bridge (vmbr0)

### Creation via CLI
```bash
# Create VM
qm create 100 \
  --name ubuntu-server \
  --memory 2048 \
  --cores 2 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-pci

# Add disk
qm set 100 --scsi0 local-lvm:32

# Set boot order
qm set 100 --boot order=scsi0

# Attach ISO
qm set 100 --ide2 local:iso/ubuntu-22.04-server-amd64.iso,media=cdrom

# Start VM
qm start 100
```

### Creation via Web UI
1. Click "Create VM"
2. General:
   - VM ID: 100
   - Name: ubuntu-server
3. OS:
   - ISO image: ubuntu-22.04-server-amd64.iso
4. System:
   - SCSI Controller: VirtIO SCSI
5. Disks:
   - Disk size: 32 GB
   - Bus/Device: SCSI
6. CPU:
   - Cores: 2
7. Memory:
   - Memory: 2048 MB
8. Network:
   - Bridge: vmbr0
   - Model: VirtIO
9. Confirm and Finish

## Docker Host VM

### Specifications
- **OS**: Ubuntu Server 22.04 LTS
- **CPU**: 4 cores
- **RAM**: 4096 MB
- **Disk**: 50 GB
- **Network**: Bridge (vmbr0)

### Creation
```bash
qm create 101 \
  --name docker-host \
  --memory 4096 \
  --cores 4 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-pci

qm set 101 --scsi0 local-lvm:50
qm set 101 --boot order=scsi0
qm set 101 --ide2 local:iso/ubuntu-22.04-server-amd64.iso,media=cdrom
```

### Post-Install: Docker Installation
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose -y

# Enable Docker service
sudo systemctl enable docker
sudo systemctl start docker
```

## Windows 10 VM

### Specifications
- **OS**: Windows 10 Pro
- **CPU**: 4 cores
- **RAM**: 8192 MB
- **Disk**: 80 GB
- **Network**: Bridge (vmbr0)

### Creation
```bash
qm create 102 \
  --name windows10 \
  --memory 8192 \
  --cores 4 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-pci \
  --ostype win10

qm set 102 --scsi0 local-lvm:80
qm set 102 --boot order=scsi0
qm set 102 --ide2 local:iso/windows-10.iso,media=cdrom

# Add VirtIO drivers ISO
qm set 102 --ide0 local:iso/virtio-win.iso,media=cdrom

# Enable UEFI
qm set 102 --bios ovmf
qm set 102 --efidisk0 local-lvm:1
```

### Notes
- Download VirtIO drivers from: https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/
- During Windows installation, load VirtIO SCSI driver from second CD
- After installation, install remaining VirtIO drivers

## pfSense Firewall VM

### Specifications
- **OS**: pfSense CE
- **CPU**: 2 cores
- **RAM**: 2048 MB
- **Disk**: 16 GB
- **Network**: 
  - WAN: vmbr0
  - LAN: vmbr1

### Creation
```bash
qm create 103 \
  --name pfsense \
  --memory 2048 \
  --cores 2 \
  --net0 virtio,bridge=vmbr0 \
  --net1 virtio,bridge=vmbr1 \
  --scsihw virtio-scsi-pci

qm set 103 --scsi0 local-lvm:16
qm set 103 --boot order=scsi0
qm set 103 --ide2 local:iso/pfSense-CE.iso,media=cdrom
```

### Network Setup
- net0 (vmbr0): WAN interface
- net1 (vmbr1): LAN interface

## Home Assistant VM

### Specifications
- **OS**: Home Assistant OS
- **CPU**: 2 cores
- **RAM**: 4096 MB
- **Disk**: 32 GB
- **Network**: Bridge (vmbr0)

### Creation
```bash
qm create 104 \
  --name homeassistant \
  --memory 4096 \
  --cores 2 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-pci

qm set 104 --scsi0 local-lvm:32
qm set 104 --boot order=scsi0

# Use Home Assistant OS image (convert qcow2 to raw first)
qm importdisk 104 haos.qcow2 local-lvm
```

## Backup Configuration

### Manual Backup
```bash
# Backup specific VM
vzdump 100 --storage local --mode snapshot --compress zstd

# Backup all VMs (1 = VMs only, use --all for both VMs and containers)
vzdump --all 1 --storage local --mode snapshot
```

### Scheduled Backup (via Web UI)
1. Datacenter → Backup
2. Add backup job:
   - Schedule: Daily 02:00
   - Selection: All
   - Storage: local
   - Mode: Snapshot
   - Compression: ZSTD
   - Retention: Keep last 7

## Resource Limits

### CPU Limits
```bash
# Set CPU limit (percentage)
qm set 100 --cpulimit 0.5  # 50% of one core

# Set CPU units (relative weight)
qm set 100 --cpuunits 512  # Default is 1024
```

### Memory Limits
```bash
# Set minimum memory (balloon)
qm set 100 --balloon 512

# Set memory
qm set 100 --memory 2048
```

### Disk I/O Limits
```bash
# Set maximum read/write speed (MB/s)
qm set 100 --scsi0 local-lvm:32,mbps_rd=100,mbps_wr=50

# Set maximum IOPS
qm set 100 --scsi0 local-lvm:32,iops_rd=1000,iops_wr=500
```

## Snapshots

### Create Snapshot
```bash
# Create snapshot
qm snapshot 100 before_update

# Create snapshot with description
qm snapshot 100 before_kernel_update --description "Before kernel 5.15 update"
```

### List Snapshots
```bash
qm listsnapshot 100
```

### Rollback to Snapshot
```bash
qm rollback 100 before_update
```

### Delete Snapshot
```bash
qm delsnapshot 100 before_update
```

## Clone VM

### Full Clone
```bash
# Clone VM 100 to new VM 200
qm clone 100 200 --name ubuntu-server-clone --full
```

### Linked Clone
```bash
# Linked clone (faster, uses less space)
qm clone 100 201 --name ubuntu-server-linked
```

## VM Migration

### Migrate to Another Storage
```bash
# Move disk to different storage
qm move_disk 100 scsi0 local-zfs
```

### Migrate Between Nodes (Cluster)
```bash
# Online migration
qm migrate 100 node2

# Offline migration
qm migrate 100 node2 --online 0
```

## Useful Commands

### VM Management
```bash
# List all VMs
qm list

# Start VM
qm start 100

# Shutdown VM (ACPI)
qm shutdown 100

# Stop VM (force)
qm stop 100

# Reset VM
qm reset 100

# Suspend VM
qm suspend 100

# Resume VM
qm resume 100

# Delete VM
qm destroy 100
```

### VM Information
```bash
# Show VM config
qm config 100

# Show VM status
qm status 100

# Monitor VM
qm monitor 100
```

### Console Access
```bash
# Open console
qm terminal 100

# Open VNC
qm vncproxy 100
```

## Templates

### Create Template from VM
```bash
# Convert VM to template
qm template 100
```

### Use Template
```bash
# Clone from template
qm clone 100 300 --name new-vm-from-template --full

# Linked clone from template (faster)
qm clone 100 301 --name linked-vm-from-template
```

## Cloud-Init Configuration

### Enable Cloud-Init
```bash
# Add Cloud-Init drive
qm set 100 --ide2 local-lvm:cloudinit

# Set Cloud-Init parameters
qm set 100 --ciuser ubuntu
qm set 100 --cipassword mypassword
qm set 100 --sshkeys ~/.ssh/id_rsa.pub
qm set 100 --ipconfig0 ip=dhcp
```

### Static IP with Cloud-Init
```bash
qm set 100 --ipconfig0 ip=192.168.1.100/24,gw=192.168.1.1
qm set 100 --nameserver 8.8.8.8
```

## Best Practices

1. **Use VirtIO drivers** for better performance
2. **Enable backups** before major changes
3. **Use snapshots** before updates
4. **Monitor resources** to avoid overcommitment
5. **Use templates** for faster deployment
6. **Document configurations** for easy recovery
7. **Test backups** regularly
8. **Keep VM tools updated** (qemu-guest-agent)

## Performance Tuning

### Enable QEMU Guest Agent
```bash
# In VM config
qm set 100 --agent enabled=1

# Install in Ubuntu VM
sudo apt install qemu-guest-agent
sudo systemctl enable qemu-guest-agent
sudo systemctl start qemu-guest-agent
```

### CPU Type
```bash
# Set CPU type to host for better performance
qm set 100 --cpu host
```

### Disk Cache
```bash
# Set disk cache mode
qm set 100 --scsi0 local-lvm:32,cache=writeback
```

### Memory Ballooning
```bash
# Enable memory ballooning
qm set 100 --balloon 1024
```
