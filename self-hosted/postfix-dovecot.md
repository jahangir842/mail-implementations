# Postfix + Dovecot (Manual Stack)

Building a mail server manually from Postfix and Dovecot gives you the deepest understanding of how email works. Every packaged solution (Mailcow, iRedMail) is built on this stack underneath. Start here if you want to learn, or need maximum control.

---

# Architecture Overview

```
Internet → [Port 25] → Postfix (MTA) → Maildir on disk
Client   → [Port 587] → Postfix (submission) → deliver
Client   → [Port 993] → Dovecot (IMAP) → reads Maildir
                      → [Port 995] → Dovecot (POP3)
Spam     → SpamAssassin / Rspamd (optional, plugs into Postfix)
DKIM     → OpenDKIM (plugs into Postfix milter)
```

---

# 1️⃣ Prerequisites

- Ubuntu 22.04 VPS
- Static IP, port 25 open, PTR record set
- Domain with DNS access
- Root or sudo access

---

# 2️⃣ Install Packages

```bash
sudo apt update
sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d \
  dovecot-lmtpd opendkim opendkim-tools spamassassin spamc -y
```

During Postfix install:
- Mail configuration type: **Internet Site**
- System mail name: `yourdomain.com`

---

# 3️⃣ Configure Postfix

### `/etc/postfix/main.cf`

```ini
# Basic identity
myhostname = mail.yourdomain.com
mydomain = yourdomain.com
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4

# Domains to receive mail for
mydestination = $myhostname, localhost.$mydomain, localhost
virtual_mailbox_domains = yourdomain.com
virtual_mailbox_base = /var/mail/vhosts
virtual_mailbox_maps = hash:/etc/postfix/vmailbox
virtual_minimum_uid = 100
virtual_uid_maps = static:5000
virtual_gid_maps = static:5000

# TLS (inbound)
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem
smtpd_tls_key_file  = /etc/letsencrypt/live/mail.yourdomain.com/privkey.pem
smtpd_use_tls = yes
smtpd_tls_security_level = may

# TLS (outbound)
smtp_tls_security_level = may

# DKIM milter
milter_default_action = accept
smtpd_milters = inet:localhost:12301
non_smtpd_milters = $smtpd_milters

# SASL (client auth for submission)
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
```

### `/etc/postfix/vmailbox` (virtual mailboxes)

```
user1@yourdomain.com  yourdomain.com/user1/
user2@yourdomain.com  yourdomain.com/user2/
```

```bash
sudo postmap /etc/postfix/vmailbox
```

### `/etc/postfix/master.cf` — Enable submission port

Uncomment or add:

```
submission inet n - y - - smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
```

---

# 4️⃣ Create Mail Storage

```bash
sudo groupadd -g 5000 vmail
sudo useradd -g vmail -u 5000 vmail -d /var/mail/vhosts -m
sudo mkdir -p /var/mail/vhosts/yourdomain.com
sudo chown -R vmail:vmail /var/mail/vhosts
```

---

# 5️⃣ Configure Dovecot

### `/etc/dovecot/conf.d/10-mail.conf`

```ini
mail_location = maildir:/var/mail/vhosts/%d/%n
mail_privileged_group = mail
```

### `/etc/dovecot/conf.d/10-auth.conf`

```ini
disable_plaintext_auth = yes
auth_mechanisms = plain login
```

### `/etc/dovecot/conf.d/auth-passwdfile.conf.ext`

```ini
passdb {
  driver = passwd-file
  args = scheme=SHA512-CRYPT /etc/dovecot/users
}
userdb {
  driver = static
  args = uid=vmail gid=vmail home=/var/mail/vhosts/%d/%n
}
```

### `/etc/dovecot/users` (virtual users)

```bash
# Generate password hash
doveadm pw -s SHA512-CRYPT
# Enter password → copies hash

# Format: user@domain:{SCHEME}hash
echo "user1@yourdomain.com:{SHA512-CRYPT}<paste-hash>" | sudo tee -a /etc/dovecot/users
sudo chmod 640 /etc/dovecot/users
```

