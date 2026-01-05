# Proxmox VE Command Reference

Quick reference guide for common Proxmox VE commands.

## VM Management (qm)

### Basic Operations
```bash
qm list                           # List all VMs
qm start <vmid>                   # Start VM
qm shutdown <vmid>                # Shutdown VM (ACPI)
qm stop <vmid>                    # Stop VM (force)
qm reboot <vmid>                  # Reboot VM
qm reset <vmid>                   # Reset VM
qm suspend <vmid>                 # Suspend VM
qm resume <vmid>                  # Resume VM
qm destroy <vmid>                 # Delete VM
```

### VM Information
```bash
qm status <vmid>                  # Show VM status
qm config <vmid>                  # Show VM configuration
qm pending <vmid>                 # Show pending changes
qm showcmd <vmid>                 # Show QEMU command
```

### VM Configuration
```bash
qm set <vmid> --memory 2048       # Set RAM
qm set <vmid> --cores 2           # Set CPU cores
qm set <vmid> --cpu host          # Set CPU type
qm set <vmid> --net0 virtio,bridge=vmbr0  # Set network
qm set <vmid> --agent enabled=1   # Enable QEMU guest agent
```

### Snapshots
```bash
qm snapshot <vmid> <snapname>     # Create snapshot
qm listsnapshot <vmid>            # List snapshots
qm rollback <vmid> <snapname>     # Rollback to snapshot
qm delsnapshot <vmid> <snapname>  # Delete snapshot
```

### Cloning
```bash
qm clone <vmid> <newid> --name <name> --full  # Full clone
qm clone <vmid> <newid> --name <name>          # Linked clone
qm template <vmid>                             # Convert to template
```

### Console Access
```bash
qm terminal <vmid>                # Open terminal
qm vncproxy <vmid>                # Get VNC proxy
qm monitor <vmid>                 # Open QEMU monitor
```

## Container Management (pct)

### Basic Operations
```bash
pct list                          # List all containers
pct start <ctid>                  # Start container
pct shutdown <ctid>               # Shutdown container
pct stop <ctid>                   # Stop container (force)
pct reboot <ctid>                 # Reboot container
pct destroy <ctid>                # Delete container
```

### Container Information
```bash
pct status <ctid>                 # Show container status
pct config <ctid>                 # Show configuration
pct df <ctid>                     # Show disk usage
pct fsck <ctid>                   # Check filesystem
```

### Container Configuration
```bash
pct set <ctid> --memory 1024      # Set RAM
pct set <ctid> --cores 2          # Set CPU cores
pct set <ctid> --net0 name=eth0,bridge=vmbr0,ip=dhcp
pct set <ctid> --features nesting=1,keyctl=1
pct resize <ctid> rootfs +8G      # Increase disk size
```

### Container Access
```bash
pct enter <ctid>                  # Enter container console
pct exec <ctid> -- <command>      # Execute command
pct exec <ctid> --user <user> -- <command>  # Execute as user
pct console <ctid>                # Attach to console
```

### Snapshots
```bash
pct snapshot <ctid> <snapname>    # Create snapshot
pct listsnapshot <ctid>           # List snapshots
pct rollback <ctid> <snapname>    # Rollback to snapshot
pct delsnapshot <ctid> <snapname> # Delete snapshot
```

### Cloning
```bash
pct clone <ctid> <newid> --hostname <name> --full  # Full clone
pct clone <ctid> <newid> --hostname <name>         # Linked clone
pct template <ctid>                                # Convert to template
```

## Backup and Restore (vzdump)

### Backup
```bash
vzdump <vmid>                     # Backup VM/container
vzdump <vmid> --storage local     # Specify storage
vzdump <vmid> --mode snapshot     # Snapshot mode
vzdump <vmid> --mode suspend      # Suspend mode
vzdump <vmid> --mode stop         # Stop mode
vzdump <vmid> --compress zstd     # Use zstd compression
vzdump --all 1                    # Backup all VMs/containers
```

