# Using Cloudflare + Gmail to Build a Domain Email System

(Hide Real Gmail, Expose Professional Identity)

---

## 1. Problem Statement

Engineers often want a professional email like:

```
contact@your-domain.com
```

But they do **not** want to expose their real Gmail address.

This becomes a real-world system design problem:

* How to receive emails on a domain
* How to reply from that domain
* How to hide the real infrastructure email
* How to avoid SMTP and DNS misconfigurations

This is not a cosmetic problem.
This is **identity architecture**.

---

## 2. Why This Is a Real Engineering Problem

If done incorrectly:

* Real Gmail leaks in headers
* Emails fail SPF / DKIM and go to spam
* DNS breaks mail delivery
* SMTP authentication fails
* Reputation of domain is damaged

This is a **production-grade system problem**.

---

## 3. Solution Paths (Architectures)

| Method                  | Verdict           |
| ----------------------- | ----------------- |
| Gmail Alias             | ❌ Reject          |
| Self-hosted mail server | ❌ High risk       |
| Workspace / Zoho        | ✅ Valid but heavy |
| Cloudflare + Gmail      | ✅ Best approach   |

We choose:

> **Cloudflare Email Routing + Gmail SMTP**

---

## 4. Step-by-Step Implementation (Production Grade)

### STEP 1 – Attach Domain to Cloudflare

1. Log in to **Cloudflare Dashboard**.
2. Click **Onboard a domain**.
3. Enter your domain:

   ```
   your-domain.com
   ```
4. Choose:
   ✅ Quick scan DNS
   ❌ Manual upload

#### Change Nameservers

Cloudflare gives two nameservers:

```
abc.ns.cloudflare.com
xyz.ns.cloudflare.com
```

In GoDaddy / Namecheap:

* Go to domain settings
* Open **Nameservers**
* Replace old ones with Cloudflare nameservers
* Save

🔴 Common failure:
People edit NS records inside DNS instead of changing nameservers.

Wait until Cloudflare shows:

```
Status: Active
```

---

### STEP 2 – Enable Cloudflare Email Routing

In Cloudflare:

1. Open your domain
2. Click **Email → Email Routing**
3. In Overview:

   * Click **Enable Email Routing**
   * Click **Add MX Records**

Wait until:

```
Routing Status: Enabled ✅
```

---

### STEP 3 – Add Destination Gmail

1. Open **Destination Addresses**
2. Click **Add Address**
3. Enter your Gmail:

   ```
   your-real-gmail@gmail.com
   ```
4. Cloudflare sends a verification email
5. Open Gmail
6. Click **Verify**

Do NOT continue until:

```
Status: Verified ✅
```

---

### STEP 4 – Create Custom Address Rule

Go to **Routing Rules**:

Create:

```
contact → your-gmail@gmail.com
```

Now email flow is:

```
contact@your-domain.com → Gmail
```

---

### STEP 5 – Verify Incoming Mail

Send a test email from another account to:

```
contact@your-domain.com
```

If not received:

* Check spam
* Wait DNS propagation
* Confirm rule is Active

---

### STEP 6 – Configure Gmail “Send As”

In Gmail:

1. Settings → Accounts & Import
2. Under “Send mail as”:

Add:

```
Name: Your Name
Email: contact@your-domain.com
```

Uncheck:

```
Treat as alias ❌
```

Choose:

```
Send through Gmail ✅
```

Gmail sends a verification email to:

```
contact@your-domain.com
```

Which arrives back in Gmail (because Cloudflare forwards it).

Verify it.

---

### STEP 7 – Force Professional Identity

Back in Gmail:

* Set `contact@your-domain.com` as **Default**
* Reply always from default

Now Gmail behaves like:

```
From: contact@your-domain.com
```

---

## 5. Real-World Failure Modes

| Failure             | Why It Happens                     |
| ------------------- | ---------------------------------- |
| Nameserver wrong    | Edited DNS instead of registrar NS |
| SMTP fails          | Using Cloudflare MX for sending    |
| Verification delay  | DNS + Cloudflare delay             |
| Gmail leaks address | Alias checkbox left enabled        |

---

## 6. Engineering Best Practices

* Never use Cloudflare MX as SMTP
* Keep forwarding, not POP3 pulls
* Never expose vault Gmail
* Wait before changing DNS repeatedly
* Use Gmail App Passwords if SMTP manual

---

## 7. Final Engineering Rule

> Hide the root identity.
> Expose the professional shell.
> Control everything from behind.

