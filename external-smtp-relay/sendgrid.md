# SendGrid

SendGrid (owned by Twilio) is the most widely used SMTP relay and email API service. It handles both transactional and marketing email.

---

# 1️⃣ Prerequisites

- SendGrid account (free tier: 100 emails/day)
- A domain you control
- DNS access

---

# 2️⃣ Domain Authentication

This is the most important step. Without it, emails land in spam.

### In SendGrid Console:

Settings → Sender Authentication → Authenticate Your Domain

Enter your domain → SendGrid generates DNS records:

```dns
# CNAME records for DKIM (add all 3)
em1234.yourdomain.com       →  u1234567.wl.sendgrid.net
s1._domainkey.yourdomain.com →  s1.domainkey.u1234567.wl.sendgrid.net
s2._domainkey.yourdomain.com →  s2.domainkey.u1234567.wl.sendgrid.net
```

Also creates a CNAME to manage the SPF record automatically — no manual SPF needed.

After adding records → click **Verify** in SendGrid.

---

# 3️⃣ Add DMARC (Recommended)

```dns
TXT  _dmarc.yourdomain.com
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com
```

---

# 4️⃣ Get API Key

Settings → API Keys → Create API Key

Permissions needed:
- Full Access → for everything
- Restricted → Mail Send only for apps that only send

**Store the key immediately — shown only once.**

---

# 5️⃣ Send via SMTP

SMTP settings:

```
Host:     smtp.sendgrid.net
Port:     587 (STARTTLS)
Username: apikey        ← literal string "apikey"
Password: <your-api-key>
```

### Postfix relay config:

```
relayhost = [smtp.sendgrid.net]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt
```

`/etc/postfix/sasl_passwd`:

```
[smtp.sendgrid.net]:587  apikey:<your-api-key>
```

```bash
sudo postmap /etc/postfix/sasl_passwd
sudo systemctl restart postfix
```

---

# 6️⃣ Send via API (Python)

```bash
pip install sendgrid
```

```python
import sendgrid
from sendgrid.helpers.mail import Mail

sg = sendgrid.SendGridAPIClient(api_key='YOUR_API_KEY')

message = Mail(
    from_email='sender@yourdomain.com',
    to_emails='recipient@example.com',
    subject='Hello',
    plain_text_content='Hello from SendGrid'
)

response = sg.send(message)
print(response.status_code)
```

---

# 7️⃣ Send via API (curl)

```bash
curl -X POST https://api.sendgrid.com/v3/mail/send \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "personalizations": [{"to": [{"email": "recipient@example.com"}]}],
    "from": {"email": "sender@yourdomain.com"},
    "subject": "Test",
    "content": [{"type": "text/plain", "value": "Hello from SendGrid"}]
  }'
```

---

# 8️⃣ Event Webhooks (Bounce Handling)

Settings → Mail Settings → Event Webhook

Set your endpoint URL. Events sent:
- `delivered`
- `bounce`
- `spam_report`
- `open` / `click`

**Always handle bounces** — remove hard-bounced emails from your list to protect sender reputation.

---

# 9️⃣ Warm-Up (If New Domain)

SendGrid has an **IP Warm-Up** feature in Settings.

Manual schedule:
- Day 1–3: 50–200 emails/day
- Week 2: 500–1,000/day
- Week 3+: scale to volume

Never blast thousands of emails from a fresh domain.

---

# 🔟 Limits by Plan

| Plan | Daily Limit | Price |
|------|-------------|-------|
| Free | 100/day | $0 |
| Essentials | 100,000/mo | ~$20/mo |
| Pro | 1,500,000/mo | ~$90/mo |

---

# 🧠 Memorization Trick

**A D A W**
- A = Authenticate domain (CNAME records)
- D = DMARC record
- A = API key
- W = Webhook for bounces

---

# 🔥 Common Problems

### Emails go to spam
→ Domain authentication not verified or DMARC missing

### "The from address does not match a verified Sender Identity"
→ You must verify your sending domain or sender address first

### SMTP authentication failed
→ Username must be the literal string `apikey`, not your email

### Webhook not firing
→ Check your endpoint returns HTTP 200; SendGrid retries on non-200
