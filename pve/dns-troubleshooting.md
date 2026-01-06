# Proxmox DNS Resolution Issue - Troubleshooting Guide

## Problem Summary

Proxmox server cannot resolve domain names when running `apt-get update`:

```
Failed to fetch http://download.proxmox.com/debian/pve/dists/trixie/InRelease  
Temporary failure resolving 'download.proxmox.com'
```

## Root Cause

The DNS nameservers were not configured in the network interface settings. The `/etc/resolv.conf` file was pointing to `127.0.0.1` (localhost), but no local DNS service was running on the server.

**Current `/etc/resolv.conf` content:**
```
search lan
nameserver 127.0.0.1
```

## Current Network Configuration

Located at `/etc/network/interfaces`:

```
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.10/24
        gateway 192.168.1.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0

iface nic1 inet manual
```

**Issue:** Missing `dns-nameservers` directive in the vmbr0 interface configuration.

## Solution

### Step 1: Add DNS Nameservers to Network Configuration

Edit `/etc/network/interfaces`:

```bash
nano /etc/network/interfaces
```

Update the configuration:

```
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.10/24
        gateway 192.168.1.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
        dns-nameservers 192.168.1.1 8.8.8.8 8.8.4.4
        dns-search lan

iface nic1 inet manual
```

**Key addition:**
- `dns-nameservers 192.168.1.1 8.8.8.8 8.8.4.4`
- `dns-search lan`

### Step 2: Apply Network Changes

```bash
ifreload -a
```

Alternative if `ifreload` doesn't work:
```bash
systemctl restart networking
```

### Step 3: Verify DNS Configuration

Check if `/etc/resolv.conf` has been updated:

```bash
cat /etc/resolv.conf
```

Expected output:
```
search lan
nameserver 192.168.1.1
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### Step 4: Test Connectivity and DNS Resolution

```bash
# Test basic internet connectivity
ping -c 4 8.8.8.8

# Test DNS resolution
ping -c 4 google.com

# Verify DNS lookup
nslookup google.com

# Test apt-get update
apt-get update
```

## DNS Server Options

Common DNS servers you can use:

| DNS Provider | Primary | Secondary | Notes |
|--------------|---------|-----------|-------|
| Router/Gateway | 192.168.1.1 | - | Usually has DNS forwarding |
| Google Public DNS | 8.8.8.8 | 8.8.4.4 | Fast and reliable |
| Cloudflare DNS | 1.1.1.1 | 1.0.0.1 | Privacy-focused |
| Quad9 | 9.9.9.9 | 149.112.112.112 | Security-focused |

## Quick Fix (Temporary)

If you need an immediate fix before editing the network configuration:

```bash
echo "nameserver 192.168.1.1" > /etc/resolv.conf
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

**Note:** This is temporary and will be overwritten on reboot. Always configure DNS in `/etc/network/interfaces` for a permanent solution.

## Additional Troubleshooting

If issues persist after configuration:

1. **Check if resolvconf is installed:**
   ```bash
   dpkg -l | grep resolvconf
   ```

2. **Install and enable resolvconf if needed:**
   ```bash
   apt-get install resolvconf
   systemctl enable resolvconf
   systemctl start resolvconf
   ```

3. **Verify gateway is reachable:**
   ```bash
   ping 192.168.1.1
   ```

4. **Check routing table:**
   ```bash
   ip route show
   ```

5. **Check network interface status:**
   ```bash
   ip addr show vmbr0
   ```

## Verification Checklist

- [ ] DNS nameservers added to `/etc/network/interfaces`
- [ ] Network service restarted with `ifreload -a`
- [ ] `/etc/resolv.conf` shows correct nameservers
- [ ] Can ping IP addresses (e.g., `ping 8.8.8.8`)
- [ ] Can ping domain names (e.g., `ping google.com`)
- [ ] `nslookup google.com` works
- [ ] `apt-get update` completes successfully

## Date

Issue documented: January 6, 2026
