# Proxmox VE Homelab

Complete guide for setting up a homelab using Proxmox VE on your personal computer.

## Overview

This repository contains comprehensive documentation for installing and configuring a Proxmox VE homelab environment. Whether you're new to virtualization or an experienced user, these guides will help you get your homelab up and running.

## What is Proxmox VE?

Proxmox Virtual Environment (VE) is an open-source virtualization platform that combines:
- **KVM virtualization** - Full virtual machines
- **LXC containers** - Lightweight containers
- **Web-based management** - Easy-to-use interface
- **Built-in backup** - Automated backup solutions
- **High availability** - Clustering support

Perfect for homelabs, development environments, and learning!

## Quick Start

Want to get started right away? Follow our [Quick Start Guide](docs/QUICKSTART.md) to have your homelab running in under 30 minutes!

```bash
# Basic steps:
1. Download Proxmox VE ISO
2. Create bootable USB
3. Install on your PC
4. Access web interface at https://[your-ip]:8006
5. Create VMs and containers
```

## Documentation

### Getting Started
- **[Prerequisites](docs/PREREQUISITES.md)** - Hardware and software requirements checklist
- **[Quick Start Guide](docs/QUICKSTART.md)** - Get up and running in 30 minutes
- **[Installation Guide](docs/INSTALLATION.md)** - Detailed step-by-step installation instructions

### Configuration
- **[Network Examples](docs/NETWORK_EXAMPLES.md)** - Common network configurations and examples
- **[Troubleshooting](docs/TROUBLESHOOTING.md)** - Common issues and solutions

## Features

### What You'll Learn
- ✅ Installing Proxmox VE on a PC
- ✅ Configuring network bridges and NAT
- ✅ Creating virtual machines (VMs)
- ✅ Creating Linux containers (LXC)
- ✅ Setting up automated backups
- ✅ Managing storage and resources
- ✅ Basic security hardening
- ✅ Troubleshooting common issues

### What You Can Build
Once your homelab is set up, you can run:
- **Pi-hole** - Network-wide ad blocking
- **Home Assistant** - Home automation platform
- **Nextcloud** - Private cloud storage
- **Plex/Jellyfin** - Media server
- **Docker** - Container platform
- **pfSense/OPNsense** - Firewall/router
- **Git server** - Version control hosting
- **Wiki/Documentation** - Knowledge base
- And much more!

## Requirements

### Minimum Hardware
- 64-bit CPU with virtualization support (Intel VT-x or AMD-V)
- 4 GB RAM (8 GB+ recommended)
- 32 GB storage (SSD recommended)
- Ethernet network connection

### Recommended Hardware
- Multi-core CPU (4+ cores)
- 16 GB+ RAM
- 128 GB+ SSD for OS
- Additional storage for VMs
- Gigabit Ethernet

See [Prerequisites](docs/PREREQUISITES.md) for detailed requirements.

## Installation Overview

1. **Prepare**
   - Check hardware compatibility
   - Download Proxmox VE ISO
   - Create bootable USB drive

2. **Install**
   - Boot from USB
   - Follow installation wizard
   - Configure network settings
   - Set admin password

3. **Configure**
   - Access web interface
   - Update system
   - Configure storage
   - Set up networking

4. **Deploy**
   - Upload OS images
   - Create VMs and containers
   - Configure backups
   - Deploy services

## Support and Community

### Documentation
- **Official Proxmox Docs**: https://pve.proxmox.com/pve-docs/
- **Proxmox Wiki**: https://pve.proxmox.com/wiki/

### Community
- **Forum**: https://forum.proxmox.com/
- **Reddit**: r/Proxmox
- **Reddit**: r/homelab

### Getting Help
If you encounter issues:
1. Check the [Troubleshooting Guide](docs/TROUBLESHOOTING.md)
2. Search the [Proxmox Forum](https://forum.proxmox.com/)
3. Visit r/Proxmox on Reddit
4. Review official documentation

## Contributing

Found an error or have a suggestion? Contributions are welcome!
- Open an issue to report problems
- Submit pull requests for improvements
- Share your homelab configurations

## License

This documentation is provided as-is for educational purposes. Proxmox VE is licensed under AGPLv3.

## Acknowledgments

- Proxmox team for creating an excellent virtualization platform
- The homelab community for inspiration and support

---

**Ready to start?** Head over to the [Quick Start Guide](docs/QUICKSTART.md) or [Prerequisites](docs/PREREQUISITES.md)!
