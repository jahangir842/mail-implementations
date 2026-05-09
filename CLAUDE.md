# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

A documentation knowledge base for sysadmins and DevOps engineers covering email deployment across all major approaches. All content is Markdown. No build steps, tests, or dependencies.

## Structure

```
README.md                          # Navigation hub with quick decision guide
comparison.md                      # Master comparison tables + decision flowchart

external-smtp-relay/
  README.md                        # Category overview + common setup pattern
  amazon-ses.md                    # Full AWS SES walkthrough
  sendgrid.md                      # SendGrid SMTP + API guide
  mailgun.md                       # Mailgun API + subdomain strategy
  postmark.md                      # Postmark transactional-only guide

self-hosted/
  README.md                        # Pre-flight checklist + required DNS + ports
  mailcow.md                       # Docker-based (recommended default)
  iredmail.md                      # Script install, bare-metal
  mail-in-a-box.md                 # Automated single-script, personal use
  postfix-dovecot.md               # Manual stack, full config examples
  zimbra/
    introduction.md                # Zimbra feature overview
    install-on-ubuntu.md           # Step-by-step install guide

managed-providers/
  README.md                        # Category overview + managed vs relay distinction
  google-workspace.md              # Google Workspace DNS + DevOps automation
  microsoft-365.md                 # M365 DKIM/SMTP relay patterns
  zoho-mail.md                     # Zoho free tier and IMAP limits
```

## Three Categories

1. **External SMTP Relay** (`external-smtp-relay/`) — for app/server outbound email. No inbox. Pay per email sent. (SES, SendGrid, Mailgun, Postmark)

2. **Self-Hosted** (`self-hosted/`) — full mail server on your own VPS. Own IP reputation, storage, and ops. (Mailcow, iRedMail, Mail-in-a-Box, Postfix+Dovecot, Zimbra)

3. **Managed Providers** (`managed-providers/`) — full mailboxes, no server to run. Pay per user/month. (Google Workspace, M365, Zoho)

## Writing Conventions

- Numbered emoji headers (1️⃣, 2️⃣, …) for step sequences
- DNS records in fenced `dns` code blocks
- Shell commands in `bash` code blocks
- Each guide ends with a `🧠 Memorization Trick` and `🔥 Common Problems` section
- Use a subdomain for SMTP relay sending (e.g., `mg.yourdomain.com`) to isolate reputation from root domain

## Key Technical Facts to Preserve

- SPF, DKIM, and DMARC are all required for production deliverability — never omit any of them
- PTR records are set by the VPS provider, not the DNS registrar
- Self-hosting requires: port 25 open, static IP, PTR record, clean IP reputation — check all before starting
- Mailcow is the recommended self-hosted default; Postfix+Dovecot is for learning only
- Managed providers and SMTP relays are complementary, not alternatives — teams commonly use both
