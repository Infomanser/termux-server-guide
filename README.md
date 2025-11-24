**How to turn an old smartphone into a full-fledged server for bots (Python, AI, Tunneling).**

[![Ukrainian](https://img.shields.io/badge/Lang-Ukrainian-blue)](README.ua.md)

This guide describes how to configure **Termux** on Android to host Telegram bots, web apps, or scripts using professional tools (PM2, Cloudflare, Logrotate). No VPS required, just your phone.

---

## 🚀 Chapter 1: Termux Preparation

Transforming your phone into a Linux terminal.

**1. Base Installation:**

```bash
pkg update && pkg upgrade
pkg install python nodejs git openssh
```

**Immortality (Wakelock):**
To prevent Android from "killing" background processes, you must disable CPU sleep. 
* Click `Acquire Wakelock` in the Termux notification shade.
* Or run the command:

```bash
termux-wake-lock
```

---

## 🚪 Chapter 2: SSH & Remote Access

How to manage your phone from a laptop over the internet (bypassing NAT without a static IP).

### 1. SSH Server On the phone:

```bash
# Задати пароль (обов'язково!)
passwd

# Запустити сервер (порт 8022)
sshd

# Дізнатися свій логін
whoami
```

**⚠️ Note:** SSH in Termux runs on port **8022**.

### 2. Cloudflare Tunnel (cloudflared)

Tunneling to the outside world.

**Installation (ARM64):**

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
mv cloudflared-linux-arm64 $PREFIX/bin/cloudflared
chmod +x $PREFIX/bin/cloudflared
``` 

**Manual Start (Test):**

```bash
cloudflared tunnel run --token <YOUR_TOKEN>
```

**Auto-start via PM2:**
*⚠️ Important: Use full path to avoid PM2 errors:*

```bash
pm2 start /data/data/com.termux/files/usr/bin/cloudflared --name "phone" -- tunnel run --token <YOUR_TOKEN>
```

**DNS Fix** (if you get "connection refused" error):
Open `nano $PREFIX/etc/resolv.conf` and write:

```text
nameserver 1.1.1.1
```

### 3. Client Connection (PC/Mac)

Configuration file: `~/.ssh/config`:

```text
Host phone
  HostName phone.your-domain.com
  User u0_a... (your phone username)
  ProxyCommand cloudflared access ssh --hostname %h
```

Connect:
```bash
ssh phone
```

---

## ⚙️ Chapter 3: PM2 (Process Manager)

Running scripts as services (auto-restart, logs, background).

**1. Installation:**

```bash
npm install pm2 -g
```

**2. Basic Commands:**

* `pm2 start bot.py --interpreter python --name "my-bot"` — start Python bot.
* `pm2 list` — status of all processes.
* `pm2 logs` — view logs.
* `pm2 save` — **save process list** (to restore after reboot).
* `pm2 resurrect` — restore saved processes.

---

## 🧹 Chapter 4: Automatic Cleaning (Logrotate)

Prevent logs from consuming all phone storage.

**1. Install Module:**

```bash
pm2 install pm2-logrotate
```

**2. Optimal Config for Android:**

```bash
# Max file size limit (10 MB)
pm2 set pm2-logrotate:max_size 10M

# Check frequency: Mondays and Thursdays (1,4). Evenly splits the week.
pm2 set pm2-logrotate:rotateInterval '0 0 * * 1,4'

# Keep only last 3 archives
pm2 set pm2-logrotate:retain 3
pm2 set pm2-logrotate:compress true
```

**3. Save Settings:**

```bash
pm2 save
```

---

## 🤖 Bonus: .env Template for Bots

**Never store tokens in your code! Use environment variables.**

File `.env`:

```ini
BOT_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11
GROQ_API_KEY=gsk_...
ADMIN_ID=123456789
```

Python Code:

```python
import os
from dotenv import load_dotenv

load_dotenv()
token = os.getenv("BOT_TOKEN")
```
