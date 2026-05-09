# Mail Implementations

Reference guide for deploying email infrastructure as a sysadmin or DevOps engineer. Covers every major approach: from managed cloud providers to fully self-hosted stacks.

---

## Quick Decision

**Need human mailboxes (inbox)?** → [Managed Providers](#managed-providers)

**Need your app to send email?** → [External SMTP Relay](#external-smtp-relay)

**Want full control / data sovereignty?** → [Self-Hosted](#self-hosted)

Not sure? → [Full Comparison](comparison.md)

---

## External SMTP Relay

Your app or server sends email through a third-party relay. No IP reputation management. No mail server to run.

| Guide | Summary |
|---|---|
| [Amazon SES](external-smtp-relay/amazon-ses.md) | Cheapest at scale, AWS-native |
| [SendGrid](external-smtp-relay/sendgrid.md) | Best DX, marketing + transactional |
| [Mailgun](external-smtp-relay/mailgun.md) | API-first, excellent logs |
| [Postmark](external-smtp-relay/postmark.md) | Fastest delivery, transactional only |

→ [Category overview + common setup pattern](external-smtp-relay/README.md)

---

## Self-Hosted

You run the mail server on your own VPS. Full control, no per-user cost, but you own the operational burden.

| Guide | Summary |
|---|---|
| [Mailcow](self-hosted/mailcow.md) | Docker-based, recommended default |
| [iRedMail](self-hosted/iredmail.md) | Script install, no Docker needed |
| [Mail-in-a-Box](self-hosted/mail-in-a-box.md) | Automated everything, personal use |
| [Postfix + Dovecot](self-hosted/postfix-dovecot.md) | Manual stack, learn the internals |
| [Zimbra](self-hosted/zimbra/) | Enterprise suite: email + calendar + contacts |

→ [Category overview + pre-flight checklist](self-hosted/README.md)

---

## Managed Providers

Point your MX records at a provider. They handle everything. You pay per user per month.

| Guide | Summary |
|---|---|
| [Google Workspace](managed-providers/google-workspace.md) | Gmail for your domain |
| [Microsoft 365](managed-providers/microsoft-365.md) | Outlook + Office for your domain |
| [Zoho Mail](managed-providers/zoho-mail.md) | Free tier for up to 5 users |

→ [Category overview](managed-providers/README.md)

---

## Comparison

→ **[Full comparison of all options](comparison.md)** — side-by-side tables, decision flowchart, and DNS quick reference.

---

## DNS Fundamentals

Every mail setup — regardless of provider — requires these records:

| Record | Purpose |
|---|---|
| `MX` | Where incoming mail is delivered |
| `SPF` (TXT) | Which servers are allowed to send for your domain |
| `DKIM` (TXT/CNAME) | Cryptographic signature proving mail wasn't tampered with |
| `DMARC` (TXT) | Policy for what to do when SPF/DKIM fail |
| `PTR` | Reverse DNS — required for self-hosted, set by VPS provider |

SPF + DKIM + DMARC together = email authentication. All three are required for production deliverability.
