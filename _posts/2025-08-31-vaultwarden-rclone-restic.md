---
title: "Your Passwords, Your Hardware: Pi 5 + Docker + Encrypted Backups"
description: "Step-by-step: Pi 5 + Docker install, NTP/TZ time sync, desktop-first account to avoid iPhone JWT issues, Argon2 admin token, and rclone+restic nightly encrypted backups to Google Drive—plus LAN-only firewall allow-listing."
date: 2025-08-31
categories: [organization-data, productivity-process]
tags: [self-hosting, homelab, security, backups, encryption, vaultwarden, bitwarden, raspberry-pi, docker, compose, rclone, restic, google-drive, ntp, firewall]
published: true
---------------

Why rent your password manager when you can **own** it? With a Raspberry Pi 5 and Docker, you can run **Vaultwarden** (the lightweight, open-source Bitwarden-compatible server) at home, keep your vault on **your hardware**, and sync it across **iPhone/iPad/Mac (Apple), Windows, Android, and Linux** using the official Bitwarden apps and browser extensions—**without paying monthly fees** for someone else’s cloud.

This guide is a copy-pasteable path to a private, cross-platform setup: you’ll install Vaultwarden on a Pi 5 with Docker, create your first account using the **Bitwarden Desktop app** (a reliable first-run that avoids the occasional iPhone JWT hiccup), and add **nightly, client-side–encrypted backups** to Google Drive via **rclone + restic** so your secrets are safe even if a disk fails. We’ll also lock the service to your local network and make sure **time is NTP-synced** with the same timezone in both the OS and Docker—small details that make everything “just work.”

**Why self-host?**

* **Ownership & privacy:** your passwords live on **your Pi**, not a third-party server.
* **Lower attack surface:** the vault is **LAN-only** by default; only devices you allow can reach it.
* **Resilience without trust:** off-site backups are **end-to-end encrypted** before they ever touch the cloud provider.
* **No subscriptions:** one tiny computer, standard Docker, and the free Bitwarden apps = a secure, multi-device manager with **zero monthly fees**.

---

## What you’ll set up

* Raspberry Pi OS Lite (64-bit) on a Pi 5
* Docker + Docker Compose
* Vaultwarden (Bitwarden-compatible)
* **Correct time**: NTP on and **same TZ** in OS and docker-compose
* First account creation via **Bitwarden Desktop app** (avoids iPhone JWT hiccups)
* Optional admin UI protected by an **Argon2 PHC hash**
* **UFW + iptables allow-list** so only your devices can reach port 8123
* **Nightly encrypted backups** to Google Drive (rclone + restic)

---

## 0) Prep the Pi (updates + time)

```bash
# Update OS
sudo apt update && sudo apt full-upgrade -y

# Correct, synced time (critical for tokens)
sudo timedatectl set-ntp true
sudo timedatectl set-timezone America/New_York   # use your local TZ
timedatectl status | egrep 'Time zone|synchronized'

# Good SSD hygiene
sudo systemctl enable --now fstrim.timer
```

> **Why time matters:** registration tokens are time-based. A wrong clock or mismatched time zone is a common source of iPhone “Error decoding JWT.” Keep OS TZ and compose TZ the **same**.

---

## 1) Install Docker + Compose

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
sudo apt install -y docker-compose-plugin
```

---

## 2) Create Vaultwarden folders & Compose

```bash
sudo mkdir -p /srv/vaultwarden
sudo chown -R $USER:$USER /srv/vaultwarden
```

Create `/srv/vaultwarden/docker-compose.yml`:

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    environment:
      - TZ=America/New_York            # match the Pi's timezone
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=true           # enable only for the initial account
      - SIGNUPS_VERIFY=false           # skip email verification (no SMTP needed)
      - DOMAIN=http://<PI-IP>:8123     # exact origin you'll use (IP is simplest)
      # - ADMIN_TOKEN=<we'll add a hashed token later>  # optional admin UI
    volumes:
      - /srv/vaultwarden:/data
    ports:
      - "8123:80"          # LAN-only; do NOT port-forward on your router
    restart: unless-stopped
```

Start it:

```bash
cd /srv/vaultwarden
docker compose up -d
docker logs -f vaultwarden   # wait for: "Rocket has launched from http://0.0.0.0:80"
```

---

## 3) Create the first account **with the Bitwarden Desktop app**