### Restore
```bash
# Restore VM
qmrestore <backup-file> <vmid>

# Restore container
pct restore <ctid> <backup-file>
```

## Storage Management (pvesm)

### Storage Operations
```bash
pvesm status                      # List all storage
pvesm list <storage>              # List storage content
pvesm alloc <storage> <vmid> <filename> <size>  # Allocate disk
pvesm free <volid>                # Free disk
```

### Storage Scanning
```bash
pvesm scan nfs <server>           # Scan NFS server
pvesm scan iscsi <portal>         # Scan iSCSI portal
pvesm scan lvm                    # Scan LVM volume groups
```

## Template Management (pveam)

### Template Operations
```bash
pveam update                      # Update template list
pveam available                   # List available templates
pveam available --section system  # List system templates
pveam download <storage> <template>  # Download template
pveam list <storage>              # List downloaded templates
pveam remove <template-file>      # Remove template
```

## Network Management

### Network Configuration
```bash
ifreload -a                       # Reload network configuration
ifup <interface>                  # Bring interface up
ifdown <interface>                # Bring interface down
ip addr show                      # Show IP addresses
ip route show                     # Show routes
brctl show                        # Show bridges
```

### Network Testing
```bash
ping <host>                       # Test connectivity
traceroute <host>                 # Trace route
mtr <host>                        # Network diagnostics
netstat -tlnp                     # Show listening ports
ss -tlnp                          # Show listening ports (modern)
```

## System Management

### System Information
```bash
pveversion -v                     # Show Proxmox version
pvesh get /nodes/localhost/status # Show node status
pvesh get /cluster/resources      # Show cluster resources
cat /etc/pve/storage.cfg          # Show storage config
cat /etc/network/interfaces       # Show network config
```

### Service Management
```bash
systemctl status pveproxy         # Check web proxy status
systemctl status pvedaemon        # Check daemon status
systemctl status pvestatd         # Check statistics daemon
systemctl restart pveproxy        # Restart web proxy
systemctl restart networking      # Restart networking
```

### Updates
```bash
apt update                        # Update package list
apt full-upgrade                  # Upgrade packages
apt dist-upgrade                  # Distribution upgrade
pveam update                      # Update container templates
```

### Logs
```bash
journalctl -xe                    # View system logs
journalctl -u pveproxy            # View proxy logs
journalctl -u pve-container@<ctid>  # View container logs
tail -f /var/log/pve/tasks/active # View active tasks
dmesg                             # Kernel messages
```

## Cluster Management (pvecm)

### Cluster Operations
```bash
pvecm status                      # Show cluster status
pvecm nodes                       # List cluster nodes
pvecm add <hostname>              # Add node to cluster
pvecm delnode <nodename>          # Remove node from cluster
```

### Migration
```bash
qm migrate <vmid> <target-node>   # Migrate VM
qm migrate <vmid> <target-node> --online  # Online migration
pct migrate <ctid> <target-node>  # Migrate container
```

## Firewall Management

### Datacenter Firewall
```bash
cat /etc/pve/firewall/cluster.fw  # Show cluster firewall
```

### Node Firewall
```bash
cat /etc/pve/nodes/<node>/host.fw # Show node firewall
iptables -L -n                     # List iptables rules
iptables -t nat -L -n              # List NAT rules
```

## User Management

### User Operations
```bash
pveum user list                   # List users
pveum user add <user>@pam         # Add user
pveum passwd <user>@pam           # Set password
pveum user delete <user>@pam      # Delete user
```

### Role Management
```bash
pveum role list                   # List roles
pveum aclmod / -user <user>@pam -role Administrator  # Assign role
```

## Disk Management

### LVM Operations
```bash
lvs                               # List logical volumes
vgs                               # List volume groups
pvs                               # List physical volumes
lvcreate -L 10G -n <name> <vg>    # Create logical volume
lvextend -L +10G /dev/<vg>/<lv>   # Extend logical volume
lvremove /dev/<vg>/<lv>           # Remove logical volume
```

