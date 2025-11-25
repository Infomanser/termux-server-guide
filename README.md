# 📱 Android Pocket Server Handbook

**Turn your smartphone into a full-fledged server: Python bots, tunnels, SSH, autostart, Zero Trust.**

[![Ukrainian](https://img.shields.io/badge/Lang-Ukrainian-blue)](README.ua.md)

This guide describes how to configure **Termux** on Android to host Telegram bots, web apps, or scripts using professional tools (PM2, Cloudflare, Logrotate). No VPS required, just your phone.

> **Tested on Android 13 (Termux from F-Droid).**

---

# ⚠️ Important Notes

### ❗ Use ONLY Termux from F-Droid

Link:
[https://f-droid.org/packages/com.termux/](https://f-droid.org/packages/com.termux/)

The Google Play version is deprecated and:
* Has restricted permissions.
* Subject to phantom process killing.
* Breaks SSH and cloudflared functionality.
* Is NOT suitable for server use.

### ❗ Android 12–14 Restrictions

`cloudflared` might crash due to system restrictions. To fix stability and DNS issues, use **proot-wrapper** (included in this guide).

---

# 🚀 Chapter 1: Termux Preparation

### Base Installation

```bash
# 1. Update packages
pkg update && pkg upgrade

# 2. Install Python, Node.js, Git, SSH server, Nano, Proot
pkg install python nodejs git openssh nano proot
```
### 2. Enable Wakelock (prevents Android from killing your server)

**Option A:**
Swipe down → find Termux notification → tap **Acquire Wakelock**

**Option B:**

```bash
termux-wake-lock
```

### 3. Battery optimization (mandatory)

Open:
**Settings → Apps → Termux → Battery → Allow background activity + Disable battery optimizations**

---

# 🚪 Chapter 2 — SSH Server (with Zero Trust)

This setup gives you stable SSH access **from anywhere**, even behind CG-NAT or mobile internet.

## Step 1 — Configure SSH on Termux

```bash
# Set password for SSH access
passwd

# Start SSH server (port 8022)
sshd

# Get your username (e.g., u0_a123)
whoami
```

⚠️ Termux SSH always uses **port 8022**.

---

# 🌐 Chapter 3 — Cloudflare Zero Trust SSH

This is the recommended and stable method for remote access.

## Step 1 — Install cloudflared (with proot for Android 12–14)

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
chmod +x cloudflared-linux-arm64
mv cloudflared-linux-arm64 $PREFIX/bin/cloudflared
```

If cloudflared fails on your device (Android 12–14), use **proot wrapper**:

```bash
proot -0 cloudflared tunnel run --token <YOUR_TOKEN>
```

Works 100% reliably.

---

## Step 2 — Configure Tunnel in Zero Trust Dashboard

1. Open Cloudflare → Zero Trust → Access → Tunnels
2. Create new tunnel
3. Add Public Hostname
4. Choose:

   * **Service**: `ssh://localhost:8022`
   * **Authentication**: your email / OTP / GitHub login
5. Copy your **Tunnel Token**

---

## Step 3 — Start tunnel (manual test)

```bash
cloudflared tunnel run --token <YOUR_TOKEN>
```

or if Android 12–14 gives issues:

```bash
# Stable method using termux-chroot
termux-chroot cloudflared tunnel run --token <YOUR_TOKEN>
```

---

## Step 4 — PM2 Autostart (Stable Method)

```bash
npm install pm2 -g

# We use termux-chroot to fix DNS resolution issues
pm2 start termux-chroot --name tunnel -- cloudflared tunnel run --protocol http2 --token <YOUR_TOKEN>

pm2 save
```

---

## Step 5 — SSH from PC

Install cloudflared on your computer:
[https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/)

Login:

```bash
# Login to Zero Trust
cloudflared access login phone.your-domain.com
```

Your SSH config (`~/.ssh/config`):

```text
Host phone
  HostName phone.your-domain.com
  User u0_a...  # Your phone username (from whoami)
  ProxyCommand cloudflared access ssh --hostname %h
```

Connect:

```bash
ssh phone
```

---

# ⚙️ Chapter 4 — PM2 for Bots

### Install:

```bash
npm install pm2 -g
```

### Run Python bot:

```bash
pm2 start bot.py --interpreter python --name mybot
pm2 save
```

### Useful commands:

* `pm2 logs` — view logs
* `pm2 list` — status of all processes
* `pm2 restart mybot` — restart specific process
* `pm2 resurrect` — restore saved processes after reboot

---

# 🧹 Chapter 5 — Logrotate (prevents storage overflow)

```bash
# Install module
pm2 install pm2-logrotate

# Limit file size to 10MB
pm2 set pm2-logrotate:max_size 10M

# Rotate every Monday and Thursday (1,4) to balance load
pm2 set pm2-logrotate:rotateInterval '0 0 * * 1,4'

# Keep only last 3 archives
pm2 set pm2-logrotate:retain 3
pm2 set pm2-logrotate:compress true
```

---

# 🔐 Chapter 6 — .env Template

```
BOT_TOKEN=123
GROQ_API_KEY=gsk_...
ADMIN_ID=123456
```

Python:

```python
from dotenv import load_dotenv
import os
load_dotenv()
token = os.getenv("BOT_TOKEN")
```

---

# ✅ Done

Your smartphone is now a pocket-sized server with Zero Trust security, stable SSH tunneling, logging, and auto-recovery.

```
Portable. Reliable. Zero Trust secured.
```
