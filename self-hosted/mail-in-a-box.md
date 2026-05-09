# Mail-in-a-Box

Mail-in-a-Box (MIAB) is the simplest self-hosted option. A single script sets up a complete mail server with automatic DNS configuration guidance, TLS certificates, and a web UI. Designed for personal use and small setups.

---

# 1️⃣ Prerequisites

- **Fresh Ubuntu 22.04 only** (MIAB is strict about this — no other distro)
- Minimum 1 GB RAM (512 MB usable for tiny setups)
- Static IP, port 25 open
- PTR record set: `your.ip → box.yourdomain.com`
- You should have a **dedicated VPS** — MIAB takes full control of the server

---

# 2️⃣ Set Hostname

```bash
sudo hostnamectl set-hostname box.yourdomain.com
```

MIAB uses `box.yourdomain.com` as the standard hostname convention.

---

# 3️⃣ Install

```bash
curl -s https://mailinabox.email/setup.sh | sudo -E bash
```

The script will:
1. Install all packages (Postfix, Dovecot, Nginx, Roundcube, Z-Push, etc.)
2. Ask for your email address (becomes the admin account)
3. Confirm your domain
4. Generate TLS certs via Let's Encrypt
5. Start all services

Takes 15–30 minutes.

---

# 4️⃣ Post-Install: Configure DNS

After install, MIAB shows you exactly what DNS records to add:

```
https://box.yourdomain.com/admin
```

Go to: **System → External DNS**

You'll see a complete list of records to add at your registrar:
- A, AAAA records
- MX record
- SPF, DKIM, DMARC
- DNSSEC (optional)

**MIAB can also host its own DNS** if you delegate your domain's nameservers to it — but using your registrar's DNS is simpler.

---

# 5️⃣ Admin Panel

```
https://box.yourdomain.com/admin
```

Login: your admin email + password set during install.

From here:
- Add/remove mailboxes
- Add aliases
- Check system status (green = good)
- View DNS records to configure
- Manage backups

---

# 6️⃣ Webmail

```
https://box.yourdomain.com/mail
```

Roundcube webmail. Users log in with their full email and password.

---

# 7️⃣ CalDAV / CardDAV (Calendar + Contacts)

MIAB includes Nextcloud for calendar and contacts sync:

```
https://box.yourdomain.com/cloud
```

Configure in your phone/desktop calendar app:
- CalDAV: `https://box.yourdomain.com/cloud/remote.php/dav`

---

# 8️⃣ Mail Client Settings

| Setting | Value |
|---|---|
| IMAP server | `box.yourdomain.com` port 993 (SSL) |
| SMTP server | `box.yourdomain.com` port 587 (STARTTLS) |
| Username | full email address |
| Password | mailbox password |

---

# 9️⃣ System Status Check

Admin → System → Status Checks

MIAB shows a green/yellow/red status for every component. Fix all reds before considering setup complete.

---

# 🔟 Update MIAB

```bash
sudo mailinabox
```

Run this to apply updates. MIAB is updated regularly — check: https://mailinabox.email/

---

# 1️⃣1️⃣ Backup

Admin → System → Backup Status

MIAB backs up to a local directory by default. Configure S3 or rsync backup target:

Admin → System → Backup → set S3 credentials or rsync target.

Encrypted backups are enabled by default using a key stored in `/home/user-data/backup/secret_key.txt` — save this key separately.

---

# 1️⃣2️⃣ Useful Commands

```bash
# Re-run setup (safe to re-run)
sudo mailinabox

# Check mail queue
sudo postqueue -p

# View logs
sudo tail -f /var/log/mail.log
sudo tail -f /var/log/syslog

# List mailboxes via CLI
sqlite3 /home/user-data/mail/users.sqlite "SELECT email FROM users"
```

---

# 🧠 Memorization Trick

MIAB = **Maximum automation, minimum flexibility**

Think of it like a home router — plug it in, works out of the box, but you can't reconfigure the internals freely. Perfect for personal use, not for complex multi-domain enterprise setups.

---

# ⚠️ Limitations

- Ubuntu 22.04 **only** (strict)
- **Takes over the entire server** — don't install other services alongside it
- Not suitable for high-traffic or multi-tenant setups
- Nextcloud version is older (MIAB controls the version)
- Limited customization vs. Mailcow or manual Postfix

---

# 🔥 Common Problems

### Setup fails immediately
→ Must be a fresh Ubuntu 22.04. MIAB checks and aborts if previous install detected.

### Let's Encrypt fails
→ Port 80 must be open and the domain must resolve to your IP

### Status checks show red
→ Usually DNS propagation. Wait and re-check. Run `sudo mailinabox` to re-apply.

### Port 25 blocked
→ Contact your VPS provider to unblock. If they refuse, MIAB won't work.

### Forgot admin password
```bash
sudo mailinabox
# Re-enter your email when prompted — it will ask to reset password
```