### ZFS Operations
```bash
zpool list                        # List ZFS pools
zpool status                      # Show pool status
zfs list                          # List ZFS datasets
zfs create <pool>/<dataset>       # Create dataset
zfs destroy <pool>/<dataset>      # Destroy dataset
```

### Disk Information
```bash
lsblk                             # List block devices
fdisk -l                          # List disks
smartctl -a /dev/sda              # Show disk SMART data
df -h                             # Show disk usage
du -sh /var/lib/vz/*              # Show directory sizes
```

## Performance Monitoring

### Resource Usage
```bash
top                               # Process monitor
htop                              # Enhanced process monitor
iostat -x 1                       # Disk I/O statistics
vmstat 1                          # Virtual memory stats
free -h                           # Memory usage
uptime                            # System uptime and load
```

### Process Management
```bash
ps aux                            # List all processes
ps aux --sort=-%cpu               # Sort by CPU usage
ps aux --sort=-%mem               # Sort by memory usage
kill <pid>                        # Kill process
killall <process-name>            # Kill by name
```

## Troubleshooting Commands

### Diagnostic Commands
```bash
pveversion -v                     # Version info
dmesg | tail                      # Kernel messages
journalctl -xe                    # System logs
systemctl --failed                # Failed services
ip addr show                      # Network interfaces
ip route show                     # Routing table
cat /proc/cpuinfo                 # CPU information
cat /proc/meminfo                 # Memory information
```

### Reset and Recovery
```bash
qm unlock <vmid>                  # Unlock VM
pct unlock <ctid>                 # Unlock container
systemctl restart pveproxy        # Restart web interface
systemctl restart pvedaemon       # Restart daemon
passwd root                       # Reset root password
```

## Useful One-Liners

### Backup All VMs
```bash
vzdump --all 1 --storage local --mode snapshot --compress zstd
```

### List All Running VMs/Containers
```bash
qm list | grep running
pct list | grep running
```

### Show Total Resource Usage
```bash
pvesh get /nodes/localhost/status --output-format json-pretty
```

### Stop All VMs
```bash
for vm in $(qm list | awk '{if(NR>1) print $1}'); do qm stop $vm; done
```

### Start All VMs
```bash
for vm in $(qm list | awk '{if(NR>1) print $1}'); do qm start $vm; done
```

### Check VM Network Settings
```bash
qm config <vmid> | grep net
```

### List VM Disk Usage
```bash
for vm in $(qm list | awk '{if(NR>1) print $1}'); do 
  echo "VM $vm:"
  qm config $vm | grep scsi
done
```

### Monitor Backup Progress
```bash
tail -f /var/log/pve/tasks/active
```

### Clean Old Backups (keep last 7)
```bash
cd /var/lib/vz/dump
ls -t vzdump-* | tail -n +8 | xargs rm
```

## Configuration Files

### Important Files
```
/etc/pve/                         # Proxmox config directory
/etc/pve/storage.cfg              # Storage configuration
/etc/pve/qemu-server/             # VM configurations
/etc/pve/lxc/                     # Container configurations
/etc/network/interfaces           # Network configuration
/etc/hosts                        # Hosts file
/var/lib/vz/                      # Storage root
/var/lib/vz/dump/                 # Backup storage
/var/lib/vz/template/cache/       # Container templates
```

## Tips

- Use `Tab` for command completion
- Use `man <command>` for detailed help
- Use `--help` flag for quick help
- Combine commands with `&&` for sequential execution
- Use `watch` command for live updates: `watch -n 1 qm list`
- Always backup before making changes
- Test commands on non-critical VMs first

## External Resources

- **Man Pages**: `man qm`, `man pct`, `man vzdump`
- **Official Docs**: https://pve.proxmox.com/pve-docs/
- **API Docs**: https://pve.proxmox.com/pve-docs/api-viewer/
- **Forum**: https://forum.proxmox.com/
- **Wiki**: https://pve.proxmox.com/wiki/
