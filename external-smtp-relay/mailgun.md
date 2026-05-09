# Mailgun

Mailgun is an API-first email service by Sinch, popular for its detailed email logs and developer-friendly REST API. Strong choice when email observability matters.

---

# 1️⃣ Prerequisites

- Mailgun account (free: 100 emails/day for 3 months, then pay-as-you-go)
- A domain you control
- DNS access

---

# 2️⃣ Add and Verify Your Domain

Sending → Domains → Add New Domain

Use a subdomain for sending: `mg.yourdomain.com`

**Why a subdomain?** Isolates sending reputation from your root domain's email (e.g., Google Workspace). Best practice.

Mailgun generates DNS records:

```dns
# TXT — domain verification
TXT  mg.yourdomain.com  "v=mailgun..."

# SPF
TXT  mg.yourdomain.com  "v=spf1 include:mailgun.org ~all"

# DKIM
TXT  krs._domainkey.mg.yourdomain.com  "k=rsa; p=<long-key>"

# MX (for receiving, optional)
MX   mg.yourdomain.com  10  mxa.mailgun.org
MX   mg.yourdomain.com  10  mxb.mailgun.org
```

After adding records → Verify DNS in Mailgun console.

---

# 3️⃣ Add DMARC

```dns
TXT  _dmarc.mg.yourdomain.com
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com
```

---

# 4️⃣ Get API Key

Settings → API Keys → Private API Key

Also note your **region**:
- US: `api.mailgun.net`
- EU: `api.eu.mailgun.net`

---

# 5️⃣ Send via API (curl)

```bash
curl -s -X POST \
  https://api.mailgun.net/v3/mg.yourdomain.com/messages \
  -u "api:YOUR_API_KEY" \
  -F from="Sender <sender@mg.yourdomain.com>" \
  -F to="recipient@example.com" \
  -F subject="Test" \
  -F text="Hello from Mailgun"
```

EU region:
```bash
curl -s -X POST \
  https://api.eu.mailgun.net/v3/mg.yourdomain.com/messages \
  ...
```

---

# 6️⃣ Send via API (Python)

```bash
pip install requests
```

```python
import requests

def send_email():
    return requests.post(
        "https://api.mailgun.net/v3/mg.yourdomain.com/messages",
        auth=("api", "YOUR_API_KEY"),
        data={
            "from": "Sender <sender@mg.yourdomain.com>",
            "to": ["recipient@example.com"],
            "subject": "Hello",
            "text": "Hello from Mailgun",
        },
    )

response = send_email()
print(response.json())
```

---

# 7️⃣ Send via SMTP

```
Host:     smtp.mailgun.org
Port:     587
Username: postmaster@mg.yourdomain.com
Password: <domain-specific-SMTP-password from Mailgun dashboard>
```

Note: SMTP credentials are per-domain, not your account password.

---

# 8️⃣ Webhooks (Bounce Handling)

Sending → Webhooks → Add Webhook

Events:
- `delivered`
- `permanent_fail` (hard bounce)
- `temporary_fail` (soft bounce)
- `complained` (spam report)
- `opened` / `clicked`

Payload is JSON, signed with your webhook signing key (verify the signature).

---

# 9️⃣ Email Logs

One of Mailgun's best features: Sending → Logs

- See every email, delivery status, bounce reason
- Searchable by recipient, subject, date
- Available for 30 days on paid plans, 1 day on free

---

# 🔟 Pricing

| Usage | Price |
|-------|-------|
| First 3 months | 100/day free |
| Pay-as-you-go | ~$1.00 per 1,000 emails |
| Foundation plan | 50,000/mo for ~$35 |

---

# 🧠 Memorization Trick

**S D A L**
- S = Subdomain (use `mg.yourdomain.com`)
- D = DNS (TXT + SPF + DKIM + MX)
- A = API key + region
- L = Logs (your debugging superpower)

---

# 🔥 Common Problems

### DNS not verifying
→ Wait 15–30 min for propagation. Use `dig TXT mg.yourdomain.com` to confirm

### "Domain not found" on API call
→ Wrong domain in URL, or using EU endpoint with US key (or vice versa)

### Emails landing in spam
→ DKIM not verified, or warming up too fast

### Free plan expired
→ After 3 months, Mailgun requires a credit card for pay-as-you-go