Some mobile flows (especially on iPhone) can hit the “**Error decoding JWT**” loop if invites/verification or cached config don’t line up. The **desktop app** avoids that.

1. Install the **Bitwarden Desktop** app on your Linux/macOS/Windows computer.
2. In the app: **Settings → Server** → choose **Self-hosted** and set
   `http://<PI-IP>:8123`
3. Click **Create account**, set your email + master password, and sign in.
4. After your account works on desktop, add the **iPhone** app:
   Settings → Server → `http://<PI-IP>:8123` → log in (should now work cleanly).

Now **lock signups** back down:

```bash
# Edit compose: set SIGNUPS_ALLOWED=false
docker compose up -d
```

---

## 4) (Optional) Secure the Admin UI with an **Argon2 PHC** hash

Generate a hashed admin token inside the container and escape `$` for YAML:

```bash
docker exec -it vaultwarden /vaultwarden hash | sed 's#\$#\$\$#g'
```

Paste the output into compose:

```yaml
environment:
  ...
  - ADMIN_TOKEN=$$argon2id$$v=19$$m=...   # (note the $$ everywhere)
```

Apply:

```bash
docker compose up -d
```

Admin UI: `http://<PI-IP>:8123/admin`

---

## 5) Lock it to your network (UFW + Docker-aware allow-list)

**Goal:** Only your chosen devices can reach port **8123**.

```bash
sudo apt install -y ufw netfilter-persistent

# Default policy: deny inbound, allow outbound
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from your LAN only (change CIDR to your LAN)
LAN_CIDR=192.168.1.0/24
sudo ufw allow from $LAN_CIDR to any port 22 proto tcp

sudo ufw enable
sudo ufw status verbose
```

Docker publishes ports with its own rules; enforce your policy **before** Docker via the `DOCKER-USER` chain:

```bash
# Replace with your actual device IPs (DHCP-reserve them on your router)
ALLOWED_PHONE=192.168.1.50
ALLOWED_PC=192.168.1.51

sudo iptables -I DOCKER-USER -i eth0 -p tcp --dport 8123 -s $ALLOWED_PHONE -j ACCEPT
sudo iptables -I DOCKER-USER -i eth0 -p tcp --dport 8123 -s $ALLOWED_PC -j ACCEPT
sudo iptables -A DOCKER-USER -i eth0 -p tcp --dport 8123 -j DROP

sudo netfilter-persistent save
sudo iptables -S DOCKER-USER
```

> Extra belt-and-suspenders: in compose, bind Vaultwarden only to your LAN IP:
> `ports: ["<PI-IP>:8123:80"]` instead of `"8123:80"`.

---

## 6) Nightly, **encrypted** backups to Google Drive (rclone + restic)

We’ll use **rclone** (to talk to Drive) and **restic** (to encrypt/dedupe/retain). Everything is encrypted **before** upload with a **random 256-bit key** you control. Run rclone as **root** so the systemd job uses the same config.

### 6.1 Configure rclone (Google Drive)

```bash
sudo apt install -y rclone
sudo -H rclone config
```

* `n` → name: **gdrive**
* storage: **drive**
* **client\_id / client\_secret**:

  * **Quick**: leave both blank (uses rclone’s built-in key; fine for home backups), or
  * **Recommended**: create your own Google OAuth “Desktop app” client in Google Cloud Console and paste ID/secret (add your Google account as a **Test user** if the app is in Testing).
* scope: **drive.file** (least privilege)
* auto config: `y` (or `n` on headless; run `rclone authorize "drive"` on a desktop and paste the code)
* Save, then verify:

```bash
sudo -H rclone lsd gdrive:
sudo chmod 600 /root/.config/rclone/rclone.conf
```

*(If OAuth is a hassle, you can use a **Service Account** instead: share a Drive folder with the service account email and make a `gdrive-sa:` remote. The rest stays the same.)*

### 6.2 Install restic + generate a **random 256-bit key**

```bash
sudo apt install -y restic
sudo mkdir -p /root/backup-secrets
openssl rand -hex 32 | sudo tee /root/backup-secrets/restic-password >/dev/null
sudo chmod 600 /root/backup-secrets/restic-password
```

Initialize the repo on Drive (one time):

```bash
export RESTIC_REPOSITORY=rclone:gdrive:vaultwarden-restic
export RESTIC_PASSWORD_COMMAND='cat /root/backup-secrets/restic-password'
sudo -E restic init
```

