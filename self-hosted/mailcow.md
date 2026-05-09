# Mailcow

Mailcow is the recommended self-hosted mail server for most DevOps use cases. It runs entirely in Docker, bundles everything you need (Postfix, Dovecot, SOGo, Rspamd, ClamAV, Let's Encrypt), and provides a clean web UI for administration.

---

# 1️⃣ Prerequisites

- Ubuntu 22.04/24.04 or Debian 12 VPS
- Minimum 4 GB RAM, 20 GB disk (8 GB RAM recommended for ClamAV)
- Static IP with port 25 open
- PTR record set on your IP
- Docker + Docker Compose installed
- Domain with DNS access

---

# 2️⃣ System Requirements Check

```bash
# Verify port 25 is open
telnet gmail-smtp-in.l.google.com 25

# Verify Docker
docker --version
docker compose version
```

---

# 3️⃣ Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

---

# 4️⃣ Clone Mailcow

```bash
cd /opt
sudo git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
```

---

# 5️⃣ Generate Configuration

```bash
sudo ./generate_config.sh
```

When prompted:
- **Hostname**: `mail.yourdomain.com` (must match your A record)
- **Timezone**: your local timezone

This creates `mailcow.conf` and `docker-compose.yml`.

---

# 6️⃣ Configure mailcow.conf (Optional Tweaks)

```bash
sudo nano mailcow.conf
```

Key settings:

```ini
# Disable ClamAV if RAM < 4GB
SKIP_CLAMD=y

# HTTP/HTTPS ports (change if 80/443 are taken by Nginx)
HTTP_PORT=80
HTTPS_PORT=443

# Skip Let's Encrypt if using external cert
SKIP_LETS_ENCRYPT=n
```

---

# 7️⃣ Start Mailcow

```bash
sudo docker compose pull
sudo docker compose up -d
```

First start downloads images and initializes — takes 5–10 minutes.

Monitor:

```bash
sudo docker compose logs -f
```

---

# 8️⃣ Configure DNS

```dns
# A record for mail hostname
A      mail.yourdomain.com  →  <your-ip>

# MX record
MX     10  mail.yourdomain.com

# SPF
TXT    @   "v=spf1 mx ~all"

# DMARC
TXT    _dmarc  "v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com"
```

DKIM is generated automatically — get it from the admin UI after setup.

---

# 9️⃣ Access Admin UI

```
https://mail.yourdomain.com
```

Default credentials:
- Username: `admin`
- Password: `moohoo`

**Change this immediately.**

---

# 🔟 First Steps in Admin UI

1. **Configuration → Mail Setup → Domains** → Add your domain
2. **Configuration → Mail Setup → Mailboxes** → Create mailboxes
3. **Configuration → Configuration → ARC/DKIM Keys** → Copy DKIM record → add to DNS
4. **E-Mail → Configuration → Spam Filter** → tune Rspamd thresholds

---

# 1️⃣1️⃣ Verify DKIM

After adding DKIM DNS record:

```bash
dig TXT mail._domainkey.yourdomain.com
```

Or use: https://mxtoolbox.com/dkim.aspx

---

# 1️⃣2️⃣ Maintenance Commands

```bash
# Stop
sudo docker compose down

# Start
sudo docker compose up -d

# Update Mailcow
sudo ./update.sh

# View mail queue
sudo docker compose exec postfix-mailcow postqueue -p

# Flush mail queue
sudo docker compose exec postfix-mailcow postqueue -f

# Check logs
sudo docker compose logs postfix-mailcow -f
sudo docker compose logs dovecot-mailcow -f
sudo docker compose logs rspamd-mailcow -f
```

---

# 1️⃣3️⃣ Backup

```bash
# Built-in backup script
sudo ./helper-scripts/backup_and_restore.sh backup all
```

Backs up: mailboxes, configuration, databases. Store backups off-server (S3, rclone, etc.).

---

# 1️⃣4️⃣ Nginx Reverse Proxy (If Port 80/443 Occupied)

If you already run Nginx on the host, set `HTTP_PORT=8080` / `HTTPS_PORT=8443` in `mailcow.conf`, then proxy from Nginx:

```nginx
server {
    listen 443 ssl;
    server_name mail.yourdomain.com;

    location / {
        proxy_pass https://127.0.0.1:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

# 🧠 Memorization Trick

Mailcow components = **P D R C S**
- **P** = Postfix (SMTP)
- **D** = Dovecot (IMAP)
- **R** = Rspamd (spam filter)
- **C** = ClamAV (antivirus)
- **S** = SOGo (webmail + calendar)

---

# 🔥 Common Problems

### Emails not arriving (inbound)
→ MX record wrong or not propagated. Check: `dig MX yourdomain.com`

### Emails going to spam
→ PTR record not set, DKIM not in DNS, or IP is blacklisted

### Let's Encrypt failing
→ Port 80 must be open and pointed to the server. Check firewall.

### Out of memory
→ ClamAV is the biggest consumer. Set `SKIP_CLAMD=y` if RAM < 4 GB.

### Admin password forgotten
```bash
sudo docker compose exec mysql-mailcow mysql -u${DBUSER} -p${DBPASS} ${DBNAME} \
  -e "UPDATE admin SET password = '{SHA512-CRYPT}$(python3 -c "import crypt; print(crypt.crypt('newpassword', crypt.mksalt(crypt.METHOD_SHA512)))")' WHERE username = 'admin';"
```
