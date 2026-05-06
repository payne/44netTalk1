---
marp: true
theme: default
paginate: true
backgroundColor: #1a1a2e
color: #eaeaea
style: |
  section {
    font-family: 'Courier New', monospace;
    background-color: #1a1a2e;
    color: #eaeaea;
  }
  h1 {
    color: #00d4ff;
    border-bottom: 2px solid #00d4ff;
    padding-bottom: 8px;
  }
  h2 {
    color: #7fdbff;
  }
  h3 {
    color: #39ff14;
  }
  code {
    background: #0d0d1a;
    color: #39ff14;
    padding: 2px 6px;
    border-radius: 3px;
  }
  pre {
    background: #0d0d1a;
    border-left: 4px solid #00d4ff;
    padding: 16px;
  }
  pre code {
    background: transparent;
    padding: 0;
  }
  strong {
    color: #ffdd57;
  }
  a {
    color: #00d4ff;
  }
  .callout {
    background: #0d2a3a;
    border-left: 4px solid #39ff14;
    padding: 12px 16px;
    margin: 8px 0;
  }
  table {
    border-collapse: collapse;
    width: 100%;
  }
  th {
    background: #0d2a3a;
    color: #00d4ff;
    padding: 8px;
  }
  td {
    border: 1px solid #333;
    padding: 8px;
  }
---

<!-- _paginate: false -->
<!-- _backgroundColor: #0d0d1a -->

# 📡 Ham Radio Digital Infrastructure

## Banana Pi OpenWRT-One · 44Net · EchoLink
### Connecting to the K0USA .94 NET via the Internet

---

**Presenter:** [Your Call Sign]
**Date:** 2026
**Duration:** ~40 minutes

> *"The internet is just a big radio network — let's use it like one."*

---

# Agenda

1. **Why this stack?** — The big picture (3 min)
2. **Banana Pi OpenWRT-One** — The hardware (5 min)
3. **44Net / AMPRNet** — Ham-licensed IP space (8 min)
4. **EchoLink** — Voice over IP for hams (5 min)
5. **The K0USA .94 Repeater & NETs** — Our target (3 min)
6. **Step-by-step: EchoLink node on Linux** — SVXLink (10 min)
7. **Step-by-step: Windows 11 Pro alternative** (3 min)
8. **Putting it all on the Banana Pi** (3 min)
9. **Q&A & Resources** (← check the last slide)

---

# Why This Stack?

### The problem worth solving

- You want to **participate in a local NET** on K0USA 147.940 MHz
- You're not near a radio — maybe at work, traveling, or portable
- **EchoLink** lets licensed hams connect via the internet
- But most EchoLink guides assume a Windows PC on your desk

### Our solution

```
[Your radio / laptop / phone]
        │  EchoLink
        ▼
[SVXLink on Banana Pi + 44Net IP]
        │  audio + PTT
        ▼
[Radio linked to K0USA .94 repeater]
        │  RF
        ▼
[K0USA 147.940 MHz NET]
```

---

# The Banana Pi OpenWRT-One

### AP-24.XY — The official OpenWrt community board