### 6.3 Backup script (brief stop for a clean snapshot)

Create `/usr/local/bin/backup-vaultwarden.sh`:

```bash
sudo bash -c 'cat > /usr/local/bin/backup-vaultwarden.sh' <<'SH'
#!/usr/bin/env bash
set -euo pipefail
export RCLONE_CONFIG=/root/.config/rclone/rclone.conf
export RESTIC_REPOSITORY=rclone:gdrive:vaultwarden-restic
export RESTIC_PASSWORD_COMMAND='cat /root/backup-secrets/restic-password'

# Quiesce for consistent SQLite backup (fast)
docker stop vaultwarden

# Backup data dir (DB, attachments, config)
restic backup /srv/vaultwarden --tag vaultwarden

# Keep 7 daily, 5 weekly, 12 monthly; prune older
restic forget --tag vaultwarden --keep-daily 7 --keep-weekly 5 --keep-monthly 12 --prune

docker start vaultwarden
SH
sudo chmod +x /usr/local/bin/backup-vaultwarden.sh
```

Test it:

```bash
sudo -E /usr/local/bin/backup-vaultwarden.sh
sudo -E restic snapshots
```

> **Zero-downtime alternative:** copy the DB inside the container instead of stopping:
> `docker exec vaultwarden sh -lc 'apk add --no-cache sqlite || true; sqlite3 /data/db.sqlite3 ".backup /data/db-backup.sqlite3"'`
> Then back up `/srv/vaultwarden` (which now includes `db-backup.sqlite3`). The brief stop is simpler and safest.

### 6.4 Nightly schedule (systemd)

`/etc/systemd/system/restic-vaultwarden.service`:

```ini
[Unit]
Description=Vaultwarden encrypted backup (restic -> Google Drive)
After=network-online.target docker.service
Wants=network-online.target

[Service]
Type=oneshot
Environment=RCLONE_CONFIG=/root/.config/rclone/rclone.conf
Environment=RESTIC_REPOSITORY=rclone:gdrive:vaultwarden-restic
Environment=RESTIC_PASSWORD_COMMAND=cat\ /root/backup-secrets/restic-password
ExecStart=/usr/local/bin/backup-vaultwarden.sh
```

`/etc/systemd/system/restic-vaultwarden.timer`:

```ini
[Unit]
Description=Nightly Vaultwarden backup

[Timer]
OnCalendar=02:30
Persistent=true

[Install]
WantedBy=timers.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now restic-vaultwarden.timer
systemctl list-timers | grep restic-vaultwarden
```

**Restore drill (practice once):**

```bash
export RCLONE_CONFIG=/root/.config/rclone/rclone.conf
export RESTIC_REPOSITORY=rclone:gdrive:vaultwarden-restic
export RESTIC_PASSWORD_COMMAND='cat /root/backup-secrets/restic-password'

sudo -E restic snapshots
sudo -E restic restore latest --target /     # restores to /srv/vaultwarden/...
cd /srv/vaultwarden && docker compose up -d
```

> **Why this is safe:** restic encrypts everything locally (AES-256 with a random 256-bit key). Only ciphertext lands in Drive. Keep offline copies of the key file (`/root/backup-secrets/restic-password`)—printed or stored in another password manager—so you can always restore.

---

## 7) Final checklist

* `timedatectl status` → **synchronized: yes**, TZ matches compose.
* Bitwarden **Desktop** app: can log in to `http://<PI-IP>:8123`.
* iPhone app: set Server to the same URL and log in (JWT issues should be gone since the account already exists).
* `docker logs vaultwarden` shows normal requests.
* `sudo -E restic snapshots` shows your first snapshot; timer is scheduled.
* `sudo ufw status` is active; `sudo iptables -S DOCKER-USER` shows your allow-list and a final DROP for port 8123.

---

### Extra tips

* **Admin token hygiene:** store your Argon2 admin password in your password manager (it’s separate from your vault master password).
* **DHCP reservations:** give your phone and desktop fixed IPs so the allow-list doesn’t break.
* **Health check:** `sudo ss -tulpn | grep :8123` (listening), `docker ps --format 'table {{.Names}}\t{{.Status}}'`.

---

You now have a **private**, **resilient**, **encrypted** Vaultwarden on your Pi 5: passwords stored on your own hardware, backups secured with strong client-side crypto in your personal cloud, and access limited to just your devices.
