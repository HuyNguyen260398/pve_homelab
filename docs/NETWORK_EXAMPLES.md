# Network Configuration Examples

This document provides common network configuration examples for Proxmox VE homelabs.

## Table of Contents
1. [Basic Single Network](#basic-single-network)
2. [Bridge with NAT](#bridge-with-nat)
3. [VLAN Configuration](#vlan-configuration)
4. [Multiple Network Interfaces](#multiple-network-interfaces)
5. [Bond Configuration](#bond-configuration)

## Basic Single Network

The simplest configuration with one network interface bridged for VM access.

### Configuration
**File**: `/etc/network/interfaces`

```bash
auto lo
iface lo inet loopback

# Physical interface
iface enp0s3 inet manual

# Bridge for VMs
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports enp0s3
        bridge-stp off
        bridge-fd 0
        # DNS servers
        dns-nameservers 8.8.8.8 8.8.4.4
```

### Use Case
- Single home network
- All VMs on same network as host
- Direct internet access for VMs

### VM Configuration
- Network Device: Bridge (vmbr0)
- VLAN Tag: None
- IP: DHCP or static in same subnet (192.168.1.x)

## Bridge with NAT

Create an isolated internal network with NAT for internet access.

### Configuration
**File**: `/etc/network/interfaces`

```bash
auto lo
iface lo inet loopback

# Physical interface connected to internet
iface enp0s3 inet manual

# Main bridge (management network)
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports enp0s3
        bridge-stp off
        bridge-fd 0

# Internal NAT network for VMs
auto vmbr1
iface vmbr1 inet static
        address 10.10.10.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
```

### Persistent IP Forwarding
**File**: `/etc/sysctl.conf`

```bash
net.ipv4.ip_forward=1
```

Apply changes:
```bash
sysctl -p
```

### Use Case
- Isolated lab environment
- VMs hidden behind NAT
- Controlled internet access
- Testing network configurations

### VM Configuration on NAT Network
- Network Device: Bridge (vmbr1)
- IP: Static in 10.10.10.0/24 range
- Gateway: 10.10.10.1
- DNS: 8.8.8.8 or 10.10.10.1 (if DNS forwarding configured)

## VLAN Configuration

Segment network into multiple VLANs for better organization and security.

### Configuration
**File**: `/etc/network/interfaces`

```bash
auto lo
iface lo inet loopback

# Physical interface
iface enp0s3 inet manual

# Main bridge (untagged)
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports enp0s3
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094

# VLAN 10 - Management
auto vmbr0.10
iface vmbr0.10 inet static
        address 10.0.10.1/24

# VLAN 20 - Servers
auto vmbr0.20
iface vmbr0.20 inet static
        address 10.0.20.1/24

# VLAN 30 - DMZ
auto vmbr0.30
iface vmbr0.30 inet static
        address 10.0.30.1/24
```

### Use Case
- Network segmentation
- Separate management/production/DMZ
- Multiple subnets on single physical interface
- Enhanced security

### VM Configuration
- Network Device: Bridge (vmbr0)
- VLAN Tag: 10, 20, or 30 (depending on desired network)
- IP: Static in corresponding subnet

### Switch Configuration Required
Your network switch must support and have VLANs configured:
- VLAN 10: Management
- VLAN 20: Servers
- VLAN 30: DMZ
- Trunk port to Proxmox host

## Multiple Network Interfaces

Use multiple physical NICs for different purposes.

### Configuration
**File**: `/etc/network/interfaces`

```bash
auto lo
iface lo inet loopback

# First interface - Management
iface enp0s3 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports enp0s3
        bridge-stp off
        bridge-fd 0
        # Management network only

# Second interface - VM network
iface enp0s8 inet manual

auto vmbr1
iface vmbr1 inet static
        address 10.0.0.1/24
        bridge-ports enp0s8
        bridge-stp off
        bridge-fd 0
        # VM traffic only

# Third interface - Storage network
iface enp0s9 inet manual

auto vmbr2
iface vmbr2 inet static
        address 192.168.100.100/24
        bridge-ports enp0s9
        bridge-stp off
        bridge-fd 0
        # Storage traffic (NFS/iSCSI)
```

### Use Case
- Separate management from VM traffic
- Dedicated storage network
- Higher bandwidth
- Network isolation

### VM Configuration
Choose appropriate bridge based on purpose:
- vmbr0: Management/Admin VMs
- vmbr1: Production VMs
- vmbr2: Storage VMs

## Bond Configuration

Link aggregation for redundancy and increased bandwidth.

### Configuration
**File**: `/etc/network/interfaces`

```bash
auto lo
iface lo inet loopback

# First physical interface
iface enp0s3 inet manual

# Second physical interface
iface enp0s8 inet manual

# Bond configuration (LACP/802.3ad)
auto bond0
iface bond0 inet manual
        bond-slaves enp0s3 enp0s8
        bond-mode 802.3ad
        bond-miimon 100
        bond-downdelay 200
        bond-updelay 200
        bond-lacp-rate 1
        bond-xmit-hash-policy layer3+4

# Bridge on top of bond
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports bond0
        bridge-stp off
        bridge-fd 0
```

### Install Required Packages
```bash
apt install ifenslave
```

### Bond Modes
- **balance-rr (0)**: Round-robin, load balancing
- **active-backup (1)**: One active, others backup
- **balance-xor (2)**: XOR load balancing
- **broadcast (3)**: All traffic on all interfaces
- **802.3ad (4)**: LACP aggregation (requires switch support)
- **balance-tlb (5)**: Adaptive transmit load balancing
- **balance-alb (6)**: Adaptive load balancing

### Use Case
- High availability
- Increased bandwidth
- Link redundancy
- Production environments

### Switch Configuration Required
For 802.3ad (LACP):
- Configure LACP/port channel on switch
- Add both ports to same LACP group
- Configure as trunk if using VLANs

## Advanced: Combined Configuration

Example combining multiple features.

### Configuration
**File**: `/etc/network/interfaces`

```bash
auto lo
iface lo inet loopback

# Physical interfaces for bond
iface enp0s3 inet manual
iface enp0s8 inet manual

# Bond configuration
auto bond0
iface bond0 inet manual
        bond-slaves enp0s3 enp0s8
        bond-mode 802.3ad
        bond-miimon 100
        bond-lacp-rate 1
        bond-xmit-hash-policy layer3+4

# Main VLAN-aware bridge
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge-ports bond0
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094

# Management VLAN
auto vmbr0.10
iface vmbr0.10 inet static
        address 10.0.10.1/24

# Production VLAN with NAT
auto vmbr0.20
iface vmbr0.20 inet static
        address 10.0.20.1/24
        post-up   iptables -t nat -A POSTROUTING -s '10.0.20.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.0.20.0/24' -o vmbr0 -j MASQUERADE

# Internal isolated network
auto vmbr1
iface vmbr1 inet static
        address 172.16.0.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
```

## Testing Network Configuration

### Apply Changes
```bash
# Test configuration syntax
ifup --no-act vmbr0

# Apply new configuration
ifreload -a

# If issues occur, revert
ifdown vmbr0 && ifup vmbr0
```

### Verify Configuration
```bash
# Show all interfaces
ip addr show

# Show bridge details
brctl show

# Test connectivity
ping -c 4 192.168.1.1  # Gateway
ping -c 4 8.8.8.8      # Internet

# Check routes
ip route show

# Check bond status (if applicable)
cat /proc/net/bonding/bond0
```

### Test VM Connectivity
```bash
# From VM, test:
ping 10.10.10.1        # Bridge IP
ping 192.168.1.1       # Gateway
ping 8.8.8.8           # Internet
nslookup google.com    # DNS
```

## Firewall Rules

### Allow Traffic Between Networks
```bash
# WARNING: These rules allow ALL traffic between networks
# For production use, restrict to specific ports and protocols

# Example: Allow specific services only
# Allow HTTP/HTTPS from vmbr0 to vmbr1
iptables -A FORWARD -i vmbr0 -o vmbr1 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -i vmbr0 -o vmbr1 -p tcp --dport 443 -j ACCEPT

# Allow established connections back
iptables -A FORWARD -i vmbr1 -o vmbr0 -m state --state ESTABLISHED,RELATED -j ACCEPT

# Or, allow all traffic (use with caution in homelab only)
# iptables -A FORWARD -i vmbr0 -o vmbr1 -j ACCEPT
# iptables -A FORWARD -i vmbr1 -o vmbr0 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### Block Traffic Between Networks
```bash
# Block traffic from vmbr1 to vmbr0
iptables -A FORWARD -i vmbr1 -o vmbr0 -j DROP
```

## Common Commands

### Restart Networking
```bash
# Restart networking service
systemctl restart networking

# Or reload interfaces
ifreload -a
```

### Bring Interface Up/Down
```bash
ifdown vmbr0
ifup vmbr0
```

### Show Bridge Information
```bash
brctl show
bridge link show
```

### Show IP Addresses
```bash
ip addr show
```

### Show Routes
```bash
ip route show
```

## Troubleshooting

### Bridge Not Working
```bash
# Check if bridge-utils is installed
apt install bridge-utils

# Verify bridge exists
brctl show

# Check interface is up
ip link show vmbr0
```

### NAT Not Working
```bash
# Verify IP forwarding
cat /proc/sys/net/ipv4/ip_forward

# Check NAT rules
iptables -t nat -L -n -v

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### VLAN Not Working
```bash
# Check VLAN configuration
cat /proc/net/vlan/config

# Install VLAN package
apt install vlan

# Load 8021q module
modprobe 8021q
```

## Additional Resources

- **Proxmox Network Configuration**: https://pve.proxmox.com/wiki/Network_Configuration
- **Linux Bridge**: https://wiki.linuxfoundation.org/networking/bridge
- **VLANs**: https://pve.proxmox.com/wiki/Network_Configuration#_vlan_802_1q
- **Bonding**: https://pve.proxmox.com/wiki/Network_Configuration#_bond
