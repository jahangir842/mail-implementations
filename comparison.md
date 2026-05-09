# Email Solution Comparison

A reference for choosing between all options covered in this repo.

---

## Decision Flowchart

```
What do you need?
│
├── Human mailboxes (inbox, send, receive)
│   └── → Managed Providers
│       ├── Already use Google tools?     → Google Workspace
│       ├── Already use Windows/Office?   → Microsoft 365
│       └── Need cheapest / small team?   → Zoho Mail
│
├── App/server sending email (notifications, alerts, password resets)
│   └── → External SMTP Relay
│       ├── On AWS / want cheapest?       → Amazon SES
│       ├── Need best developer UX?       → SendGrid
│       ├── Need best log visibility?     → Mailgun
│       └── Need fastest delivery / transactional only? → Postmark
│
└── Full control, data sovereignty, or learning
    └── → Self-Hosted
        ├── Modern, Docker, recommended?  → Mailcow
        ├── Script install, no Docker?    → iRedMail
        ├── Personal use, dead simple?    → Mail-in-a-Box
        ├── Learning how email works?     → Postfix + Dovecot
        └── Enterprise: calendar+email+contacts? → Zimbra
```

---

## External SMTP Relay Comparison

| | Amazon SES | SendGrid | Mailgun | Postmark |
|---|---|---|---|---|
| **Free tier** | 62k/mo (from EC2), else 3k trial | 100/day | 100/day (3 mo) | 100 emails |
| **Pricing (1k emails)** | ~$0.10 | ~$0.60–1.00 | ~$1.00 | ~$1.50 |
| **API quality** | Good (boto3) | Excellent | Excellent | Good |
| **Dashboard / logs** | CloudWatch | Good | Excellent | Good |
| **Deliverability** | Excellent | Excellent | Excellent | Best-in-class |
| **Bounce webhooks** | SNS | Yes | Yes | Yes |
| **Marketing email** | Yes | Yes | Yes | ❌ No |
| **Dedicated IP** | Yes (extra cost) | Yes (Pro+) | Yes (paid) | Yes (paid) |
| **Setup complexity** | Medium (AWS IAM) | Low | Low | Low |
| **Best for** | High volume, AWS users | Developers, marketing+transactional | Observability-focused | Fastest delivery |

### When to pick each relay

- **Amazon SES**: High volume at lowest cost. Already in AWS. Willing to deal with AWS IAM setup.
- **SendGrid**: You need one tool for both marketing campaigns and transactional emails.
- **Mailgun**: You want detailed per-email logs and good API ergonomics.
- **Postmark**: You care most about deliverability speed and inbox placement. Transactional only.

---

## Self-Hosted Comparison

| | Mailcow | iRedMail | Mail-in-a-Box | Postfix+Dovecot | Zimbra |
|---|---|---|---|---|---|
| **Installation** | Docker | Shell script | Single script | Manual | Interactive installer |
| **RAM requirement** | 4–6 GB | 4 GB | 1 GB | 512 MB | 8 GB+ |
| **Difficulty** | Medium | Medium | Easy | Hard | Hard |
| **Webmail** | SOGo | Roundcube | Roundcube | None (manual) | Zimbra webclient |
| **Calendar/Contacts** | SOGo | None | Nextcloud | None | Built-in |
| **Admin UI** | Yes | iRedAdmin | Yes | None | Yes |
| **Spam filter** | Rspamd | SpamAssassin | SpamAssassin | Manual | Built-in |
| **Antivirus** | ClamAV | ClamAV | ClamAV | Manual | Built-in |
| **Updates** | `./update.sh` | Version scripts | `sudo mailinabox` | Manual | Admin console |
| **Multi-domain** | Yes | Yes | Limited | Yes (manual) | Yes |
| **Recommended for** | Most DevOps setups | Bare-metal, no Docker | Personal/homelab | Learning | Enterprise |

### When to pick each self-hosted solution

- **Mailcow**: Default choice for new self-hosted setups. Docker makes it portable and easy to maintain.
- **iRedMail**: When Docker is not available or not wanted. Bare-metal, traditional package management.
- **Mail-in-a-Box**: Personal email. You want it working in 30 minutes with no decisions to make.
- **Postfix + Dovecot**: You want to deeply understand email. Learn the internals. Not for production unless you know what you're doing.
- **Zimbra**: Enterprise requirement — calendar, contacts, email in one suite, often for on-premise corporate deployment.

---

## Managed Provider Comparison

| | Google Workspace | Microsoft 365 | Zoho Mail |
|---|---|---|---|
| **Free tier** | ❌ (14-day trial) | ❌ (30-day trial) | ✅ 5 users, 5 GB each |
| **Entry price** | $6/user/mo | $6/user/mo | $1/user/mo (Lite) |
| **Webmail** | Gmail | Outlook Web | Zoho Mail |
| **Desktop client** | — | Outlook (Standard+) | — |
| **Calendar** | Google Calendar | Outlook Calendar | Zoho Calendar |
| **Video calls** | Google Meet | Teams | Zoho Meeting |
| **File storage** | Google Drive | OneDrive + SharePoint | Zoho WorkDrive |
| **Mobile push** | Excellent | Excellent | Good |
| **IMAP support** | Yes | Yes (paid) | Paid plans only |
| **Admin console** | Google Admin | Microsoft Admin Center | Zoho Admin |
| **Best for** | Google ecosystem users | Microsoft/Office users | Budget-conscious teams |

---

## Category Comparison: When to Use What

| Category | Has Inbox | Sends App Email | Cost Model | Control |
|---|---|---|---|---|
| Managed Provider | ✅ | ❌ (limited) | Per user/month | None |
| SMTP Relay | ❌ | ✅ | Per email sent | None |
| Self-Hosted | ✅ | ✅ | VPS cost only | Full |

### Most common production pattern

```
Team email          → Google Workspace or M365
App transactional   → Amazon SES or SendGrid
```

Using your Workspace SMTP to send app email is a bad idea — Google/Microsoft throttle it, and your domain reputation is tied to your team's mailboxes.

---

## IP Reputation Reality Check

Self-hosting success depends heavily on factors outside your control:

| Check | How | Why It Matters |
|---|---|---|
| Port 25 open | `telnet gmail-smtp-in.l.google.com 25` | Can't send without it |
| PTR record set | Ask VPS provider | Missing PTR = instant spam |
| IP not blacklisted | https://mxtoolbox.com/blacklists.aspx | Pre-abused IP = ban |
| IP not on dynamic ranges | same tool | Dynamic IP blocks = spam |

If any of these fail → self-hosting is not viable on that VPS. Use an SMTP relay instead.

---

## Quick Reference: DNS Records Every Setup Needs

| Record | Purpose | Who Sets It |
|---|---|---|
| `MX` | Where to deliver incoming mail | You (DNS) |
| `A` (mail.yourdomain.com) | IP of your mail server | You (DNS) |
| `PTR` | Reverse DNS (IP → hostname) | Your VPS provider |
| `SPF` (TXT) | Who is authorized to send mail | You (DNS) |
| `DKIM` (TXT or CNAME) | Cryptographic mail signature | You (DNS) — key from mail server |
| `DMARC` (TXT) | What to do when SPF/DKIM fail | You (DNS) |

SPF + DKIM + DMARC are non-negotiable for any production mail setup. Skipping any one of them causes deliverability problems.
