## Amazon Simple Email Service (Amazon SES)

I’ll cover:

1. Account preparation
2. Domain verification
3. DNS (SPF, DKIM, DMARC)
4. Sandbox → Production
5. SMTP credentials
6. Sending via CLI + Python example
7. Bounce/complaint handling
8. Production best practices

---

# 1️⃣ Prerequisites

You need:

* An AWS account in Amazon Web Services
* A verified domain (example: `example.com`)
* DNS access (Cloudflare, Route53, GoDaddy, etc.)

---

# 2️⃣ Choose Region (Important)

SES is regional.

Common choices:

* `us-east-1` (default)
* `eu-west-1`
* `ap-southeast-2`

Pick region close to your users.

Example:
If VPS is in Europe → use `eu-west-1`.

---

# 3️⃣ Verify Your Domain in SES

### Step 1 — Open SES Console

Go to:
AWS Console → SES → “Verified identities”

### Step 2 — Create Identity

* Choose **Domain**
* Enter: `example.com`
* Enable DKIM
* Create identity

SES will generate DNS records.

---

# 4️⃣ Add Required DNS Records

SES gives you:

### 1️⃣ TXT record (verification)

```dns
Name: _amazonses.example.com
Type: TXT
Value: random-verification-string
```

### 2️⃣ DKIM (CNAME records)

Example:

```dns
abc123._domainkey.example.com  →  abc123.dkim.amazonses.com
def456._domainkey.example.com  →  def456.dkim.amazonses.com
ghi789._domainkey.example.com  →  ghi789.dkim.amazonses.com
```

Add all 3.

---

# 5️⃣ Add SPF (If Not Already Present)

Add:

```dns
TXT
Name: example.com
Value: v=spf1 include:amazonses.com -all
```

If you already use Microsoft 365 or Google:

Example:

```dns
v=spf1 include:spf.protection.outlook.com include:amazonses.com -all
```

Only ONE SPF record allowed.

---

# 6️⃣ Add DMARC (Highly Recommended)

```dns
Name: _dmarc.example.com
Type: TXT
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com
```

Later you can switch `p=reject`.

---

# 7️⃣ Wait for Verification

Return to SES → Identity Status should become:

✅ Verified
✅ DKIM: Verified

This may take 5–30 minutes.

---

# 8️⃣ Sandbox vs Production

New SES accounts are in **Sandbox mode**.

Sandbox restrictions:

* Can send only to verified emails
* 200 emails/day limit

### Request Production Access

SES → Account dashboard → “Request production access”

Fill form:

* Use case (transactional emails)
* Website URL
* Estimated volume

Approval usually takes 24–48 hours.

---

# 9️⃣ Create SMTP Credentials

SES does NOT use your AWS root password.

Go to:

SES → SMTP Settings → Create SMTP credentials

This creates:

* SMTP Username
* SMTP Password

Store securely.

---

# 🔟 SMTP Endpoints

Format:

```
email-smtp.<region>.amazonaws.com
```

Example:

```
email-smtp.eu-west-1.amazonaws.com
Port: 587
```

Ports:

* 25 (often blocked)
* 587 (recommended)
* 465 (TLS)

---

# 1️⃣1️⃣ Test Using Telnet (Optional)

```bash
telnet email-smtp.eu-west-1.amazonaws.com 587
```

If blocked → VPS provider is blocking outbound SMTP.

---

# 1️⃣2️⃣ Send Email Using AWS CLI

Install CLI:

```bash
sudo apt install awscli
```

Configure:

```bash
aws configure
```

Send test:

```bash
aws ses send-email \
  --from sender@example.com \
  --destination ToAddresses=recipient@example.com \
  --message "Subject={Data=Test},Body={Text={Data=Hello World}}" \
  --region eu-west-1
```

---

# 1️⃣3️⃣ Send Email Using Python (boto3)

Install:

```bash
pip install boto3
```

Example:

```python
import boto3

ses = boto3.client('ses', region_name='eu-west-1')

response = ses.send_email(
    Source='sender@example.com',
    Destination={'ToAddresses': ['recipient@example.com']},
    Message={
        'Subject': {'Data': 'Test Email'},
        'Body': {'Text': {'Data': 'Hello from SES'}}
    }
)

print(response)
```

---

# 1️⃣4️⃣ Production Setup (Important)

## Configure Bounce & Complaint Handling

Create SNS topic in
Amazon Simple Notification Service

Steps:

1. Create SNS topic
2. Subscribe email or webhook
3. In SES → Configuration Set → Event destination
4. Attach SNS topic

Now you’ll receive:

* Bounce
* Complaint
* Delivery notifications

---

# 1️⃣5️⃣ Warm-Up Strategy (Critical)

If new account:

* Day 1: 50 emails
* Day 2: 100
* Gradually increase

Avoid blasting 10,000 emails immediately.

---

# 1️⃣6️⃣ Security Best Practices

* Use IAM user (not root)
* Rotate SMTP credentials
* Enforce DKIM
* Set DMARC to `reject` eventually
* Monitor CloudWatch metrics

SES integrates with:

* Amazon CloudWatch
* AWS Identity and Access Management

---

# 1️⃣7️⃣ Common Problems

### Emails go to spam?

* Missing DKIM
* Weak DMARC
* No reverse DNS on VPS (if using direct SMTP)

### “Email address not verified”

You are still in sandbox.

### SMTP timeout?

Port 25 blocked.

---

# 🧠 Memorization Trick

Think SES setup in 5 blocks:

**V D S S P**

* V = Verify domain
* D = DNS (SPF/DKIM/DMARC)
* S = Sandbox removal
* S = SMTP credentials
* P = Production monitoring

---

# 🏗 Architecture Example (DevOps Pattern)

Typical production setup:

```
FastAPI App (VPS)
        ↓
Amazon SES SMTP
        ↓
Recipient inbox
```

OR

```
Kubernetes → SES API → SNS → CloudWatch
```

---