### `/etc/dovecot/conf.d/10-master.conf` — Postfix auth socket

```ini
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}

service lmtp {
  unix_listener /var/spool/postfix/private/dovecot-lmtp {
    mode = 0600
    user = postfix
    group = postfix
  }
}
```

### `/etc/dovecot/conf.d/10-ssl.conf`

```ini
ssl = required
ssl_cert = </etc/letsencrypt/live/mail.yourdomain.com/fullchain.pem
ssl_key  = </etc/letsencrypt/live/mail.yourdomain.com/privkey.pem
```

---

# 6️⃣ TLS Certificate

```bash
sudo apt install certbot -y
sudo certbot certonly --standalone -d mail.yourdomain.com
```

Set up auto-renewal hook to restart Postfix/Dovecot:

```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/restart-mail.sh
```

```bash
#!/bin/bash
systemctl restart postfix dovecot
```

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/restart-mail.sh
```

---

# 7️⃣ Configure DKIM (OpenDKIM)

```bash
sudo mkdir -p /etc/opendkim/keys/yourdomain.com
sudo opendkim-genkey -s mail -d yourdomain.com -D /etc/opendkim/keys/yourdomain.com/
sudo chown -R opendkim:opendkim /etc/opendkim/keys/
```

### `/etc/opendkim.conf`

```ini
Mode                  sv
Syslog                yes
SyslogSuccess         yes
LogWhy                yes
Canonicalization      relaxed/simple
Domain                yourdomain.com
Selector              mail
KeyFile               /etc/opendkim/keys/yourdomain.com/mail.private
Socket                inet:12301@localhost
```

Get your DKIM public key to add to DNS:

```bash
cat /etc/opendkim/keys/yourdomain.com/mail.txt
```

Add as TXT record for `mail._domainkey.yourdomain.com`.

---

# 8️⃣ Configure DNS

```dns
A      mail.yourdomain.com  →  <your-ip>
MX     10  mail.yourdomain.com
TXT    @   "v=spf1 mx ~all"
TXT    mail._domainkey  "<content from mail.txt>"
TXT    _dmarc  "v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com"
```

---

# 9️⃣ Start and Enable Services

```bash
sudo systemctl enable postfix dovecot opendkim
sudo systemctl restart postfix dovecot opendkim
```

---

# 🔟 Useful Diagnostic Commands

```bash
# View mail queue
postqueue -p

# Flush queue
postqueue -f

# Remove all queued mail
postsuper -d ALL

# Test SMTP connection
telnet mail.yourdomain.com 25
EHLO test.com
QUIT

# Check Postfix logs
tail -f /var/log/mail.log

# Check Dovecot auth
doveadm auth test user1@yourdomain.com password

# Check open ports
ss -tlnp | grep -E '25|587|993|995'

# Validate Postfix config
postfix check

# View active Postfix config
postconf -n
```

---

# 🧠 Memorization Trick

Manual stack responsibilities:

| Component | Role |
|---|---|
| **Postfix** | Routes and delivers email (SMTP) |
| **Dovecot** | Serves mailboxes to clients (IMAP/POP3) |
| **OpenDKIM** | Signs outgoing mail |
| **CertBot** | TLS certificates |
| **Fail2ban** | Blocks brute-force (add separately) |

---

# 🔥 Common Problems

### "Relay access denied" on port 25
→ Client is trying to send outbound through port 25. Use port 587 (submission) with auth.

### Authentication fails in mail client
→ Dovecot auth socket not connected to Postfix. Check `smtpd_sasl_path` and the unix listener in `10-master.conf`.

### Emails sent but landing in spam
→ DKIM not signing (check `opendkim` is running and milter is in `main.cf`), or PTR record missing.

### "Invalid user" in Dovecot logs
→ User not in `/etc/dovecot/users`, or mail directory doesn't exist. Check `mail_location` path.

### TLS errors
→ Cert path wrong or cert not readable by postfix user. Check permissions on `/etc/letsencrypt/live/`.
