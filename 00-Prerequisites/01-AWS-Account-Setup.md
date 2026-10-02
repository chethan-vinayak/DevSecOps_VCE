# 01 — AWS Account Setup

| | |
|---|---|
| ⏱️ **Time** | 20–30 minutes (+ up to a few hours for account activation) |
| 🎯 **Outcome** | A working AWS account, region set to **Asia Pacific (Mumbai)**, and a **budget alert** that emails you before you spend money |
| 🧰 **You need** | Email address, mobile phone, a debit/credit card enabled for international/online transactions |

---

## 📚 Concept — what is AWS and why do we need it?

**AWS (Amazon Web Services)** rents out computers in Amazon's data centres. Instead of buying a server, you
"launch" one in a few clicks, use it for a few hours, and stop it when you are done.

In this workshop, the server you rent is called an **EC2 instance** (*Elastic Compute Cloud*). It is simply a
Linux computer in Mumbai that you control from your laptop.

> 🏠 **Analogy:** Buying a server = buying a house. AWS EC2 = booking a hotel room. You pay only for the nights
> (hours) you stay, and you can check out any time.

### Free plan vs Paid plan

AWS accounts created **on or after 15 July 2025** choose a plan at sign-up:

| | **Free plan** | **Paid plan** |
|---|---|---|
| Credits | Sign-up credits (up to the amount shown on the sign-up page) | Same credits |
| Charges | You **cannot** be charged beyond the credits | Pay-as-you-go after credits |
| Duration | 6 months, or until credits are used | No limit |
| Good for this workshop? | ✅ **Yes — choose this** | Also works |

The instance types used in this workshop — `t3.micro`, `c7i-flex.large`, `m7i-flex.large` — are on AWS's
Free Tier list for these new accounts.

> ⚠️ AWS changes its free-tier rules from time to time. The sign-up page you see is the source of truth.

---

## 🛠️ Steps

### Step 1 — Start the sign-up 🌐 Browser

1. Open **https://aws.amazon.com** and click **Create an AWS Account** (top-right).
2. Enter:
   - **Root user email address** — an email you check regularly (personal, not a shared lab email).
   - **AWS account name** — e.g. `ravi-devsecops-workshop`.
3. Click **Verify email address**, then enter the **verification code** sent to your email.
4. Create a **strong root password** (at least 12 characters, mix of letters, numbers and symbols).
   Save it in your private `workshop-secrets.txt` or a password manager.

✅ **Check:** you reach the **Contact information** page.

---

### Step 2 — Contact information 🌐 Browser

1. **How do you plan to use AWS?** → **Personal – for your own projects**.
2. Fill in your name, phone number (with country code **+91**) and address exactly as on your card/bank records.
3. Accept the customer agreement → **Continue**.

---

### Step 3 — Choose a plan 🌐 Browser

If the page asks you to choose between **Free plan** and **Paid plan**, choose **Free plan**.

> 💡 If you don't see this choice, your page layout may be different — continue; the budget alert in Step 7
> protects you either way.

---

### Step 4 — Billing information 🌐 Browser

1. Enter your **debit/credit card** details.
2. Your bank may ask for an **OTP** to approve a small verification charge (Indian cards typically see a
   charge of about ₹2 that is refunded/reversed).

❌ **If you see** *"There was a problem with your payment information"*
→ Your card is probably not enabled for **international / online transactions**. Enable it in your bank app
(look for "International usage" or "Online transactions"), then try again. RuPay-only cards often fail — use a
Visa or Mastercard.

---

### Step 5 — Confirm your identity 🌐 Browser

1. Choose **Text message (SMS)** and enter your mobile number.
2. Complete the security check (CAPTCHA) and enter the code you receive.

---

### Step 6 — Support plan 🌐 Browser

Select **Basic support – Free** → **Complete sign up**.

> ⏳ Activation can take from a few minutes up to a few hours. You will receive an email
> *"Your AWS Account is ready"*. Wait for it before continuing.

---

### Step 7 — Set a budget alert (protects you from surprise bills) 🌐 Browser

1. Sign in at **https://console.aws.amazon.com** → choose **Root user** → enter your email and password.
2. In the top search bar type **Budgets** → open **Budgets** (under *Billing and Cost Management*).
3. Click **Create budget**.
4. Choose **Use a template (simplified)** → select **Zero spend budget**.
5. **Email recipients:** your email address.
6. Click **Create budget**.

✅ **Check:** the Budgets page lists a budget named **My Zero-Spend Budget** (or similar).
You will now get an email if your account is ever charged even ₹1.

---

### Step 8 — Select the Mumbai region 🌐 Browser

AWS has data centres in many cities ("regions"). All our labs use **Mumbai** because it is closest to us
(faster SSH, lower latency).

1. Look at the **top-right** of the AWS console — you will see a region name like *N. Virginia* or *Ohio*.
2. Click it and choose **Asia Pacific (Mumbai) ap-south-1**.

✅ **Check:** the top-right of the console shows **Mumbai** (or `ap-south-1`).

> ⚠️ **Most common Day 1 problem:** "My instance disappeared!" — it didn't. The console is showing a
> **different region**. Always confirm **Mumbai** is selected before looking for your servers.

---

### Step 9 — Confirm EC2 is available 🌐 Browser

1. In the top search bar type **EC2** → open **EC2**.
2. You should see the EC2 Dashboard with an orange **Launch instance** button.

✅ **Check:** the EC2 dashboard opens **without** a message like *"Your account is being verified"* or
*"You are not subscribed to this service"*.

❌ **If you see** *"Your account is currently being verified"* → wait for the activation email (Step 6).
If it takes more than 24 hours, contact AWS Support from the console (Support → Create case → *Account and billing*).

> 🚫 **Do NOT launch an instance yet.** You will launch it together with the trainer in Day 1, Lab 03.

---

## 🛡️ Security angle — protect the root user

The **root user** (the email you signed up with) can do *anything* in your account, including deleting it.
Attackers who steal root credentials can run expensive crypto-mining servers on your card.

Two quick protections (strongly recommended, 3 minutes):

1. **Turn on MFA for root:** top-right account name → **Security credentials** →
   **Multi-factor authentication (MFA)** → **Assign MFA device** → *Authenticator app* (Google Authenticator,
   Microsoft Authenticator, etc.) → scan the QR code → enter two codes.
2. **Never create access keys for the root user.** We don't need any for this workshop.

---

## ❌ Common errors

| You see | Why | Fix |
|---|---|---|
| Payment/card error | Card not enabled for international/online use | Enable it in bank app; use Visa/Mastercard |
| No SMS code | Network delay / DND | Choose **Voice call** instead, or retry after 2 minutes |
| "Account being verified" for hours | AWS manual verification | Wait; check email (and spam folder) |
| Can't find my instance later | Wrong region selected | Switch to **Asia Pacific (Mumbai)** |

---

➡️ **Next:** [02-GitHub-Account-and-Token.md](02-GitHub-Account-and-Token.md)