![bg right:35% 90%](https://www.banana-pi.org/images/bananapi/OpenWrt-One.jpg)

| Feature | Spec |
|---|---|
| SoC | MediaTek MT7981B (Filogic 810) |
| CPU | Dual ARM Cortex-A53 @ 1.3 GHz |
| RAM | 1 GB DDR4 |
| Flash | 256 MiB SPI NAND + 16 MiB NOR |
| WiFi | MT7976C — WiFi 6 dual-band (2.4/5 GHz) |
| Ethernet | 1× 2.5 GbE WAN, 1× 1 GbE LAN |
| USB | 1× USB 2.0 Type-A |
| Expansion | mikroBUS™ connector |
| Storage | M.2 2230/2242 NVMe PCIe 2.0 x1 |

---

# Why the Banana Pi OpenWRT-One for Ham Radio?

### It was *designed* to run OpenWrt — not hacked into it

- **Official OpenWrt board** — mainline kernel support, no patches needed
- **Dual storage = unbrickable** — 16 MiB NOR holds a rescue firmware
- **mikroBUS expansion** — add SPI/I²C/UART radios, GPS, or peripherals
- **M.2 slot** — add NVMe SSD for logging, audio recordings, databases
- **USB 2.0** — USB audio interface for radio TX/RX audio
- **Low power** — runs on PoE, great for remote installs
- **OpenWrt = full Linux** — `opkg install` anything you need

### Ham use cases on this board
- 44Net gateway (WireGuard or IPIP tunnel)
- SVXLink EchoLink node
- APRS iGate / digipeater
- Winlink gateway (with USB TNC)
- AREDN node controller

---

# Getting the Banana Pi OpenWRT-One

### Where to buy
- **Amazon** — search "Banana Pi OpenWRT-One" (~$40–55 USD)
- **AliExpress** — directly from manufacturer
- **Banana Pi official store** — banana-pi.org

### What you also need
- USB-C power supply (5V/3A minimum) **or** PoE injector
- microSD card (for initial flash, optional after)
- USB audio adapter (for radio audio) — e.g., CM108-based (~$8)
- USB-to-serial adapter (for PTT via RTS/DTR pin)
- Ethernet cable (for initial setup)
- **Radio interface cable** — audio in/out + PTT to your transceiver

---

# 44Net / AMPRNet

### What is it?

**AMPRNet** = Amateur Packet Radio Network
**44Net** = the `44.0.0.0/8` IP block

- Allocated in **1981** — one of the earliest registered /8 blocks
- Managed by **ARDC** (Amateur Radio Digital Communications)
- Only licensed amateur radio operators may request addresses
- IPs are **globally routable** — reachable from the public internet
- Free to use — no cost for the address space

### The address space today
| Block | Purpose |
|---|---|
| `44.0.0.0/9` | USA subnets |
| `44.128.0.0/10` | Rest of world |
| `44.192.0.0/10` | **Sold in 2019** (not ham radio anymore) |

---

# How 44Net Connectivity Works

### Option 1: IPIP Tunnel Mesh (classic)

```
[Your 44.x.x.x host]
      │  IPIP encapsulation over your ISP connection
      ▼
[UCSD tunnel router — ampr.org]
      │  routes 44.x.x.x traffic globally
      ▼
[Other 44Net nodes worldwide]
```

- Your ISP sees encrypted-looking IP-in-IP packets
- The 44.x.x.x address is your *inner* routable address
- Requires a static or semi-static public IP on your gateway

### Option 2: 44Net Connect (new — 2025/2026)

```
[Your device]
      │  WireGuard VPN
      ▼
[connect.44net.cloud]
      │  hands you a 44.x.x.x address automatically
```

**Easiest option for beginners** — no BGP, no tunnel config

---

# Registering for 44Net — Step by Step

### Prerequisites
- **Valid amateur radio license** (any class, any country)
- An email address
- Know your callsign

---

### Step 1 — Create your account

```
https://portal.ampr.org  →  click REGISTER
```

Fill in: name, email, **callsign**, password
Check your email → click the verification link

---

### Step 2 — Verify your callsign

Log into the portal → go to **My Profile**
Enter your callsign → ARDC verifies against FCC / ITU databases
*This may take 24–48 hours for manual review*

---

# Registering for 44Net — Step by Step (cont.)

### Step 3 — Request address space

Portal → **Request address space**
- Choose your **region/state**
- Choose type: **"End User"** (for a single node or small subnet)
- Request a **/29** (6 usable addresses) or **/30** (2 usable) for starters
- Describe your use case: *"EchoLink node + experimental networking"*

### Step 4 — Register your gateway

Once approved, portal → **Gateways** → Add Gateway
- Enter your gateway's **public IP address**
- Enter your **44.x.x.x** subnet

### Step 5 — Add DNS records

Portal → **DNS** → add an `A` record for each host
```
mynode.k0abc.ampr.org  →  44.x.x.x
```

> **The UCSD tunnel router will not route to you without a DNS entry.**

---

# 44Net Connect — The Easy Path (2025/2026)

### If you just want a 44Net IP with minimal setup

```bash
# Install WireGuard
sudo apt install wireguard

# Get your config from https://connect.44net.cloud/
# (requires portal.ampr.org account first)

# Save config as /etc/wireguard/44net.conf
# Then:
sudo wg-quick up 44net

# Verify your 44 address
ip addr show 44net
```

### What you get
- A `44.x.x.x` address assigned automatically
- Full connectivity to other 44Net nodes
- No static public IP required on your end
- Works behind NAT / CGNAT

---

# EchoLink — Voice Over IP for Licensed Hams

### What is EchoLink?

- Created by **Jonathan Taylor, K1RFD** in 2002
- Connects licensed amateur radio operators via **VoIP**
- Over **250,000 validated users** worldwide
- Three node types:

| Type | Description |
|---|---|
| **Simplex (L)** | PC with mic/speaker, no radio |
| **Single (-)** | PC linked to a simplex radio |
| **Repeater (R)** | PC linked to a repeater |

### Key facts
- **Free** — no subscription
- Requires **FCC license validation** before use
- Uses **GSM codec** for audio compression
- Ports: UDP 5198, UDP 5199, TCP 5200

---

# EchoLink Network Architecture

```
[EchoLink Client — your phone/laptop]
         │  UDP 5198/5199
         ▼
[EchoLink Directory Servers]
         │  validates callsign, routes connection
         ▼
[SVXLink Node on Banana Pi — K0ABC-L]
         │  USB audio + PTT
         ▼
[Your radio TX/RX]
         │  RF — 147.940 MHz
         ▼
[K0USA Repeater .94]
         │  RF rebroadcast
         ▼
[NET participants on 147.940 MHz]
```

---

# The K0USA .94 Repeater

### 147.940 MHz — Denver / Front Range area

| Parameter | Value |
|---|---|
| Output frequency | **147.940 MHz** |
| Input (your TX) | 147.340 MHz (−600 kHz offset) |
| CTCSS tone | Verify with repeaterbook.com for K0USA |
| Mode | FM voice |
| Coverage | Denver metro + mountains |

### The NET concept

A **NET** is a scheduled on-air meeting:
- Held at a regular time on a repeater frequency
- A **Net Control Station (NCS)** runs the check-in list
- Participants check in, pass traffic, or just say hello
- EchoLink lets remote operators **check in from anywhere**

> **Your goal:** run an EchoLink-linked node so anyone connecting to your EchoLink node number can participate in the K0USA NET.

---

# SVXLink — The Engine

### Why SVXLink, not the Windows EchoLink app?

- **Native Linux** — runs headless, no GUI needed
- **Full repeater controller** — CTCSS, hang time, IDs, courtesy tones
- **Modular** — EchoLink is just one module
- **Rock solid** — runs for months unattended
- **Open source** — `github.com/sm0svx/svxlink`
- **Packaged** — available in Debian, Ubuntu, Raspberry Pi OS

### Hardware you need

```
[Transceiver]
    ├── Audio OUT  ──►  USB audio adapter LINE IN
    ├── Audio IN   ──◄  USB audio adapter LINE OUT  
    └── PTT        ──◄  USB-serial adapter (RTS or DTR pin)
                             │
                         [Banana Pi / Linux PC]
                             │  SVXLink process
```

---

# Linux Step-by-Step: Install SVXLink

### Tested on: Debian 12, Ubuntu 24.04, Raspberry Pi OS

### Step 1 — Update and install

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y svxlink-server qtel \
    alsa-utils libsox-fmt-all \
    tcpdump curl git
```

### Step 2 — Identify your USB audio device

```bash
# List capture devices (radio audio IN to PC)
arecord -l

# List playback devices (PC audio OUT to radio)  
aplay -l

# Example output:
# card 1: Device [USB Audio Device], device 0: USB Audio [USB Audio]
```

> Note the **card number** — you'll need it (e.g., `plughw:1,0`)

---

# Linux Step-by-Step: Audio Level Setup

### Step 3 — Set audio levels with alsamixer

```bash
# Open mixer for your USB audio card (replace 1 with your card number)
alsamixer -c 1
```

- Press **F4** → Capture section → set mic/line input to ~70%
- Press **F3** → Playback section → set speaker/line out to ~70%
- Press **Esc** to exit

### Step 4 — Save ALSA settings

```bash
sudo alsactl store
```

### Step 5 — Test audio loopback

```bash
# Record 5 seconds from radio, play it back
arecord -D plughw:1,0 -f cd -t wav -d 5 /tmp/test.wav
aplay -D plughw:1,0 /tmp/test.wav
```

> You should hear your radio audio played back. Adjust levels until clear.

---

# Linux Step-by-Step: PTT Interface

### Step 6 — Identify your serial port

```bash
# Plug in your USB-serial adapter, then:
dmesg | tail -20 | grep tty

# Usually shows: /dev/ttyUSB0  or  /dev/ttyACM0
```

### Step 7 — Add your user to the dialout group

```bash
sudo usermod -aG dialout $USER
# Log out and back in for this to take effect
```

### Step 8 — Test PTT manually

```bash
# Key up the radio (assert RTS high)
python3 -c "
import serial, time
s = serial.Serial('/dev/ttyUSB0')
s.rts = True
print('PTT ON — radio should be transmitting')
time.sleep(3)
s.rts = False
print('PTT OFF')
s.close()
"
```

> Watch your radio's TX indicator. If it keys up, PTT is working.

---

# Linux Step-by-Step: SVXLink Main Config

### Step 9 — Edit `/etc/svxlink/svxlink.conf`

```ini
[GLOBAL]
MODULE_PLUG_IN_PATH=/usr/lib/svxlink
CFG_DIR=/etc/svxlink/svxlink.d
TIMESTAMP_FORMAT=%c

[LOGIC:SimplexLogic]
TYPE=Simplex
RX=Rx1
TX=Tx1
MODULES=ModuleEchoLink
CALLSIGN=K0ABC            # ← YOUR callsign
SHORT_IDENT_INTERVAL=15
LONG_IDENT_INTERVAL=60
IDENT_ONLY_AFTER_TX=1
IDLE_TIMEOUT=300
```

> Replace `K0ABC` with your actual callsign throughout.

---

# Linux Step-by-Step: RX and TX Config

### Still in `/etc/svxlink/svxlink.conf`

```ini
[RX:Rx1]
TYPE=Local
AUDIO_DEV=alsa:plughw:1,0   # ← your USB audio card
AUDIO_CHANNEL=0
SQL_DET=CTCSS               # or VOX if no CTCSS from radio
CTCSS_FQ=100.0              # ← tone your radio decodes
CTCSS_THRESH=15
SERIAL_PORT=/dev/ttyUSB0
SERIAL_PIN=CTS              # COS input from radio (optional)

[TX:Tx1]
TYPE=Local
AUDIO_DEV=alsa:plughw:1,0   # ← same USB audio card
AUDIO_CHANNEL=0
PTT_TYPE=SERIAL
SERIAL_PORT=/dev/ttyUSB0
SERIAL_PIN=RTS              # PTT output to radio
PTT_DELAY=0
TX_DELAY=500
MAX_TONE_AMP=50
```

---

# Linux Step-by-Step: EchoLink Module Config

### Step 10 — Edit `/etc/svxlink/svxlink.d/ModuleEchoLink.conf`

```ini
[ModuleEchoLink]
ID=2
TIMEOUT=60

# Your EchoLink credentials
CALLSIGN=K0ABC-L            # append -L (Link) or -R (Repeater)
PASSWORD=YourEchoLinkPass   # set at echolink.org
SYSOPNAME=Your Name
LOCATION=Denver CO USA

# EchoLink server
SERVER=naeast.echolink.org

# Ports (must be forwarded on your router)
BIND_PORT=5198

# Who can connect
ACCEPT_INCOMING=1
# To restrict to specific calls:
# CALLSIGN_ACL=K0*,W0*,N0*

# Link to K0USA .94 — connect on startup (optional)
# LINK_INCOMING=*
```

---

# Linux Step-by-Step: EchoLink Registration

### Before SVXLink will connect, you must validate your callsign

### Step 11 — Register at echolink.org

```
1. Go to:  https://www.echolink.org/validation/
2. Enter your callsign
3. Enter your name and email
4. EchoLink staff will verify against FCC/ITU database
   — takes 1-3 business days
5. You'll receive an email with your password
```

### Step 12 — Set your password in the config

```bash
sudo nano /etc/svxlink/svxlink.d/ModuleEchoLink.conf
# Set PASSWORD= to the password emailed to you
```

> **Never share this password.** It's tied to your callsign and license.

---

# Linux Step-by-Step: Port Forwarding

### Step 13 — Forward EchoLink ports on your router

| Protocol | Port | Direction | Purpose |
|---|---|---|---|
| UDP | 5198 | In + Out | Voice audio |
| UDP | 5199 | In + Out | Text / status |
| TCP | 5200 | Outbound only | Directory server |

### On OpenWrt (your Banana Pi):

```
LuCI → Network → Firewall → Port Forwards
→ Add rule:
    Name: EchoLink-UDP-5198
    Protocol: UDP
    External port: 5198
    Internal IP: [IP of your SVXLink machine]
    Internal port: 5198
→ Repeat for 5199
```

### Or via command line on OpenWrt:

```bash
uci add firewall redirect
uci set firewall.@redirect[-1].dest_port='5198'
uci set firewall.@redirect[-1].proto='udp'
uci set firewall.@redirect[-1].dest_ip='192.168.1.100'
uci commit firewall && /etc/init.d/firewall restart
```

---

# Linux Step-by-Step: Start SVXLink

### Step 14 — Start and enable SVXLink

```bash
# Start the service
sudo systemctl start svxlink

# Enable on boot
sudo systemctl enable svxlink

# Check status
sudo systemctl status svxlink

# Watch the log in real time
sudo journalctl -fu svxlink
```

### What success looks like in the log

```
EchoLink: Logging in to server naeast.echolink.org...
EchoLink: Logged in as K0ABC-L (node #XXXXXX)
EchoLink: Ready to accept connections
```

> Write down your **node number** — that's what other hams will dial.

---

# Connecting to the K0USA .94 NET

### How to connect your EchoLink node TO the K0USA repeater

### Option A — They connect to you
- K0USA net control or a trustee keys your EchoLink node number
- Their repeater controller connects to `K0ABC-L`
- All audio bridges between EchoLink and K0USA .94

### Option B — Your node connects to theirs
In `ModuleEchoLink.conf`, or live via DTMF:

```ini
# Auto-connect to K0USA EchoLink node on startup
# (get their node number from echolink.org/links)
AUTOCONNECT_ON_START=K0USA-R
```

### Joining the NET via EchoLink client (as a user)

```
1. Open EchoLink app (iOS, Android, or desktop)
2. Search for:  K0ABC-L   (your link node)
3. Connect
4. You are now on the air on K0USA 147.940 MHz
5. Wait for net control, then key up and give your call
```

---

# Participating in the NET — Operating Procedure

### Before the NET starts

- **Confirm the NET schedule** — day/time/format (voice check-in, traffic, etc.)
- **Confirm your audio levels** — transmit a few seconds, listen back
- **Announce your EchoLink node** to the net control trustee in advance

### During the NET

```
Net Control: "QRZ for check-ins..."

You (via EchoLink):
  → Key up (push-to-talk on your client)
  → "[Your callsign], checking in via EchoLink, 44Net node"
  → Unkey

Net Control: "K0ABC via EchoLink, copy 5/9, thank you"
```

### Good EchoLink operating practice
- Add **0.5–1 second** after keying before speaking (propagation delay)
- Keep transmissions **short** — others may have high latency
- Announce **"EchoLink"** when identifying so local ops know

---

# Windows 11 Pro — Alternative Path

### When you want a quick setup without configuring Linux

### Step 1 — Download EchoLink for Windows

```
https://www.echolink.org/download.htm
→ Download EchoLink Setup (latest version)
→ Run the installer
```

### Step 2 — First launch validation

```
1. Enter your callsign → Next
2. EchoLink checks the FCC database automatically
3. If not found, click "Request Validation" → email process
4. Enter the password from your validation email
```

### Step 3 — Configure audio device

```
EchoLink → Tools → Setup → Audio tab
  Soundcard Input:  [Your USB audio adapter]
  Soundcard Output: [Your USB audio adapter]
  Test both with the built-in test tones
```

---

# Windows 11 Pro — PTT and Radio Interface

### Step 4 — Configure PTT

```
EchoLink → Tools → Setup → PTT tab
  PTT method: VOX  (easiest, no serial needed)
    OR
  PTT method: Serial Port
    Port: COM3  (check Device Manager for your USB-serial port)
    Signal: RTS
```

### Step 5 — Sysop Mode (Link Node)

```
EchoLink → Tools → Sysop Setup
  Node type: Link (-L)
  Callsign: K0ABC-L
  Password: [your EchoLink password]
  Location: Denver CO
  Enable "Receive-only when not connected"
```

### Step 6 — Port forwarding on your Windows router

```
Router admin page → Port Forwarding:
  UDP 5198 → [Windows PC IP]
  UDP 5199 → [Windows PC IP]
  TCP 5200 → outbound (usually open by default)
```

---

# Windows 11 Pro — Running as a Service

### Keep EchoLink running even when not logged in

### Option A — Startup folder (simple)

```powershell
# Press Win+R → shell:startup
# Copy EchoLink shortcut into that folder
# EchoLink starts on login but not before
```

### Option B — Task Scheduler (better)

```
Task Scheduler → Create Task
  General: Run whether user is logged on or not
           Run with highest privileges
  Trigger: At system startup
  Action:  Start program → C:\Program Files\EchoLink\EchoLink.exe
  Settings: If already running, do not start new instance
```

### Option C — NSSM (Non-Sucking Service Manager)

```powershell
# Download nssm.cc
nssm install EchoLink "C:\Program Files\EchoLink\EchoLink.exe"
nssm start EchoLink
# Now it's a proper Windows service
```

---

# Putting It All Together on the Banana Pi

### Running SVXLink directly on the OpenWRT-One

### Step 1 — Install required packages via opkg

```bash
# SSH into your Banana Pi (default: ssh root@192.168.1.1)
opkg update
opkg install svxlink kmod-usb-audio alsa-utils \
             kmod-usb-serial-cp210x kmod-usb-serial
```

> SVXLink may need to be compiled from source if not in opkg.
> Alternative: run a **Docker container** or **LXC** with Debian inside OpenWrt.

### Step 2 — Verify USB audio on OpenWrt

```bash
aplay -l     # list audio devices
arecord -l   # list capture devices
```

### Step 3 — Use the same config files

The `svxlink.conf` and `ModuleEchoLink.conf` from earlier work unchanged.
Copy them to `/etc/svxlink/` on the Banana Pi.

---

# Banana Pi — 44Net Gateway + EchoLink Together

### The full architecture on one board

```
[Internet WAN — 2.5 GbE port]
         │
         │  WireGuard tunnel to 44net.connect.cloud
         │  Your 44.x.x.x IP lives here
         │
[Banana Pi OpenWRT-One]
         │
         ├── WiFi AP — local ham shack network
         │
         ├── USB audio adapter ──► Radio audio I/O
         │
         ├── USB-serial adapter ──► Radio PTT
         │
         └── SVXLink process
               │  EchoLink module active
               └──► connects to K0USA .94 via EchoLink
```

### Why 44Net on the same box?
- Your SVXLink node gets a **globally routable ham IP**
- Other 44Net nodes can reach you directly (no NAT issues)
- Useful for future AMPR services: Winlink, APRS, D-STAR gateways

---

# Banana Pi — OpenWrt Firewall for EchoLink

### Allowing EchoLink traffic through OpenWrt's firewall

```bash
# SSH into Banana Pi
# Allow EchoLink UDP ports in from WAN
uci batch << 'EOF'
add firewall rule
set firewall.@rule[-1].name='EchoLink-5198'
set firewall.@rule[-1].src='wan'
set firewall.@rule[-1].dest='lan'
set firewall.@rule[-1].proto='udp'
set firewall.@rule[-1].dest_port='5198-5199'
set firewall.@rule[-1].target='ACCEPT'
EOF

uci commit firewall
/etc/init.d/firewall restart
```

### If SVXLink runs ON the Banana Pi (not a separate machine):

```bash
# Allow traffic destined for the router itself
uci set firewall.@rule[-1].dest='*'
uci set firewall.@rule[-1].target='ACCEPT'
uci commit firewall && /etc/init.d/firewall restart
```

---

# Troubleshooting — Common Issues

| Symptom | Likely cause | Fix |
|---|---|---|
| SVXLink won't log in | Wrong password / callsign | Re-check `ModuleEchoLink.conf` |
| "Node not found" | EchoLink not validated | Complete echolink.org validation |
| Audio one-way | Wrong ALSA device | Run `aplay/arecord -l`, fix `AUDIO_DEV=` |
| Radio never keys | PTT pin wrong | Test PTT with Python script (slide 14) |
| Connections drop instantly | Ports not forwarded | Check UDP 5198/5199 with `nmap` |
| High latency / choppy | ISP throttling UDP | Try different EchoLink server |
| 44Net tunnel down | DNS not registered | Add DNS entry in AMPR portal |

### Useful diagnostic commands

```bash
# Test EchoLink port reachability from outside
curl -s https://echolink.org/looptest.htm

# Check which process owns port 5198
ss -ulnp | grep 5198

# SVXLink live log
journalctl -fu svxlink --since "5 min ago"
```

---

# Security Considerations

### Running a ham radio node on the internet

- **EchoLink validates licenses** — only hams can connect
- Use **CALLSIGN_ACL** in ModuleEchoLink.conf to restrict to known calls
- The `44.0.0.0/8` space is **not encrypted by default** — don't pass sensitive data
- Keep your Banana Pi **updated**: `opkg update && opkg upgrade`
- Use **WPA3** on the Banana Pi's WiFi AP
- Consider **fail2ban** if running SSH on a public IP
- **Never share your EchoLink password** — it's your license identity

### Regulatory reminders
- **No encryption** of voice content on amateur frequencies
- **No third-party traffic** for hire
- **Identify every 10 minutes** and at end of transmission (FCC Part 97)
- SVXLink handles automatic ID for you if configured correctly

---

# Resources & Links

### Hardware
- **Banana Pi OpenWRT-One docs:** https://docs.banana-pi.org/en/OpenWRT-One/BananaPi_OpenWRT-One
- **OpenWrt on BPI-One:** https://openwrt.org/toh/banana_pi/banana_pi_bpi-r2_mini

### 44Net / AMPRNet
- **AMPR Portal (registration):** https://portal.ampr.org
- **44Net Wiki Quickstart:** https://wiki.ampr.org/wiki/Quickstart
- **44Net Connect (WireGuard):** https://connect.44net.cloud
- **ARDC 44Net page:** https://www.ardc.net/44net/

### EchoLink
- **EchoLink official site:** https://www.echolink.org
- **Callsign validation:** https://www.echolink.org/validation/
- **SVXLink project:** https://www.svxlink.org
- **SVXLink GitHub:** https://github.com/sm0svx/svxlink

### Repeaters
- **K0USA .94 info:** https://www.repeaterbook.com (search K0USA, 147.940)
- **Colorado EchoLink repeaters:** https://www.repeaterbook.com/repeaters/feature_search.php?state_id=08

---

<!-- _paginate: false -->
<!-- _backgroundColor: #0d2a3a -->

# Q & A

## Thank you!

---

**Node information:**
- EchoLink node: `[Your callsign]-L`
- 44Net IP: `44.x.x.x` (after registration)
- Linked to: **K0USA 147.940 MHz** — Denver area

**NET schedule:**
- Ask K0USA trustees or check: https://www.repeaterbook.com

---

*Slides built with [MARP](https://marp.app) — open source presentation framework*
*73 de [Your Call Sign]*
