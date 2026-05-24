Here is a very detailed README for the pre-event that students can keep and refer back to throughout the competition.

---
# AWS Sprint Pre-Games: Complete Account Setup Guide

## Your reference guide for setting up an AWS account, IAM user, MFA, and budget alerts.

**Version:** 1.0  
**Last Updated:** 5/24/2026 
**Duration to complete:** 45-60 minutes  
**Difficulty:** Beginner (no cloud experience required)

---

## Table of Contents

1. [What You Will Accomplish](#what-you-will-accomplish)
2. [Before You Start](#before-you-start)
3. [Step 1: Create Your AWS Account](#step-1-create-your-aws-account)
4. [Step 2: Secure Your Root Account with MFA](#step-2-secure-your-root-account-with-mfa)
5. [Step 3: Create an IAM Admin User](#step-3-create-an-iam-admin-user)
6. [Step 4: Set Up Billing Alerts](#step-4-set-up-billing-alerts)
7. [Step 5: Test Your IAM User](#step-5-test-your-iam-user)
8. [Step 6: Optional — Install AWS CLI](#step-6-optional--install-aws-cli)
9. [Common Problems & Solutions](#common-problems--solutions)
10. [Security Best Practices](#security-best-practices)
11. [Quick Reference Card](#quick-reference-card)
12. [Need Help?](#need-help)

---

## What You Will Accomplish

By the end of this guide, you will have:

| Item | Why It Matters |
|------|----------------|
| ✅ AWS Free Tier account | Access to 40+ AWS services free for 12 months |
| ✅ Root account with MFA | Protects your account from hackers |
| ✅ IAM admin user | Daily use account — never log in as root |
| ✅ IAM user access keys | For AWS CLI and programmatic access |
| ✅ Budget alert ($5) | Prevents surprise bills |
| ✅ Verified account ready for Week 1 | You can launch EC2 immediately |

---

## Before You Start

### What You Need

| Item | Details | Where to Get It |
|------|---------|-----------------|
| Email address | Personal email (not @egerton if it expires) | Gmail, Outlook, Yahoo — any works |
| Phone number | Must receive SMS/text messages | Your personal mobile |
| Credit/Debit card | AWS requires for identity verification | Any international card (Visa, Mastercard, Amex) You **CANNOT** use MPESA visa for this|
| Laptop | Windows, Mac, or Linux | Your device |
| Internet connection | Stable enough to load web pages | Campus Wi-Fi, home, or mobile hotspot |

### Important Notes About Payment

| Question | Answer |
|----------|--------|
| **Will I be charged?** | No. Free tier covers everything in this course. |
| **Why does AWS need my card?** | Identity verification. Prevents bots and fraud. |
| **What if I don't have a credit card?** | Debit cards work if they have international capability.  |
| **What if my card is declined?** | Contact your bank to enable international transactions. Or borrow a parent's/sibling's card with permission. |
| **What if I forget to stop resources?** | Your $5 budget alert will email you before any real charge happens. |

### How Much Will This Cost?

**$0 for the entire sprint. If you are smart about it**

Every service we use has a free tier:
- EC2 t2.micro: 750 hours/month free
- EBS storage: 30 GB free
- VPC, IAM, Budgets: Always free
- Data transfer: First 100 GB free

The $5 budget alert is a safety net. You will never reach it.

---

## Step 1: Create Your AWS Account

### 1.1 Go to AWS

Open your browser and go to: **https://aws.amazon.com**

Click the orange **"Create an AWS Account"** button (top right).

### 1.2 Enter Your Email and Account Name

| Field | What to Put |
|-------|-------------|
| Root user email address | Your personal email (e.g., yourname@gmail.com) |
| AWS account name | `[YourName]-sprint` (e.g., `alex-sprint`) |

**Pro tip:** Do NOT use your @egerton.ac.ke email. It may expire after graduation and you will lose access to your account forever.

Click **"Verify email address"**.

### 1.3 Verify Your Email

AWS will send a verification code to your email. Check your inbox (and spam folder).

Enter the 6-digit code. Click **"Verify"**.

### 1.4 Create a Strong Password

| Field | Requirement |
|-------|-------------|
| Root user password | Minimum 8 characters. Upper + lower + number + symbol. |

**Example:** `Sprint2026!aws`

**Do not lose this password.** Save it in a password manager (Bitwarden, LastPass, or Google Password Manager).

Click **"Continue"**.

### 1.5 Enter Contact Information

| Field | What to Put |
|-------|-------------|
| Account type | **Personal** (not Professional) |
| Full name | Your legal name |
| Phone number | Your mobile number (with country code: +254 for Kenya) |
| Country/Region | Kenya (or your country) |
| Address | Your physical or mailing address |
| City/Town | Your city |
| State/Province/Region | Your county or region |
| Postal/ZIP code | Your postal code |
| AWS Customer Agreement | ✅ Check "I have read and agree..." |

Click **"Verify phone number"**.

### 1.6 Verify Your Phone Number

AWS will call or text you with a 4-digit code. Enter it in the box.

Click **"Continue"**.

### 1.7 Enter Payment Information

| Field | What to Put |
|-------|-------------|
| Credit/Debit card number | Your card number |
| Expiration date | MM/YY |
| Cardholder name | As it appears on card |
| Billing address | Same as above or card's registered address |

**Important:** AWS will charge a small temporary authorization (usually $1). This is NOT a real charge. It will disappear in 3-5 days.

Click **"Verify and Continue"**.

### 1.8 Identity Verification (Phone)

AWS may ask for phone verification again. Enter your number, receive a code, and submit.

### 1.9 Choose Support Plan

Select **"Basic Support — Free"**.

Do NOT select Developer, Business, or Enterprise (those cost money).

Click **"Complete sign up"**.

### 1.10 Wait for Account Activation

AWS takes **5 minutes to 24 hours** to activate new accounts.

You will receive an email: **"Your AWS account is ready"**

While waiting, move to the next steps (you can do them after activation).

---

## Step 2: Secure Your Root Account with MFA

**Why this matters:** If someone steals your root password, they own your entire AWS account. MFA stops them cold.

### 2.1 Install Google Authenticator on Your Phone

| Phone Type | Where to Get It |
|------------|-----------------|
| iPhone | App Store → Google Authenticator |
| Android | Google Play Store → Google Authenticator |
| Alternative | Microsoft Authenticator, Authy, or Duo Mobile |

### 2.2 Log in as Root User

Go to: **https://console.aws.amazon.com**

Use your root email and password (from Step 1).

### 2.3 Go to IAM Dashboard

In the search bar at the top, type **"IAM"** and click it.

### 2.4 Enable MFA

On the IAM dashboard, find the **"Security status"** section.

Look for **"Root access key"** and **"Root MFA"** — it will say "Not enabled" next to MFA.

Click **"Add MFA"** or **"Activate MFA"**.

### 2.5 Configure MFA Device

| Option | Choice |
|--------|--------|
| Virtual MFA device | ☑️ Select this |
| Device name | Type `root-mfa` |

Click **"Next"**.

### 2.6 Scan the QR Code

Open Google Authenticator on your phone.

Tap the **"+"** button (or "Add Code").

Choose **"Scan a QR code"**.

Point your phone at the computer screen to scan the QR code.

A 6-digit code will appear in the app.

### 2.7 Enter Two Consecutive Codes

| Field | What to Enter |
|-------|---------------|
| MFA code 1 | The 6-digit code currently showing in Google Authenticator |
| MFA code 2 | Wait 30 seconds for the code to change → enter the NEW code |

Click **"Add MFA"**.

### 2.8 Confirmation

You will see: **"MFA device added successfully"**

 Root account is now secured.

---

## Step 3: Create an IAM Admin User

**Why this matters:** Root account should NEVER be used for daily work. IAM users are safer and can be deleted/rotated.

### 3.1 Still in IAM Dashboard

Click **"Users"** on the left menu.

Click **"Create user"**.

### 3.2 User Details

| Field | What to Put |
|-------|-------------|
| User name | `[yourfirstname]-admin` (e.g., `alex-admin`) |
| Provide user access to AWS Management Console |  Check this |
| I want to create an IAM user |  Select this |
| Console password |  Select "Autogenerated password" (you will change it on first login) |
| Users must create a new password at next sign-in |  Check this |

Click **"Next"**.

### 3.3 Set Permissions

| Option | Choice |
|--------|--------|
| Add user to group | ❌ Skip this for now |
| Copy permissions from existing user | ❌ Skip |
| **Attach policies directly** | ☑️ Select this |

In the policy search box, type `AdministratorAccess`.

☑️ Check the box next to **AdministratorAccess**.

**Note:** In production, you would use least privilege. For learning, AdminAccess is fine because you're on free tier with a budget alert.

Click **"Next"**.

### 3.4 Review and Create User

Review your settings. They should look like:

- User name: `[yourname]-admin`
- Console access: Yes
- Permissions: AdministratorAccess

Click **"Create user"**.

### 3.5 DOWNLOAD THE CREDENTIALS (.CSV FILE)

**THIS IS CRITICAL.**

After creating the user, AWS shows a green success box.

Click the **"Download .csv file"** button.

Save this file somewhere safe. It contains:
- User name
- Console sign-in URL
- Password
- Access key ID (for CLI)
- Secret access key (for CLI)

**If you lose this file:** You will need to delete the user and create a new one.

### 3.6 Sign Out of Root

Click your name in the top right corner (next to "us-east-1").

Click **"Sign out"**.

---

## Step 4: Set Up Billing Alerts

**Why this matters:** New AWS users sometimes forget to stop resources. A $5 budget alert emails you before any real charge happens.

### 4.1 Sign In as IAM User

Open the **Console sign-in URL** from your .csv file. It looks like:
`https://your-account-id.signin.aws.amazon.com/console`

Enter:
- **IAM user name:** from .csv file (e.g., `alex-admin`)
- **Password:** from .csv file

You will be prompted to change your password. Create a strong one.

### 4.2 Go to Billing Dashboard

In the search bar, type **"Billing"** and click **"Billing and Cost Management"**.

### 4.3 Enable Billing Access for IAM Users (First Time Only)

If this is your first time accessing billing as an IAM user:

You may see "Access Denied" or a blank page.

**Solution:** You must log in as Root user once to enable IAM billing access.

1. Sign out of IAM user
2. Sign in as Root user (your root email + password + MFA code)
3. In the search bar, type "IAM"
4. Click "Account settings" on the left
5. Find "IAM User and Role Access to Billing Information"
6. ☑️ Check "Activate IAM Access"
7. Click "Update"
8. Sign out of Root
9. Sign back in as your IAM user
10. Go to Billing Dashboard again — it should now work

### 4.4 Create a Budget

In Billing Dashboard, click **"Budgets"** on the left menu.

Click **"Create budget"**.

### 4.5 Choose Budget Type

| Option | Choice |
|--------|--------|
| Use a template | ☑️ Select this (easiest) |
| Template type | ☑️ Select "Monthly cost budget" |

### 4.6 Configure Budget

| Field | What to Put |
|-------|-------------|
| Budget name | `sprint-budget` |
| Budget amount (monthly) | **$5** |
| Email recipients | Your email address |

Click **"Create budget"**.

### 4.7 (Optional) Create Additional Alerts

You can add a second alert at 80% ($4) to get an early warning.

Click **"Edit budget"** → Add alert at 80% → Enter your email → Save.

☑️ Budget alert is now active.

**What happens if you hit $5?** AWS sends you an email. You log in and stop any running resources. No surprise bills.

---

## Step 5: Test Your IAM User

### 5.1 Verify Console Access

- [ ] You can log in using your IAM user credentials
- [ ] You see the AWS Management Console home page
- [ ] You do NOT see "Account is not fully activated" messages

### 5.2 Launch a Test EC2 Instance (No Cost)

This confirms your account can actually do things.

1. In search bar, type **"EC2"** and click it
2. Click **"Launch instance"**
3. Name: `test-instance`
4. AMI: Select **"Amazon Linux 2023"** (free tier eligible)
5. Instance type: Select **"t2.micro"** (free tier eligible)
6. Key pair: Click **"Create new key pair"**
   - Name: `sprint-key`
   - Type: RSA
   - Private key format: .pem (Mac/Linux) or .ppk (Windows with PuTTY)
   - Click **"Create key pair"** → it downloads automatically
7. Keep all other defaults
8. Click **"Launch instance"**

Wait 1-2 minutes for the instance to show **"Running"**.

### 5.3 Terminate the Test Instance

After confirming it launches:

1. Check the box next to your `test-instance`
2. Click **"Instance state"** → **"Terminate"**
3. Click **"Terminate"** to confirm

**Why terminate?** You don't want it counting against your free tier hours.

☑️ Test complete. Your account works perfectly for Week 1.

---

## Step 6: Optional — Install AWS CLI

**Why install CLI?** Weeks 5-8 use AWS CLI for Auto Scaling, Terraform, and Kubernetes. Do this now to save time later.

### Windows

```powershell
# Download the installer
https://awscli.amazonaws.com/AWSCLIV2.msi

# Run the downloaded .msi file
# Follow the installation wizard (all defaults)

# Verify installation
aws --version
```

### Mac

```bash
# Download and install via Homebrew (easiest)
brew install awscli

# Or use the official installer
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

# Verify installation
aws --version
```

### Linux (Ubuntu/Debian)

```bash
# Download and install
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
```

### Configure AWS CLI

After installing, configure it with your IAM user credentials:

```bash
aws configure
```

You will be prompted:

| Prompt | What to Enter |
|--------|---------------|
| AWS Access Key ID | From your .csv file (e.g., AKIA...) |
| AWS Secret Access Key | From your .csv file |
| Default region name | `us-east-1` (or `eu-west-1` for Ireland) |
| Default output format | `json` |

### Test CLI

```bash
aws sts get-caller-identity
```

Expected output:
```json
{
    "UserId": "AIDA...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/alex-admin"
}
```

☑️ CLI is ready.

---

## Common Problems & Solutions

### Problem 1: "Your card was declined"

**Why this happens:** Your bank blocks international transactions.

**Solutions:**
- Call your bank and ask them to enable international transactions (tell them it's a $1 verification, not a real purchase)
- Use a different card
- Use a virtual card (M-Pesa Global, CardBlanc, etc.)
- Borrow a parent's or sibling's card with permission

---

### Problem 2: "This phone number is already associated with an AWS account"

**Why this happens:** You or someone else used this number before.

**Solutions:**
- Use a different phone number (friend, family, second SIM)
- If it's your own old account, log into that one instead
- Contact AWS Support (free for billing issues)

---

### Problem 3: "Account is not fully activated" for 24+ hours

**Why this happens:** AWS verification sometimes takes longer.

**Solution:** 
- Wait. Check your email (and spam) for AWS messages.
- If >48 hours, contact AWS Support → "Create a case" → "Account and billing"

---

### Problem 4: "I lost my .csv file with IAM credentials"

**Why this happens:** You saved it somewhere and forgot.

**Solution:**
1. Log in as Root user
2. Go to IAM → Users → Click your username
3. Click "Security credentials"
4. Under "Access keys" → "Create access key"
5. Choose "Command Line Interface (CLI)" → Confirm
6. Click "Create access key"
7. **Download the new .csv file immediately**
8. Delete the old access key (click the X next to it)

---

### Problem 5: "Access Denied" when viewing Billing

**Why this happens:** IAM users cannot see billing by default.

**Solution:**
1. Log in as Root user
2. Go to IAM → Account settings
3. ✅ Check "Activate IAM Access to Billing Information"
4. Sign out of Root
5. Sign back in as IAM user

---

### Problem 6: "My budget alert email never arrived"

**Why this happens:** Budget alerts take up to 24 hours to activate.

**Solution:**
- Wait 24 hours after creating budget
- Check spam folder
- Verify email address is correct in budget settings

---

### Problem 7: "AWS CLI says 'Unable to locate credentials'"

**Why this happens:** You didn't run `aws configure` or entered wrong keys.

**Solution:**
```bash
# Run configure again
aws configure

# Check if credentials exist (should show your keys)
cat ~/.aws/credentials
```

---

### Problem 8: "I accidentally spent money"

**Why this happens:** You launched a non-free-tier resource or left it running too long.

**Solutions:**
- **First, stop the resource:** Terminate EC2, delete volumes, etc.
- **Check the bill:** Billing Dashboard → Bills
- **If under $5:** Ignore it — that's why we have the alert
- **If over $5:** Contact AWS Support → Ask for a one-time courtesy waiver (they often grant it for students)

---

## Security Best Practices

| Practice | Why It Matters | How To Do It |
|----------|----------------|--------------|
| Never share your password | Prevents others from accessing your account | Keep it in a password manager |
| Never commit access keys to GitHub | Bots scan GitHub for keys within minutes | Use `.gitignore` or environment variables |
| Always use IAM user, never root | Root has unlimited power | Log in as `[yourname]-admin` daily |
| Keep MFA on root account permanently | Stops hackers even if password is stolen | Already done in Step 2 |
| Delete unused access keys | Old keys are security risks | IAM → Users → Security credentials → Delete |
| Use budget alerts | Prevents surprise bills | Already done in Step 4 |

---

## Quick Reference Card

### Your Credentials (Fill This In)

| Item | Your Value |
|------|------------|
| Root account email | _________________ |
| Root account password | _________________ |
| Root MFA backup code (from setup) | _________________ |
| IAM user name | _________________ |
| IAM user password | _________________ |
| IAM user sign-in URL | _________________ |
| IAM access key ID | _________________ |
| IAM secret access key | _________________ |
| AWS account ID (12 digits) | _________________ |

**Store this reference card somewhere safe.** A password manager is best. A notebook in a locked drawer is fine.

### Important URLs

| URL | Purpose |
|-----|---------|
| https://console.aws.amazon.com | AWS Management Console (root login) |
| https://your-account-id.signin.aws.amazon.com/console | IAM user login (replace with your account ID) |
| https://aws.amazon.com/free | Free tier details |

### Emergency Stop (If You Think You're Being Charged)

1. Go to EC2 Dashboard → Instances → Select all → Terminate
2. Go to S3 → Buckets → Delete any non-empty buckets
3. Go to RDS → Databases → Delete any databases
4. Go to Billing Dashboard → Budgets → Confirm alert is active

---

## Need Help?

### Self-Help Resources

| Resource | Link |
|----------|------|
| AWS Free Tier documentation | https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-free-tier.html |
| IAM user guide | https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html |
| AWS CLI installation | https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html |

### Live Help During Sprint

- 📧 Email: 
- 📱 WhatsApp group: 
- 👥 Peer support: Ask in the WhatsApp group

### AWS Support (Free for Account & Billing Issues)

1. Go to https://console.aws.amazon.com/support/home
2. Click "Create case"
3. Choose "Account and billing"
4. Describe your issue
5. Submit

**Response time:** Usually 24 hours for free support.

---

## You're Ready for Week 1!

After completing all steps:

-  AWS account created
-  Root MFA enabled
-  IAM admin user created
-  Budget alert ($5) active
-  Test EC2 launched and terminated
-  (Optional) AWS CLI installed

**What's next:** Week 1 — EC2 Fundamentals

You will launch your first EC2 instance, SSH into it, and serve a web page.

**May the odds be ever in your favor.**

