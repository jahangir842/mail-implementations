# Zoho Mail

Zoho Mail is the most cost-effective managed email provider. The free plan supports up to 5 users with 5 GB each — the only reputable free business email that lets you use your own domain (not a `@zoho.com` address).

---

# 1️⃣ Plans

| Plan | Price/user/mo | Storage |
|---|---|---|
| Free (Forever) | $0 | 5 GB/user, up to 5 users |
| Mail Lite | ~$1 | 5 GB/user |
| Mail Premium | ~$4 | 50 GB/user |
| Workplace | ~$3 | Includes Zoho Docs, Cliq, etc. |

Free plan limitation: no IMAP/POP3 (web access only), no email forwarding to external. Paid plans unlock IMAP.

---

# 2️⃣ Sign Up and Add Domain

Go to: https://zoho.com/mail → Sign up → choose **Zoho Mail for Business** → enter your domain.

---

# 3️⃣ Verify Domain Ownership

Zoho gives you a TXT record:

```dns
TXT  @  zoho-verification=<code>
```

Or alternatively: add a CNAME record. After adding → click **Verify** in Zoho.

---

# 4️⃣ Configure MX Records

Replace all existing MX records with:

```dns
MX  10  mx.zoho.com
MX  20  mx2.zoho.com
MX  50  mx3.zoho.com
```

For EU data region:

```dns
MX  10  mx.zoho.eu
MX  20  mx2.zoho.eu
MX  50  mx3.zoho.eu
```

---

# 5️⃣ Add SPF

```dns
TXT  @  "v=spf1 include:zoho.com ~all"
```

EU region:

```dns
TXT  @  "v=spf1 include:zoho.eu ~all"
```

---

# 6️⃣ Add DKIM

In Zoho Mail Admin Console: Mail Admin Console → Domains → select domain → DKIM

Zoho generates:

```dns
TXT  zoho._domainkey.yourdomain.com  "v=DKIM1; k=rsa; p=<key>"
```

Add → click **Verify** in Zoho.

---

# 7️⃣ Add DMARC

```dns
TXT  _dmarc.yourdomain.com
Value: v=DMARC1; p=quarantine; rua=mailto:admin@yourdomain.com
```

---

# 8️⃣ Create Users

Zoho Admin Console → User Details → Add User

Zoho also supports user import via CSV:

Admin Console → Users → Bulk Operations → Import Users

---

# 9️⃣ Access Webmail

```
https://mail.zoho.com
```

Or set up a custom webmail URL:

Admin Console → Domains → Mail Client Access → configure `mail.yourdomain.com` CNAME to point to `business.zoho.com`.

---

# 🔟 Email Client Settings (IMAP — Paid Plans Only)

```
IMAP:  imappro.zoho.com   port 993  SSL
SMTP:  smtppro.zoho.com   port 465  SSL
       smtppro.zoho.com   port 587  TLS (alternative)
Username: full email address
Password: account password (or app-specific password if 2FA enabled)
```

EU region: replace `zoho.com` with `zoho.eu`.

---

# 1️⃣1️⃣ App-Specific Password (if 2FA enabled)

Zoho Account → Security → App Passwords → Generate

Use this password in email clients instead of your main password.

---

# 1️⃣2️⃣ Sending from App via SMTP

For apps/VPS sending through a Zoho mailbox:

```
Host:     smtppro.zoho.com
Port:     587
Username: sender@yourdomain.com
Password: account password (or app-specific password)
```

Zoho limits: free plan blocks SMTP outbound from external apps. Upgrade to Mail Lite for SMTP relay use.

---

# 🧠 Memorization Trick

**Zoho = Cheapest option with your own domain**

Free tier is genuinely free (no credit card), but:
- 5 users max
- No IMAP (web only)
- No external SMTP for apps

As soon as you need IMAP or more than 5 users → Mail Lite at ~$1/user/mo.

---

# 🔥 Common Problems

### MX records showing old values
→ Remove ALL old MX records before adding Zoho's. Check: `dig MX yourdomain.com`

### IMAP not working
→ Free plan does not include IMAP. Upgrade to Mail Lite.

### Email client rejected (credentials fail)
→ 2FA is on — generate an app-specific password in Zoho Account settings

### DKIM verification fails in Zoho
→ DNS propagation delay. Wait 30 min, then re-verify.

### Outbound SMTP blocked for apps
→ Free plan restriction. Upgrade plan, or use SES/SendGrid for transactional email instead.
