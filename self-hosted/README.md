# Self-Hosted Mail Servers

Self-hosting gives you full control over mailboxes, storage, and data. The trade-off is operational overhead: you own IP reputation, spam filtering, security patching, and uptime.

**Use this when:** You want unlimited mailboxes, full data sovereignty, or are building a homelab/learning environment.

---

## Options Covered

| Solution | Stack | Difficulty | Best For |
|---|---|---|---|
| [Mailcow](mailcow.md) | Docker (Postfix + Dovecot + SOGo) | Medium | Modern, recommended starting point |
| [iRedMail](iredmail.md) | Postfix + Dovecot + Roundcube | Medium | Script-based, minimal Docker required |
| [Mail-in-a-Box](mail-in-a-box.md) | All-in-one (Ubuntu only) | Easy | Personal use, automated everything |
| [Postfix + Dovecot](postfix-dovecot.md) | Manual stack | Hard | Learning, maximum control |
| [Zimbra](zimbra/) | Java-based enterprise suite | Hard | Enterprise, calendar+contacts+email |

---

## Before You Start — Critical Checklist

Run these checks **before** installing anything:

### 1. Is port 25 open?

```bash
telnet gmail-smtp-in.l.google.com 25
```

Many VPS providers (AWS EC2, Google Cloud, Hetzner by default) block outbound port 25. If blocked, you cannot send email — request an unblock or use an SMTP relay.

### 2. Do you have a static IP?

Self-hosting requires a static IP. Dynamic IPs are almost always blacklisted.

### 3. Can you set a PTR (reverse DNS) record?

```
your.server.ip → mail.yourdomain.com
```

Your VPS provider sets this, not your DNS registrar. Without it, your emails go to spam.

### 4. Check your IP reputation

Before spending hours on setup, verify your IP isn't already blacklisted:

```bash
# Check multiple blacklists at once
# Use: https://mxtoolbox.com/blacklists.aspx
```

---

## Required DNS for Any Self-Hosted Setup

```dns
# Point to your mail server
A      mail.yourdomain.com  →  <your-server-ip>

# Declare your mail server for the domain
MX     10  mail.yourdomain.com

# SPF — only your server sends mail
TXT    @   "v=spf1 mx ~all"

# DMARC
TXT    _dmarc  "v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com"

# DKIM — generated after installation
TXT    mail._domainkey  "v=DKIM1; k=rsa; p=<generated-key>"
```

---

## Required Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 25 | SMTP | Server-to-server mail delivery (inbound) |
| 587 | SMTP | Client submission (your apps/email clients) |
| 993 | IMAPS | Secure IMAP (email clients) |
| 995 | POP3S | Secure POP3 |
| 443 | HTTPS | Webmail |
| 80 | HTTP | Let's Encrypt / redirect to HTTPS |

Open these in your VPS firewall:

```bash
sudo ufw allow 25,587,993,995,443,80/tcp
```

---

## See Also

- [Full comparison of all options](../comparison.md)
