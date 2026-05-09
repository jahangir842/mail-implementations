# Postmark

Postmark is a transactional-email-only service known for extremely fast delivery (usually under 10 seconds) and excellent inbox placement. It does not support bulk/marketing email — this is by design.

---

# 1️⃣ Prerequisites

- Postmark account (free trial: 100 emails)
- A domain you control
- DNS access

---

# 2️⃣ Create a Server

In Postmark, email is sent through **Servers** — logical containers for your app.

Dashboard → Servers → Create Server

Name it after your app or environment (e.g., `production`, `staging`).

---

# 3️⃣ Add and Verify a Sender Domain

Your Server → Sender Signatures → Add Domain

Postmark generates DNS records:

```dns
# DKIM
TXT  20230101._domainkey.yourdomain.com  "k=rsa; p=<key>"

# Return-Path (for bounce tracking)
CNAME  pm-bounces.yourdomain.com  →  pm.mtasv.net
```

SPF note: Postmark uses the `pm.mtasv.net` include:

```dns
TXT  @  "v=spf1 include:spf.mtasv.net ~all"
```

After adding records → Verify in Postmark.

---

# 4️⃣ Add DMARC

```dns
TXT  _dmarc.yourdomain.com
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com
```

---

# 5️⃣ Get Server API Token

Your Server → API Tokens → copy the **Server API Token**

(Different from your account API token.)

---

# 6️⃣ Send via API (curl)

```bash
curl -X POST https://api.postmarkapp.com/email \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -H "X-Postmark-Server-Token: YOUR_SERVER_TOKEN" \
  -d '{
    "From": "sender@yourdomain.com",
    "To": "recipient@example.com",
    "Subject": "Test",
    "TextBody": "Hello from Postmark"
  }'
```

---

# 7️⃣ Send via API (Python)

```bash
pip install postmarker
```

```python
from postmarker.core import PostmarkClient

postmark = PostmarkClient(server_token='YOUR_SERVER_TOKEN')

postmark.emails.send(
    From='sender@yourdomain.com',
    To='recipient@example.com',
    Subject='Hello',
    TextBody='Hello from Postmark',
)
```

---

# 8️⃣ Send via SMTP

```
Host:     smtp.postmarkapp.com
Port:     587
Username: <your-server-api-token>
Password: <your-server-api-token>
```

Both username and password are the same token.

---

# 9️⃣ Bounce Handling (Webhooks)

Your Server → Settings → Webhooks

Set a URL. Events:
- `Bounce` (hard and soft)
- `SpamComplaint`
- `Delivery`
- `Open` / `Click`

Postmark also has a **Bounces API** to query and manage bounces programmatically:

```bash
curl https://api.postmarkapp.com/bounces \
  -H "X-Postmark-Server-Token: YOUR_SERVER_TOKEN"
```

---

# 🔟 Templates

Postmark has server-side templates — useful for keeping HTML out of code:

```bash
# Send using a template
curl -X POST https://api.postmarkapp.com/email/withTemplate \
  -H "X-Postmark-Server-Token: YOUR_SERVER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "From": "sender@yourdomain.com",
    "To": "recipient@example.com",
    "TemplateAlias": "welcome",
    "TemplateModel": {"name": "John"}
  }'
```

---

# 1️⃣1️⃣ Pricing

| Plan | Price |
|------|-------|
| Trial | 100 emails free |
| Pay-as-you-go | $1.50 per 1,000 emails |
| Monthly (100k) | ~$15/mo |

More expensive per email than SES/Mailgun, but faster delivery and better support.

---

# 🧠 Memorization Trick

**Postmark = Transactional ONLY**

Think: Post office — delivers one letter at a time, reliably and fast. Not a bulk mail warehouse.

Key rule: **Don't use Postmark for newsletters or marketing.** They will suspend your account.

---

# 🔥 Common Problems

### "422 The 'From' address is not a registered sender signature"
→ Add and verify your sender domain first

### Account suspended
→ Marketing/bulk email detected. Postmark is transactional only.

### SMTP auth failing
→ Both username AND password must be your server token (not account token)

### Delivery slow in testing
→ Check spam folder first. If consistently slow, check your DKIM is verified.
