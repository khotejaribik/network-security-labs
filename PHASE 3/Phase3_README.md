# Phase 3 — Host Hardening + Detection (UFW + Fail2ban)

## Goal
1. **Network-level visibility:** pfSense logs blocked traffic (e.g., LAN → DMZ FTP/21).
2. **Host hardening:** Ubuntu DMZ server exposes only required ports.
3. **Detection + response:** Fail2ban bans repeated SSH login failures.

## Lab Topology
- **pfSense**
  - WAN: VirtualBox NAT (10.0.2.0/24)
  - LAN: 10.0.1.1/24 (DHCP enabled)
  - DMZ (OPT1): 10.0.3.1/24
- **Kali (LAN client)**: 10.0.1.100/24
- **Ubuntu (DMZ server)**: 10.0.3.10/24

> Note: DMZ originally used 10.0.2.0/24, which conflicted with pfSense WAN (VirtualBox NAT uses 10.0.2.0/24). DMZ was moved to **10.0.3.0/24** to remove the subnet collision.

## Steps Performed

### A) pfSense firewall logging (evidence)
1. Enable **“Log packets that are handled by this rule”** on the LAN→DMZ block rule.
2. Generate blocked traffic from Kali:
   ```bash
   nc -vz -w 2 10.0.3.10 21
   ```
3. Confirm the block shows up in: **Status → System Logs → Firewall**.

### B) Ubuntu hardening with UFW
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 8000/tcp
sudo ufw enable
sudo ufw status verbose
```

### C) Ubuntu detection with Fail2ban (sshd jail)
```bash
sudo apt update
sudo apt -y install fail2ban
sudo systemctl enable --now fail2ban

sudo tee /etc/fail2ban/jail.d/sshd.local >/dev/null <<'EOF'
[sshd]
enabled = true
maxretry = 3
findtime = 10m
bantime = 30m
EOF

sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

### D) Proof test
From Kali, fail SSH login 3 times:
```bash
ssh ubuntu@10.0.3.10
```
Then on Ubuntu:
```bash
sudo fail2ban-client status sshd
```
Expected: `Banned IP list` includes `10.0.1.100`.

## Evidence Files
- `Phase3_Diagram.png` — lab topology diagram
- `Ubuntu_UFW.png` — UFW status evidence
- `Ubuntu_Fail2ban.png` — Fail2ban ban evidence
- `Kali_NC_21.png` — blocked FTP/21 test
- `Ubuntu_Ping_Block.png` — DMZ→LAN blocked evidence
- `pfSense_Log_Placeholder.png` — where to insert your pfSense GUI log screenshot

## Screenshots to Capture (GUI)
1. pfSense **Status → Interfaces** (WAN/LAN/OPT1 IPs)
2. pfSense **Firewall → Rules → LAN** (allow 22/8000 and block to DMZ)
3. pfSense **Status → System Logs → Firewall** (show blocked port 21 entry)
