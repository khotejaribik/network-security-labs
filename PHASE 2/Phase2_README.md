# Phase 2: Network Segmentation Lab (pfSense LAN + DMZ)

## Goal
Build a segmented lab network and prove:
- LAN -> DMZ: only TCP 22 (SSH) and TCP 8000 (HTTP lab) are allowed.
- LAN -> DMZ: other ports (example: TCP 21) are blocked.
- DMZ -> LAN: blocked.

## Topology (VirtualBox)
- pfSense (router/firewall)
  - WAN: NAT (VirtualBox NAT)
  - LAN: Internal Network `intnet-lan`, gateway `10.0.1.1/24`
  - DMZ/OPT1: Internal Network `intnet-dmz`, gateway `10.0.3.1/24`
- Kali (LAN client): `10.0.1.100/24` via DHCP, GW `10.0.1.1`
- Ubuntu (DMZ server): `10.0.3.10/24` static, GW `10.0.3.1`

> Note (important): Do NOT use `10.0.2.0/24` for DMZ when pfSense WAN is VirtualBox NAT, because NAT typically uses `10.0.2.0/24` and you will create a subnet conflict.

## Evidence (commands)
### Allowed: SSH (22)
Kali:
- `ssh ubuntu@10.0.3.10`

### Allowed: HTTP lab (8000)
Kali:
- `curl -I http://10.0.3.10:8000`

### Blocked: FTP test (21)
Kali:
- `nc -vz -w 2 10.0.3.10 21`  -> Connection timed out

### Blocked: DMZ -> LAN
Ubuntu:
- `ping -c 2 10.0.1.100` -> 100% packet loss

## Screenshot checklist (for your report)
1. pfSense Status -> Interfaces (WAN/LAN/OPT1 IPs)
2. pfSense Firewall -> Rules -> LAN (allow 22, allow 8000, block to 10.0.3.0/24)
3. pfSense Firewall -> Rules -> OPT1 (block DMZ -> LAN)
4. Kali terminal: SSH success
5. Kali terminal: curl HTTP 200 OK
6. Kali terminal: nc timeout to 21
7. Ubuntu terminal: ip route and DMZ->LAN ping loss
