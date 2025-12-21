# Phase 5 (Lite) — Centralized Logging with Syslog (pfSense → Ubuntu)

## Goal
Centralize pfSense logs on the Ubuntu DMZ server (syslog) so you can investigate from one place:
- **Who** attempted access?
- **What** port/service?
- **Was it blocked/allowed?**
- **When** did it happen?

## Topology
- **pfSense**: LAN `10.0.1.1/24`, DMZ `10.0.3.1/24`, WAN VirtualBox NAT
- **Kali (LAN)**: `10.0.1.100`
- **Ubuntu (DMZ syslog server)**: `10.0.3.10`

## Implementation Summary
### Ubuntu: enable syslog receiver (UDP/514)
- rsyslog listens on UDP 514 (imudp)
- UFW allows syslog from pfSense DMZ IP:
```bash
sudo ufw allow from 10.0.3.1 to any port 514 proto udp
```
Verify listener:
```bash
sudo ss -ulnp | grep ':514' || true
```

### pfSense: forward logs remotely
pfSense GUI:
**Status → System Logs → Settings → Remote Logging Options**
- Enable Remote Logging ✅
- Remote log server: `10.0.3.10:514`
- Source Address: `10.0.3.1`
- Categories: at least **Firewall Events** and **System Events**

### Ubuntu: store pfSense logs in a dedicated file
- `/var/log/pfsense.log`

(Optional) Suricata service/status messages may also be stored separately:
- `/var/log/pfsense-suricata-service.log`

## Proof Test (Kali → pfSense → Ubuntu)
From Kali (blocked FTP/21 attempt):
```bash
nc -vz -w 2 10.0.3.10 21
```
On Ubuntu (central proof line):
```bash
sudo tail -n 20 /var/log/pfsense.log
```

Expected proof pattern in the log:
- `filterlog ... match,block ... 10.0.1.100 -> 10.0.3.10 ... dst port 21 ... Flags S`

## Evidence Images
- `Phase5_Diagram.png`
- `Kali_NC_Block.png`
- `Ubuntu_Syslog_Listener.png`
- `Ubuntu_pfsense_log_proof.png`

## Screenshots to Capture (optional)
- pfSense remote logging settings page
- Kali `nc` output
- Ubuntu `tail /var/log/pfsense.log` proof
