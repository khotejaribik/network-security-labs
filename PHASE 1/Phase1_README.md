# Phase 1 — Make Networking Visible (DNS / TCP / HTTP) ✅

This Phase 1 lab turns “networking theory” into **visible, repeatable proof** using a VirtualBox Ubuntu VM, `tcpdump`, and `curl`.

---

## What you achieved in Phase 1

- Confirmed your VM network mode (**VirtualBox NAT**) and understood why NAT needs **port forwarding** for inbound access.
- Enabled and verified **SSH** on Ubuntu.
- SSH’d from **Windows → Ubuntu VM** through a NAT port-forward (`127.0.0.1:2222 → VM:22`).
- Captured and explained **DNS** query/response packets (UDP/53).
- Captured and explained **TCP 3‑way handshake** (SYN → SYN‑ACK → ACK).
- Hosted your own HTTP service inside the VM and accessed it from Windows via port-forward (`127.0.0.1:8080 → VM:8000`).
- Captured and viewed **plaintext HTTP** request payload (`GET / HTTP/1.1`, `Host: …`) using `tcpdump -A`.

---

## Environment

- Host: **Windows**
- VM: **Ubuntu 24.04.x** in **VirtualBox**
- VM interface: `enp0s3`
- VM IP (NAT): `10.0.2.15/24`
- NAT gateway: `10.0.2.2`
- DNS resolver (VirtualBox NAT): `10.0.2.3`

> Note: Your host IP can also be 10.x.x.x (e.g., 10.250.x.x). That does **not** mean it’s the same network. NAT uses its own private 10.0.2.0/24 network.

---

## 1) Confirm NAT mode inside Ubuntu

Run:

```bash
ip -4 a
ip route
```

Expected NAT proof:

- VM IP like `10.0.2.15`
- Default route like:

```text
default via 10.0.2.2 dev enp0s3 ...
```

---

## 2) Update Ubuntu + install networking tools

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install net-tools dnsutils tcpdump curl openssh-server
```

Tools used:
- `dnsutils` → `dig`
- `tcpdump` → packet capture
- `curl` → generate HTTP/HTTPS traffic
- `openssh-server` → SSH service

---

## 3) Enable and verify SSH service

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ssh
sudo systemctl status ssh --no-pager
```

Expected:
- `Active: active (running)`
- `Server listening on ... port 22`

### If SSH fails to start
Run the config test:

```bash
sudo /usr/sbin/sshd -t -e
```

If it says “no hostkeys available”:

```bash
sudo ssh-keygen -A
sudo systemctl restart ssh
```

---

## 4) VirtualBox NAT Port Forward: Windows SSH → Ubuntu

Because NAT hides the VM, Windows **cannot** directly SSH to `10.0.2.15`. We create a NAT forward:

**VirtualBox → VM → Settings → Network → Adapter 1 (NAT) → Advanced → Port Forwarding**

Add rule:

- Name: `ssh`
- Protocol: `TCP`
- Host IP: *(blank)*
- Host Port: `2222`
- Guest IP: `10.0.2.15`
- Guest Port: `22`

Then from Windows PowerShell:

```powershell
ssh -p 2222 ubuntu@127.0.0.1
```

**Important meaning:**
- `127.0.0.1` is **Windows localhost**, not Ubuntu.
- VirtualBox forwards `Windows:127.0.0.1:2222` → `Ubuntu:10.0.2.15:22`.

---

## 5) DNS visibility lab (UDP/53)

### Capture DNS packets
Terminal A:

```bash
sudo tcpdump -ni enp0s3 udp port 53
```

### Generate a DNS request
Terminal B:

```bash
dig google.com
```

Example capture interpretation:

```text
10.0.2.15.58661 > 10.0.2.3.53: ... A? google.com
10.0.2.3.53 > 10.0.2.15.58661: ... A 142.xxx.xxx.xxx
```

- Source port `58661` = ephemeral client port
- Destination `:53` = DNS
- Reply returns to the same ephemeral port

Stop capture with **Ctrl+C**.

---

## 6) TCP handshake lab (SYN / SYN‑ACK / ACK)

Start capture (shows SYN/ACK activity):

```bash
sudo tcpdump -ni enp0s3 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
```

Generate traffic:

```bash
curl -I https://google.com
```

Interpretation:
- `[S]` → SYN (start connection)
- `[S.]` → SYN‑ACK (server acknowledges)
- `[.]` → ACK (client confirms)

You also observed:
- `[P.]` → data push + ack
- `[F.]` → connection close (FIN)

---

## 7) Create your own HTTP service in Ubuntu (so you control everything)

Start a simple web server inside the VM:

```bash
mkdir -p ~/web
echo "Hello, This is from Ubuntu VM" > ~/web/index.html
cd ~/web
python3 -m http.server 8000
```

Leave it running.

---

## 8) VirtualBox NAT Port Forward: Windows HTTP → Ubuntu (8080 → 8000)

Add a second forwarding rule in VirtualBox:

- Name: `http`
- Protocol: `TCP`
- Host IP: *(blank)*
- Host Port: `8080`
- Guest IP: `10.0.2.15`
- Guest Port: `8000`

Test from Windows:

### Note about “curl” in PowerShell
PowerShell often maps `curl` to `Invoke-WebRequest`.

Use real curl:

```powershell
curl.exe http://127.0.0.1:8080
```

Expected output:
- `Hello, This is from Ubuntu VM`

---

## 9) Capture plaintext HTTP inside the VM

Terminal A (packet capture with ASCII):

```bash
sudo tcpdump -A -ni enp0s3 tcp port 8000
```

Terminal B (Windows request again):

```powershell
curl.exe -v http://127.0.0.1:8080
```

In tcpdump output you should see payload like:

```text
GET / HTTP/1.1
Host: 127.0.0.1:8080
```

This is the key learning:
- **HTTP is plaintext**
- **HTTPS is encrypted** (you can still see handshake/metadata, not the content)

Stop capture with **Ctrl+C**.

---

## Phase 1 Deliverables (Portfolio)

Create a folder in a repo like `networking-labs/phase1/` and include:

1) `README.md` (this file)
2) Screenshots or pasted logs for:
   - `ip route` showing NAT gateway `10.0.2.2`
   - SSH working from Windows
   - DNS tcpdump lines (2 lines are enough)
   - TCP handshake tcpdump (first 3 lines show SYN/SYN‑ACK/ACK)
   - HTTP tcpdump showing `GET / HTTP/1.1`
3) A simple network diagram (can be drawn in PowerPoint):
   - Windows host
   - VirtualBox NAT
   - Ubuntu VM (10.0.2.15)
   - Port forward rules (2222→22, 8080→8000)

---

## What’s next (Phase 2 preview)

- Add a router/firewall VM (pfSense)
- Create **two subnets** (LAN + DMZ)
- Apply firewall rules and prove “allowed vs blocked”
- Add IDS (Suricata/Zeek) and generate alerts safely in-lab

---

**End of Phase 1 ✅**
