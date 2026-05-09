# iRedMail

iRedMail is a shell script that installs and configures a full mail server stack on a fresh Linux system. No Docker required. It sets up Postfix, Dovecot, Roundcube (webmail), Fail2ban, and ClamAV in one run.

---

# 1️⃣ Prerequisites

- Fresh Ubuntu 22.04 or Debian 12 (iRedMail must be installed on a clean system)
- Minimum 4 GB RAM
- Static IP, port 25 open, PTR record set
- FQDN hostname configured before install

---

# 2️⃣ Set Hostname (Required Before Install)

```bash
sudo hostnamectl set-hostname mail.yourdomain.com

# Add to /etc/hosts
sudo nano /etc/hosts
```

Add:

```
127.0.0.1  mail.yourdomain.com  mail
```

Verify:

```bash
hostname -f
# Should output: mail.yourdomain.com
```

---

# 3️⃣ Update System

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

# 4️⃣ Download iRedMail

Check latest version at: https://www.iredmail.org/download.html

```bash
cd /root
wget https://github.com/iredmail/iRedMail/archive/refs/tags/1.7.0.tar.gz
tar xzf 1.7.0.tar.gz
cd iRedMail-1.7.0
chmod +x iRedMail.sh
```

---

# 5️⃣ Run Installer

```bash
sudo bash iRedMail.sh
```

The TUI (text UI) will ask:

| Question | Recommended Answer |
|---|---|
| Mail storage directory | `/var/vmail` (default) |
| Web server | `Nginx` |
| Backend (user storage) | `MariaDB` or `OpenLDAP` |
| MySQL root password | Set a strong password |
| Mail domain | `yourdomain.com` |
| Admin password | Set a strong password |
| Optional components | Roundcube ✓, Fail2ban ✓, ClamAV ✓ |

When complete → **reboot** as instructed.

---

# 6️⃣ Configure DNS

```dns
A      mail.yourdomain.com  →  <your-ip>
MX     10  mail.yourdomain.com
TXT    @   "v=spf1 mx ~all"
TXT    _dmarc  "v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com"
```

DKIM key is generated during install:

```bash
sudo cat /etc/opendkim/keys/yourdomain.com/default.txt
```

Add this as a TXT record for `default._domainkey.yourdomain.com`.

---

# 7️⃣ Access Interfaces

| Interface | URL | Credentials |
|---|---|---|
| Roundcube (webmail) | `https://mail.yourdomain.com/mail` | mailbox email + password |
| iRedAdmin (admin panel) | `https://mail.yourdomain.com/iredadmin` | `postmaster@yourdomain.com` + password |
| SOGo (calendar, if enabled) | `https://mail.yourdomain.com/SOGo` | same |

---

# 8️⃣ Create Mailboxes

Via iRedAdmin web UI:
- Domains → Add mailbox
- Or via CLI:

```bash
# iRedMail uses iRedAdmin-Pro CLI (open source: manage via web UI)
```

---

# 9️⃣ Useful Commands

```bash
# Mail queue
postqueue -p

# Flush queue
postqueue -f

# Check Postfix logs
tail -f /var/log/mail.log

# Check Dovecot logs
tail -f /var/log/dovecot.log

# Restart all services
sudo systemctl restart postfix dovecot nginx

# Check Fail2ban bans
sudo fail2ban-client status
sudo fail2ban-client status postfix
```

---

# 🔟 SSL Certificate

iRedMail uses a self-signed cert by default. Replace with Let's Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d mail.yourdomain.com
```

Then update Postfix and Dovecot to use the new cert:

```bash
# /etc/postfix/main.cf
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem
smtpd_tls_key_file  = /etc/letsencrypt/live/mail.yourdomain.com/privkey.pem

# /etc/dovecot/conf.d/10-ssl.conf
ssl_cert = </etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem
ssl_key  = </etc/letsencrypt/live/mail.yourdomain.com/privkey.pem

sudo systemctl restart postfix dovecot
```

---

# 1️⃣1️⃣ Upgrade iRedMail

iRedMail releases upgrade scripts per version:

```bash
# Follow the official upgrade guide for your version:
# https://docs.iredmail.org/iredmail.releases.html
```

Do not run the installer again — always use the version-specific upgrade script.

---

# 🧠 Memorization Trick

iRedMail = **One script, real packages**

Unlike Mailcow (Docker), iRedMail installs packages directly on the OS. This means:
- Easier to debug (standard log locations)
- Harder to update (version-locked packages)
- No containerization overhead

---

# 🔥 Common Problems

### Installer fails partway through
→ iRedMail must run on a **fresh** system. Any prior mail server config will cause conflicts.

### DKIM not working
→ Verify DNS: `dig TXT default._domainkey.yourdomain.com` — key must match what's in `/etc/opendkim/keys/`

### Webmail redirects to self-signed cert warning
→ Replace with Let's Encrypt cert (see step 10)

### Fail2ban blocking your own IP
```bash
sudo fail2ban-client set postfix unbanip <your-ip>
```

### Emails in spam
→ PTR record missing or IP blacklisted. Check: https://mxtoolbox.com/blacklists.aspx
