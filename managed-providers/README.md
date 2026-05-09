# Managed Email Providers

Managed providers give you full mailboxes — send, receive, webmail, mobile sync — without running any server. You point your domain's MX records at them and pay per user per month.

**Use this when:** You need business email for humans (inbox, calendar, contacts) rather than app-to-user transactional sending.

---

## Options Covered

| Provider | Price/user/mo | Best For |
|---|---|---|
| [Google Workspace](google-workspace.md) | $6–$18 | Teams already on Google products |
| [Microsoft 365](microsoft-365.md) | $6–$22 | Teams already on Office/Windows |
| [Zoho Mail](zoho-mail.md) | Free–$4 | Cost-sensitive, small business |

---

## How It Works

1. Sign up with the provider and add your domain
2. Verify domain ownership (TXT record)
3. Replace your MX records with the provider's servers
4. Add SPF, DKIM, DMARC
5. Create user accounts

No servers to manage. The provider handles delivery, spam filtering, storage, and uptime.

---

## vs. External SMTP Relay

| | Managed Provider | SMTP Relay |
|---|---|---|
| Purpose | Human mailboxes | App/transactional sending |
| Has inbox | ✅ | ❌ |
| Cost model | Per user/month | Per email sent |
| Examples | Google Workspace, M365 | SES, SendGrid |

You can — and often should — **use both**: Google Workspace for your team's mailboxes, and SES/SendGrid for your application's outbound email.

---

## Common DNS Template

```dns
# Replace ALL existing MX records
MX   1    <provider-mx-1>
MX   5    <provider-mx-2>
MX   10   <provider-mx-3>

# SPF — authorize the provider
TXT  @    "v=spf1 include:<provider-spf> ~all"

# DKIM — provider generates this TXT or CNAME record
TXT  <selector>._domainkey  "v=DKIM1; k=rsa; p=<key>"

# DMARC
TXT  _dmarc  "v=DMARC1; p=quarantine; rua=mailto:admin@yourdomain.com"
```

---

## See Also

- [Full comparison of all options](../comparison.md)
