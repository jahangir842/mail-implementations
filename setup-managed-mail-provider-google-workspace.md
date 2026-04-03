# 🚀 High-Level Flow

1. Buy Google Workspace subscription
2. Verify your domain
3. Update DNS (MX, SPF, DKIM, DMARC)
4. Create users
5. Start sending/receiving mail

---

# 🧱 Step 1: Sign up for Google Workspace

Go to:
👉 [https://workspace.google.com/](https://workspace.google.com/)

* Enter your domain (e.g. `yourdomain.com`)
* Create admin account (e.g. `admin@yourdomain.com`)

---

# 🔐 Step 2: Verify Domain Ownership

Google will give you a **TXT record** like:

```
google-site-verification=abc123xyz
```

👉 Add it to your DNS:

```
Type: TXT
Name: @
Value: google-site-verification=abc123xyz
```

After adding:

* Click **Verify** in Google Admin Console

---

# 🌐 Step 3: Configure MX Records (MOST IMPORTANT)

This tells the world to send emails to Google instead of your server.

### Replace ALL existing MX records with:

```
Priority | Server
---------|-------------------------
1        | ASPMX.L.GOOGLE.COM
5        | ALT1.ASPMX.L.GOOGLE.COM
5        | ALT2.ASPMX.L.GOOGLE.COM
10       | ALT3.ASPMX.L.GOOGLE.COM
10       | ALT4.ASPMX.L.GOOGLE.COM
```

👉 Example (Cloudflare style):

```
Type: MX
Name: @
Mail server: ASPMX.L.GOOGLE.COM
Priority: 1
```

---

# 🛡️ Step 4: Add SPF (Anti-spoofing)

```
Type: TXT
Name: @
Value: v=spf1 include:_spf.google.com ~all
```

---

# ✍️ Step 5: Enable DKIM (Email signing)

In Google Admin Console:

* Go to: Apps → Gmail → Authenticate Email
* Generate DKIM key

It will give you something like:

```
Name: google._domainkey
Value: v=DKIM1; k=rsa; p=MIIBIjANBgkqh...
```

Add as TXT record.

Then click **Start Authentication**

---

# 🚨 Step 6: Add DMARC (Recommended)

```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=quarantine; rua=mailto:admin@yourdomain.com
```

---

# 👤 Step 7: Create Users

In Admin Console:

* Add users like:

  * `info@yourdomain.com`
  * `devops@yourdomain.com`

---

# 📬 Step 8: Test Email

* Send email from Gmail → your domain
* Send from your domain → Gmail

---

# ⚙️ Optional DevOps Enhancements

Since you're infra-focused, here’s how you can automate 👇

## 🔹 Example: Terraform (Cloudflare DNS)

```hcl
resource "cloudflare_record" "mx1" {
  zone_id = var.zone_id
  name    = "@"
  type    = "MX"
  value   = "ASPMX.L.GOOGLE.COM"
  priority = 1
}
```

## 🔹 Example: Ansible

```yaml
- name: Add SPF record
  community.general.cloudflare_dns:
    zone: yourdomain.com
    record: "@"
    type: TXT
    value: "v=spf1 include:_spf.google.com ~all"
    state: present
```

---

# ⚖️ Comparison (as you like multi-cloud view)

| Feature        | Google Workspace | AWS Equivalent          | Azure Equivalent             | Open Source |
| -------------- | ---------------- | ----------------------- | ---------------------------- | ----------- |
| Email hosting  | ✅ Built-in       | ❌ (SES is sending only) | ❌                            | Mailcow     |
| SMTP relay     | ✅                | **Amazon SES**          | Azure Communication Services | Postfix     |
| Webmail        | Gmail            | ❌                       | ❌                            | Roundcube   |
| Spam filtering | Excellent        | Basic                   | Basic                        | Rspamd      |

👉 Key insight:

* **AWS SES / Azure** = sending emails (apps, alerts)
* **Google Workspace** = full mailbox system

---

# 🧠 Memorization Trick

Think:

👉 **MX = Mail eXchange = “Where mail goes”**
👉 **SPF = “Who can send”**
👉 **DKIM = “Proof it wasn’t changed”**
👉 **DMARC = “What to do if fake”**

---

# 🔥 Common Mistakes

* ❌ Forgetting to remove old MX records
* ❌ DNS propagation not complete (wait 5–30 min)
* ❌ Not enabling DKIM → emails go to spam
* ❌ Using wrong DNS name (`@` vs domain)

---

