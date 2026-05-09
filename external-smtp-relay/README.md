# External SMTP Relay

An SMTP relay service handles outbound email delivery on your behalf. Your app (or server) connects to their SMTP endpoint or API, and they handle delivery, reputation, and bounce tracking.

**Use this when:** You need to send transactional emails (notifications, password resets, alerts) from an app or VPS without managing your own mail server or IP reputation.

---

## Options Covered

| Service | Free Tier | Best For |
|---|---|---|
| [Amazon SES](amazon-ses.md) | 62,000/mo (from EC2) | AWS-native, high volume, cheapest |
| [SendGrid](sendgrid.md) | 100/day | Developer UX, marketing + transactional |
| [Mailgun](mailgun.md) | 100/day (3 months) | API-first, good log visibility |
| [Postmark](postmark.md) | 100/mo | Transactional only, fast delivery |

---

## Common Setup Pattern

Regardless of provider, the DNS and configuration steps are the same:

1. Verify your sending domain (add DNS TXT record)
2. Add SPF record
3. Enable DKIM (provider generates CNAME/TXT records)
4. Add DMARC record
5. Get SMTP credentials or API key
6. Configure your app

---

## DNS Template

```dns
# SPF — authorize the relay to send on your behalf
TXT  @  "v=spf1 include:<provider-spf-domain> ~all"

# DKIM — provider generates these, usually 2–3 CNAME records
CNAME  <selector>._domainkey  →  <provider-dkim-endpoint>

# DMARC — what to do when checks fail
TXT  _dmarc  "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com"
```

---

## SMTP vs API

Most providers offer both:

- **SMTP**: Port 587 (STARTTLS) — works with any language/framework
- **HTTP API**: REST calls — faster, better error handling, provider-specific

For new projects, prefer the **HTTP API** when available. For legacy apps or Postfix relay config, use SMTP.

---

## See Also

- [Full comparison of all options](../comparison.md)
