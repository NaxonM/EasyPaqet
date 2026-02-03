# easyPaqet

An interactive installer and manager for Paqet tunnels between Iran and Kharej servers.

## Credits

Thanks to Evil (Evilcu) for the original findings and setup notes:
- https://t.me/Evilcu
- https://t.me/Evilcu/21

Upstream Paqet project:
- https://github.com/hanselime/paqet

## Requirements

- Linux with systemd
- Root access
- curl or wget

## Quick Start

## One-Line Install (VPS)

This will fetch the latest script and run it on your server:

- bash <(curl -Ls https://raw.githubusercontent.com/NaxonM/EasyPaqet/main/easyPaqet.sh)

### Iran (client)

Local archive (recommended):

- ./easyPaqet install --role client --local-archive /root/paqet-linux-amd64-v1.0.0-alpha.12.tar.gz

Download (mirror first, then GitHub):

- ./easyPaqet install --role client

### Kharej (server)

Local archive (recommended):

- ./easyPaqet install --role server --local-archive /root/paqet-linux-amd64-v1.0.0-alpha.12.tar.gz

Download (GitHub first, then mirror):

- ./easyPaqet install --role server

## Interactive Mode

- ./easyPaqet

The script will guide you through:
- Role selection (client=Iran, server=Kharej)
- Download or local archive
- Config values (interface, IP, MAC, ports)
- Optional firewall rules (Kharej)
- Service restart and test (Iran)

## Management Commands

- ./easyPaqet start
- ./easyPaqet stop
- ./easyPaqet restart
- ./easyPaqet status
- ./easyPaqet logs
- ./easyPaqet firewall-apply --role server --port <PAQET_PORT>
- ./easyPaqet firewall-clean --role server --port <PAQET_PORT>
- ./easyPaqet uninstall

## Testing

The script runs the same test command suggested in the original notes:

- curl -v https://google.com --proxy socks5h://127.0.0.1:1080

## Notes

- Users on Kharej do not automatically pass through Iran. They must connect to an Iran inbound routed to the Paqet SOCKS outbound.
- When using x-ui on Iran, create an inbound with the same protocol/transport settings as the Kharej panel and route it to the SOCKS outbound (127.0.0.1:1080).

## Findings (Generalized)

### Key Points

- Users created on Kharej do not automatically go through Iran.
- To send traffic from Iran to Kharej using Paqet, clients must connect to Iran.
- The Iran inbound must match the Kharej inbound settings (protocol, port, transport, path, host).
- Each user on Kharej must have a matching UUID on Iran.

### Working Routing Model on Iran (x-ui)

Create a SOCKS outbound to the Paqet tunnel:

- Address: 127.0.0.1
- Port: 1080
- Tag: grim

Create an inbound on Iran with the same settings as Kharej:

- Protocol: vless
- Port: <INBOUND_PORT>
- Transport: xhttp
- Path: /<INBOUND_PATH>
- Host: <INBOUND_DOMAIN>
- Tag: inbound-<INBOUND_PORT>
- Clients: add each user UUID from Kharej

Route that inbound to the SOCKS outbound:

- inboundTag: inbound-<INBOUND_PORT>
- outboundTag: grim

## Step-by-Step Deployment

### 1) Iran VPS (client)

1. Run the script:
	- ./easyPaqet
2. Choose: install
3. Role: client (Iran)
4. Choose download or local archive
5. Fill in interface, IP, MAC, and peer IP (Kharej)
6. Start service and run the curl test

### 2) Kharej VPS (server)

1. Run the script:
	- ./easyPaqet
2. Choose: install
3. Role: server (Kharej)
4. Choose download or local archive
5. Fill in interface, IP, and MAC
6. Apply firewall rules when prompted

## Offline Method (No Download)

1) Download the latest Linux tar.gz from the Paqet releases page:
- https://github.com/hanselime/paqet/releases/latest

2) Copy it to your VPS, then install with the full path:
- ./easyPaqet install --role client --local-archive /root/paqet-linux-amd64-vX.Y.Z.tar.gz
- ./easyPaqet install --role server --local-archive /root/paqet-linux-amd64-vX.Y.Z.tar.gz
