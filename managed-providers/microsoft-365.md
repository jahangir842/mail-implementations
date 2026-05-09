# Microsoft 365 (Exchange Online)

Microsoft 365 is the enterprise standard for business email, especially in Windows-heavy organizations. Exchange Online (the email component) pairs with Outlook, Teams, SharePoint, and OneDrive.

---

# 1️⃣ Plans

| Plan | Price/user/mo | Highlights |
|---|---|---|
| Business Basic | ~$6 | Exchange + Teams + 1TB OneDrive (web only) |
| Business Standard | ~$12.50 | + Desktop Office apps |
| Business Premium | ~$22 | + Advanced security (Defender) |
| Exchange Online Plan 1 | ~$4 | Email only, no Office apps |

---

# 2️⃣ Add Your Domain

Admin Center (admin.microsoft.com) → Settings → Domains → Add domain

Enter your domain → Microsoft gives you a TXT record to prove ownership:

```dns
TXT  @  MS=ms12345678
```

Add it → click **Verify** in Microsoft Admin Center.

---

# 3️⃣ Configure MX Records

Replace **all** existing MX records with Microsoft's:

```dns
MX  0  yourdomain-com.mail.protection.outlook.com
```

The exact value is shown in the DNS setup wizard in Admin Center. It's always `<yourdomain-com>.mail.protection.outlook.com` (hyphens replace dots in the tenant name).

**Delete any old MX records** — multiple MX records for different providers cause split delivery.

---

# 4️⃣ Add SPF

```dns
TXT  @  "v=spf1 include:spf.protection.outlook.com -all"
```

If you also send via another service (e.g., SES for app emails):

```dns
TXT  @  "v=spf1 include:spf.protection.outlook.com include:amazonses.com -all"
```

One SPF record only — merge all `include:` clauses into one record.

---

# 5️⃣ Enable DKIM

In Admin Center → Security (security.microsoft.com) → Email & Collaboration → Policies & Rules → Threat Policies → DKIM

Select your domain → Enable → Microsoft generates two CNAME records:

```dns
CNAME  selector1._domainkey.yourdomain.com  →  selector1-yourdomain-com._domainkey.yourtenant.onmicrosoft.com
CNAME  selector2._domainkey.yourdomain.com  →  selector2-yourdomain-com._domainkey.yourtenant.onmicrosoft.com
```

Add both CNAMEs → click **Enable** in the portal.

---

# 6️⃣ Add DMARC

```dns
TXT  _dmarc.yourdomain.com
Value: v=DMARC1; p=quarantine; rua=mailto:admin@yourdomain.com
```

---

# 7️⃣ Create Users

Admin Center → Users → Active Users → Add a user

Or bulk import via CSV:

Admin Center → Users → Active Users → ... → Import multiple users

---

# 8️⃣ Email Clients

**Web:** https://outlook.office.com

**Outlook desktop (Windows/Mac):** Auto-discovers settings when you enter your Microsoft 365 email address.

**Mobile:** Outlook app (iOS/Android) → Add account → enter email → auto-configure.

**Third-party clients (IMAP):**

```
IMAP:  outlook.office365.com  port 993  SSL
SMTP:  smtp.office365.com     port 587  STARTTLS
Username: full email address
```

Note: Basic auth (username/password) is disabled by default. Modern Auth (OAuth) required for most clients.

---

# 9️⃣ Send Mail from App / VPS via Office 365 SMTP

For apps that need to send mail through your M365 account:

### Option A: SMTP Relay (Direct Send)

No credentials needed if your VPS IP is whitelisted:

Admin Center → Settings → Org Settings → "SMTP Relay" setup.

```
Host:    <yourtenant>.mail.protection.outlook.com
Port:    25
No auth — only your whitelisted IPs can use this
```

### Option B: SMTP Auth (per mailbox)

```
Host:     smtp.office365.com
Port:     587
Username: sender@yourdomain.com
Password: mailbox password (or app password if MFA enabled)
```

Requires enabling SMTP AUTH per mailbox:
Admin Center → Users → select user → Mail → Manage email apps → Enable Authenticated SMTP.

---

# 🔟 Shared Mailboxes (No License Required)

Shared mailboxes (e.g., `info@yourdomain.com`, `support@yourdomain.com`) are free up to 50 GB:

Admin Center → Teams & Groups → Shared Mailboxes → Add

Users can access via Outlook or webmail without a separate login.

---

# 🧠 Memorization Trick

**M365 DNS = M S D D**
- **M** = MX record (route mail to Microsoft)
- **S** = SPF (authorize Microsoft to send)
- **D** = DKIM (two CNAME records, enable in Security portal)
- **D** = DMARC (what to do with failures)

---

# 🔥 Common Problems

### Mail still going to old server
→ Old MX records not deleted. Use `dig MX yourdomain.com` to confirm only Microsoft MX remains.

### DKIM cannot be enabled (button greyed out)
→ CNAME records not yet propagated. Wait and try again.

### SMTP auth fails from app
→ SMTP AUTH not enabled for the mailbox (see step 9B). Also check if MFA requires an app password.

### New user not receiving mail
→ License not assigned. Admin Center → Users → select user → Licenses → assign M365 license.

### Emails from VPS going to spam in M365
→ Set up SMTP relay (Step 9A) with your VPS IP whitelisted instead of using SMTP Auth.
