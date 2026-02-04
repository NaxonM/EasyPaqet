# easyPaqet

An interactive installer and manager for Paqet tunnels between Iran and Kharej servers.

## Table of Contents

- [Overview](#overview)
- [Credits](#credits)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Testing](#testing)
- [Deployment Guide](#deployment-guide)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## Overview

easyPaqet simplifies the deployment and management of Paqet tunnels, providing an interface for setting up connections between Iran (client) and Kharej (server) infrastructure.

### Features

- One-line installation
- Quick install guided flow
- Interactive configuration wizard
- Automated firewall management
- Firewall rules persistence (iptables-save)
- Support for local archives (offline installation)
- Service management commands
- Built-in connection testing
- Startup status badge (install state, service state, version)
- Update Paqet binary from GitHub
- Custom download URL fallback
- Checksum verification (sha256/sha512)

---

## Credits

Special thanks to Evil (Evilcu) for the original findings and setup documentation:

- Telegram: [@Evilcu](https://t.me/Evilcu)

**Upstream Project:**
- [Paqet by hanselime](https://github.com/hanselime/paqet)

---

## Requirements

- Linux with systemd
- Root access
- curl or wget installed
- Stable internet connection (for online installation)

---

## Installation

### One-Line Install

Install easyPaqet directly on your VPS with a single command:

```bash
bash <(curl -Ls https://raw.githubusercontent.com/NaxonM/EasyPaqet/main/easyPaqet)
```

### Role-Specific Installation

#### Iran Server (Client)

Using local archive (recommended):

```bash
./easyPaqet install --role client --local-archive /root/paqet-linux-amd64-v1.0.0-alpha.12.tar.gz
```

Using automatic download:

```bash
./easyPaqet install --role client
```

Downloads attempt GitHub first. If GitHub fails, the script offers a custom URL or local archive.

#### Kharej Server

Using local archive (recommended):

```bash
./easyPaqet install --role server --local-archive /root/paqet-linux-amd64-v1.0.0-alpha.12.tar.gz
```

Using automatic download:

```bash
./easyPaqet install --role server
```

Downloads attempt GitHub first. If GitHub fails, the script offers a custom URL or local archive.

---

## Usage

### Interactive Mode

Launch the interactive configuration wizard:

```bash
./easyPaqet
```

The wizard will guide you through:

1. **Role Selection** - Choose between client (Iran) or server (Kharej)
2. **Source Selection** - Download automatically or use local archive
3. **Configuration** - Set interface, IP, MAC address, and ports
4. **Firewall Setup** - Optional firewall rule configuration (Kharej only)
5. **Service Management** - Start service and run connection tests (Iran only)

For KCP keys, install on Kharej first. The wizard will generate a random key on the server and show it so you can paste it on Iran.

On startup, the script displays a status badge showing install state, service state, and the detected binary version. The config confirmation also shows the current binary version when available.

### Quick Install (Guided)

Use the guided flow for a step-by-step setup:

```bash
./easyPaqet
```

Then choose:

- Quick Install (Guided)
- Pick role (server first, then client)
- Choose download method or local archive
- Enter network settings and KCP key (server generates it)

### Management Commands

Control your Paqet service with these commands:

```bash
# Service Control
./easyPaqet start          # Start the Paqet service
./easyPaqet stop           # Stop the Paqet service
./easyPaqet restart        # Restart the Paqet service
./easyPaqet status         # Check service status
./easyPaqet logs           # View service logs

# Firewall Management
./easyPaqet firewall-apply --role server --port <PAQET_PORT>
./easyPaqet firewall-clean --role server --port <PAQET_PORT>

# Test
./easyPaqet test

# View/Backup Config
./easyPaqet view-config
./easyPaqet backup-config

# Binary Update
./easyPaqet update-binary # Download and replace the Paqet binary from GitHub

# Checksum Verification
./easyPaqet install --role server --checksum <SHA256_OR_SHA512>

# Uninstall
./easyPaqet uninstall      # Remove Paqet and clean up
```

---

## Configuration

### Important Routing Concepts

**Note:** Users created on Kharej do NOT automatically route through Iran. Manual configuration is required.

### Traffic Flow Requirements

To route traffic from Iran to Kharej using Paqet:

1. Clients must connect to the Iran server
2. Iran inbound must mirror Kharej inbound settings
3. Each Kharej user needs a matching UUID on Iran

### x-ui Configuration (Iran Server)

#### Step 1: Create SOCKS Outbound

Configure the outbound to route through the Paqet tunnel:

```
Address: 127.0.0.1
Port: 1080
Tag: grim
```

#### Step 2: Create Matching Inbound

Create an inbound with identical settings to your Kharej configuration:

```
Protocol: vless
Port: <INBOUND_PORT>
Transport: xhttp
Path: /<INBOUND_PATH>
Host: <INBOUND_DOMAIN>
Tag: inbound-<INBOUND_PORT>
Clients: [Add matching UUIDs from Kharej]
```

#### Step 3: Configure Routing

Route the inbound to your SOCKS outbound:

```
inboundTag: inbound-<INBOUND_PORT>
outboundTag: grim
```

---

## Testing

### Connection Test

After installation, verify your tunnel connection:

```bash
curl -v https://google.com --proxy socks5h://127.0.0.1:1080
```

A successful connection through the SOCKS5 proxy indicates proper tunnel operation.

---

## Deployment Guide

### Deployment Workflow

Install Kharej first so it can generate the KCP key. Then install Iran and paste the key.

#### Iran VPS Setup (Client)

```bash
# Launch the installer
./easyPaqet

# Follow these steps:
# - Select: install
# - Choose role: client (Iran)
# - Select: download or local archive
# - Configure: interface, IP, MAC, and peer IP (Kharej)
# - Start service and run curl test
```

#### Kharej VPS Setup (Server)

```bash
# Launch the installer
./easyPaqet

# Follow these steps:
# - Select: install
# - Choose role: server (Kharej)
# - Select: download or local archive
# - Configure: interface, IP, and MAC
# - Apply firewall rules when prompted
```

### Offline Installation Method

For restricted environments or when GitHub access is limited.

Step 1: Download the latest release from [Paqet Releases](https://github.com/hanselime/paqet/releases/latest) and download the appropriate Linux tar.gz file.

Step 2: Transfer to your VPS and install:

```bash
# For Iran (client)
./easyPaqet install --role client --local-archive /root/paqet-linux-amd64-vX.Y.Z.tar.gz

# For Kharej (server)
./easyPaqet install --role server --local-archive /root/paqet-linux-amd64-vX.Y.Z.tar.gz
```

---

## Troubleshooting

### Common Issues

**Service won't start:**
- Check logs: `./easyPaqet logs`
- Verify configuration files in `/etc/paqet/`
- Ensure proper permissions

**Connection test fails:**
- Verify both Iran and Kharej services are running
- Check firewall rules on Kharej
- Confirm network connectivity between servers

**UUID mismatches:**
- Ensure UUIDs on Iran match those configured on Kharej
- Verify inbound/outbound configurations

---

## Contributing

Contributions are welcome. Feel free to report bugs, suggest features, submit pull requests, or improve documentation.

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.
