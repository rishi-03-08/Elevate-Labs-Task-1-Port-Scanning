# Task 1: Scan Your Local Network for Open Ports

## Objective
Discover open ports on devices in my home network using Nmap, identify the services behind them, and assess the security risks of the exposure.

## Tools Used
- **Nmap 7.993** (with Npcap 1.90) on Windows

## Environment
- Network: home Wi-Fi, `192.168.0.0/24` (my PC: `192.168.0.5`, router: `192.168.0.1`)
- Date of scan: 5 November 2026
- Scope: only my own network. The virtual VirtualBox adapter (`192.168.56.x`) was not scanned.
- Scan covered the default **top 1000 TCP ports**. UDP was not scanned.

## Commands Run
```
ipconfig
nmap -sS 192.168.0.0/24 -oN scan_results.txt
nmap -sS -sV 192.168.0.0/24 -oN scan_services.txt
```
- `-sS`: TCP SYN (half-open) scan
- `-sV`: service/version detection
- `-oN`: save output as a text file

Result: **6 hosts up** out of 256 addresses.

## Findings

| IP | Device | Open ports | Services (from `-sV`) |
|---|---|---|---|
| 192.168.0.1 | TP-Link router (Linux 4.4.198) | 22, 23, 53, 80, 1900 | Dropbear SSH 2020.80, Telnet, dnsmasq 2.85, HTTP admin page, UPnP (libupnp 1.6.19) |
| 192.168.0.2 | Apple phone/tablet | 49152, 62078 | iPhone sync (tcpwrapped) |
| 192.168.0.3 | Oppo phone | none | All 1000 ports closed |
| 192.168.0.5 | My Windows PC | 135, 139, 445, 3306 | MS RPC, NetBIOS, SMB (likely), MySQL |
| 192.168.0.6 | LG webOS smart TV | 1074, 1247, 3000, 3001, 7000 | UPnP, LG HTTPS service, AirPlay (AirTunes rtspd) |
| 192.168.0.10 | Unknown device | none | All 1000 ports closed |

## Common Services on These Ports

| Port | Service | Purpose |
|---|---|---|
| 22 | SSH | Encrypted remote login |
| 23 | Telnet | Unencrypted remote login (legacy) |
| 53 | DNS | Name resolution (dnsmasq on the router) |
| 80 | HTTP | Router web admin page |
| 135 / 139 / 445 | RPC / NetBIOS / SMB | Windows networking and file sharing |
| 1900 | UPnP (SSDP) | Automatic device discovery and port mapping |
| 3306 | MySQL | Database server |
| 7000 | AirPlay | Apple screen/audio streaming to the TV |

## Security Risks and Mitigations

| Finding | Risk | Mitigation |
|---|---|---|
| Router Telnet (23) open | Unencrypted protocol, credentials can be sniffed | Disable Telnet, use SSH only if remote access is needed |
| Router admin page over HTTP (80) | Admin login sent in clear text on the LAN | Strong admin password, disable remote management, use HTTPS if available |
| Router UPnP (1900), libupnp 1.6.19 | Old library with a history of vulnerabilities; UPnP can open ports automatically | Disable UPnP if unused, update router firmware |
| Outdated router software (Dropbear 2020.80, Linux 4.4.198) | Known CVEs may apply | Update firmware, check the versions against NVD |
| MySQL (3306) on my PC | Database reachable from the network | Bind to localhost (`bind-address=127.0.0.1`) or block port 3306 in Windows Firewall |
| SMB / NetBIOS / RPC (445, 139, 135) on my PC | Common targets for worms and lateral movement | Keep Windows updated, set network profile to Private, disable file sharing if unused |
| Smart TV exposes UPnP and AirPlay | Reveals device name and model, limited firmware updates | Update TV firmware, restrict AirPlay access, consider a guest network |
| Phones with closed or filtered ports | Low exposure | No action needed |

## What I Learned
- How a SYN scan finds open ports without completing the TCP handshake
- How to work out a local IP range from `ipconfig`
- That version detection (`-sV`) gives much more useful information than port numbers alone
- How everyday devices like routers, TVs, and PCs expose services, and how to reduce that exposure

## Repository Contents
- `scan_results.txt`: basic SYN scan output
- `scan_services.txt`: SYN scan with version detection
- `screenshots/`: `01_ipconfig.png`, `02_nmap_syn_scan.png`, `02a_nmap_syn_scan.png` (MAC addresses and IPv6 addresses blacked out)
- `README.md`: this file

> Note: MAC addresses were masked and the long service-fingerprint blocks removed from the uploaded scan files for privacy.
