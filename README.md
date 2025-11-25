# 📱 Android Pocket Server Handbook

Turn an Android smartphone into a fully functional server for bots, tunneling, and automation.

> **Works on Android 8–14**
> Fully tested on Android 13 with Termux (F-Droid build).

---

# ⚠️ Important Notes

### ❗ Use ONLY Termux from F-Droid

The Google Play version:

* has broken permissions
* can't run long-living processes
* may kill SSH / PM2 / cloudflared
* lacks core patches

Download:
[https://f-droid.org/packages/com.termux/](https://f-droid.org/packages/com.termux/)

### ❗ Android 12–14 Warning

Some networking restrictions require using **proot wrapper** for cloudflared stability.
This README already includes the correct, tested configuration.

---

# 🚀 Chapter 1 — Prepare Termux

### 1. Base installation

```bash
pkg update && pkg upgrade
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
passwd              # set password
sshd                # start SSH on port 8022
whoami              # get username (example: u0_a123)
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
proot -0 cloudflared tunnel run --token <YOUR_TOKEN>
```

---

## Step 4 — PM2 Autostart (correct way)

```bash
npm install pm2 -g
pm2 start cloudflared --name tunnel -- \
  tunnel run --protocol http2 --token <YOUR_TOKEN>
pm2 save
```

⚠️ No termux-chroot, no proot inside PM2.
Let cloudflared decide internally.

---

## Step 5 — SSH from PC

Install cloudflared on your computer:
[https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/)

Login:

```bash
cloudflared access login phone.your-domain.com
```

Your SSH config (`~/.ssh/config`):

```text
Host phone
  HostName phone.your-domain.com
  User u0_a123
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

* `pm2 logs`
* `pm2 list`
* `pm2 restart mybot`
* `pm2 resurrect`

---

# 🧹 Chapter 5 — Logrotate (prevents storage overflow)

```bash
pm2 install pm2-logrotate

pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 3
pm2 set pm2-logrotate:rotateInterval '0 0 * * 1,4'
pm2 set pm2-logrotate:compress true

pm2 save
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

Now your phone works as a **portable server** with stable tunneling, secure SSH, autostarting bots, and log rotation.

```
Portable. Reliable. Zero Trust secured.
```
