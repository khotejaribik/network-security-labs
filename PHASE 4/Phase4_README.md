# Phase 4 — IDS Alerting with Suricata (pfSense)

## Goal
Enable **Suricata** on pfSense (IDS mode) and generate verifiable alerts based on real traffic from the lab:
- ICMP (ping) visibility
- TCP SYN visibility (web request)
- Portscan-style behavior (many SYN packets in a short window)

## Topology
- **pfSense**: LAN 10.0.1.1/24, DMZ 10.0.3.1/24, WAN VirtualBox NAT (10.0.2.0/24)
- **Kali (LAN)**: 10.0.1.100/24
- **Ubuntu (DMZ server)**: 10.0.3.10/24 (Python HTTP server on 8000)

## Suricata Setup Summary
1. Install Suricata: **System → Package Manager**
2. Enable Suricata globally: **Services → Suricata → Global Settings**
3. Enable Suricata on **LAN**: **Services → Suricata → Interfaces**
4. Add custom rules (LAN interface):

```text
alert icmp any any -> 10.0.3.10 any (msg:"LAB TEST ICMP to DMZ"; itype:8; sid:1000001; rev:1;)

alert tcp any any -> 10.0.3.10 any (msg:"LAB TEST TCP SYN to DMZ"; flags:S; sid:1000004; rev:1;)

alert tcp 10.0.1.0/24 any -> 10.0.3.10 any (msg:"LAB TEST PORTSCAN: many SYNs to DMZ"; flags:S; detection_filter:track by_src, count 15, seconds 5; sid:1000003; rev:2;)
```

> Note: Even if firewall policy blocks ICMP replies, Suricata can still observe the outbound request on LAN and alert on it.

## Tests (Kali)
### 1) ICMP
```bash
ping -c 2 10.0.3.10
```

### 2) Web request (port 8000)
```bash
curl -I http://10.0.3.10:8000/
```

### 3) Scan behavior
```bash
sudo nmap -sS -Pn -T4 -p 1-200 10.0.3.10
```

## Where to View Alerts
pfSense: **Services → Suricata → Alerts**
- Filter by source `10.0.1.100`
- Filter by destination `10.0.3.10`

## Evidence Files
- `Phase4_Diagram.png` — topology and custom rules
- `Kali_Ping.png` — ICMP test command
- `Kali_Curl.png` — HTTP test command
- `Kali_Nmap.png` — scan test command
- `pfSense_Suricata_Alerts_Placeholder.png` — replace with real pfSense Alerts screenshots

## Screenshots to Capture (GUI)
1. Suricata enabled on LAN interface (Interfaces page)
2. Suricata Alerts page showing:
   - LAB TEST ICMP to DMZ
   - LAB TEST TCP SYN to DMZ
   - LAB TEST PORTSCAN: many SYNs to DMZ
