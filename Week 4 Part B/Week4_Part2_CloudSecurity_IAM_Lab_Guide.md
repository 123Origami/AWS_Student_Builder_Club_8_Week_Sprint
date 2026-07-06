# Cloud FORGE — Week 4 (Part 2) Lab Guide
## Cloud Security: AWS Identity and Access Management (IAM)

**Track:** AWS Student Builders — Egerton University
**Based on:** Week 4 Part 2 session slides (Cloud Security: AWS IAM)
**Estimated time:** 75–100 minutes
**XP available:** 100 points

---

## 1. Lab Overview

Cloud security starts with controlling who can do what in your account. In this lab you will put the Week 4 IAM concepts into practice: the shared responsibility model, the principle of least privilege, IAM users/groups/roles/policies, MFA, and identity-based vs resource-based policies.

By the end, you will have built a small but realistic access-control setup: a group with a least-privilege policy, a user who inherits that policy through the group, an MFA-protected login, and a role that grants temporary permissions to an AWS service instead of a person.

### What you will build

```
AWS Account (Root — locked away and protected with MFA)
│
├── IAM Group: s3-readonly-group
│     └── Policy: AllowS3ReadAccess (least privilege, JSON policy)
│
├── IAM User: forge-learner
│     ├── Member of: s3-readonly-group
│     ├── MFA device: enabled
│     └── Console + programmatic access
│
└── IAM Role: forge-ec2-s3-role
      └── Trusted entity: EC2
      └── Policy: AllowS3ReadAccess (same permissions, granted to a resource, not a person)
```

### Learning objectives

By completing this lab, you will be able to:

1. Explain the AWS shared responsibility model and where IAM fits into it
2. Distinguish authentication from authorization
3. Apply the principle of least privilege when writing a policy
4. Enable MFA on an IAM user
5. Create and use IAM users, groups, roles, and policies correctly
6. Read and write a basic IAM policy in JSON
7. Explain the difference between identity-based and resource-based policies
8. Attach a role to an EC2 instance so it can access AWS resources without hardcoded credentials

---

## 2. Prerequisites

- [ ] An AWS account with root or administrator access (use a sandbox/free-tier account, not a production account)
- [ ] A smartphone with an authenticator app installed (Google Authenticator, Microsoft Authenticator, or Authy)
- [ ] An S3 bucket you can experiment with (you will create one during the lab)
- [ ] Basic familiarity with the AWS Management Console
- [ ] Completion of the Week 4 Part 2 session content (IAM, shared responsibility model, least privilege, MFA, policies)

> **Security note:** Never do this lab using your actual root credentials for anything other than the initial account-level MFA setup. Root should be used as little as possible — this is the first lesson of least privilege in practice.

---

## 3. Key Concepts Recap

| Term | Definition |
|------|------------|
| **Shared Responsibility Model** | AWS secures the infrastructure "of" the cloud (hardware, data centers, networking); the customer secures what they put "in" the cloud (data, IAM configuration, OS patching, application security) |
| **Authentication** | Confirming *who* you are (e.g. signing in with a username and password) |
| **Authorization** | Confirming *what* you're allowed to do once you're authenticated |
| **Principle of Least Privilege** | Grant only the permissions required to perform a task, nothing more, and add more only as needed |
| **MFA** | Multi-Factor Authentication — a second identity check beyond a password, e.g. a time-based code from an authenticator app |
| **IAM User** | A person or application with permanent credentials to interact with AWS |
| **IAM Group** | A collection of users who share the same set of permissions |
| **IAM Role** | A set of temporary permissions assumed by a resource (or trusted user) to perform a task |
| **IAM Policy** | A JSON document defining who can do what; can be identity-based or resource-based |

---

## 4. Part 1 — Secure the Foundation: Account-Level MFA

Before creating anything else, apply least privilege to the most powerful identity in the account: root.

1. Sign in to the AWS Management Console as the **root user**.
2. Go to **IAM** → **Dashboard**. Under "Security recommendations," find **Add MFA for root user**.
3. Choose **Authenticator app** as the MFA device type.
4. Scan the QR code with your authenticator app and enter two consecutive codes to confirm.
5. Complete setup.

**✅ Verify:** The IAM dashboard security status shows MFA enabled for the root user.

**Checkpoint question:** Under the shared responsibility model, is enabling MFA on your account AWS's responsibility or yours? Explain why.

---

## 5. Part 2 — Write a Least-Privilege Policy

### 5.1 Create a test S3 bucket

1. Go to **S3** → **Create bucket**.
2. Name it something globally unique, e.g. `cloudforge-iam-lab-<yourname>`.
3. Leave all other settings default → **Create bucket**.
4. Upload one small test file into the bucket (any file works).

### 5.2 Create the policy

1. Go to **IAM** → **Policies** → **Create policy**.
2. Switch to the **JSON** editor and enter:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::cloudforge-iam-lab-<yourname>",
        "arn:aws:s3:::cloudforge-iam-lab-<yourname>/*"
      ]
    }
  ]
}
```

3. Replace `<yourname>` with your actual bucket name.
4. Click **Next**, name the policy `AllowS3ReadAccess`, and add a short description.
5. Click **Create policy**.

**Checkpoint question:** This policy only grants `s3:GetObject` and `s3:ListBucket`. Name two S3 actions that this policy deliberately does **not** allow, and explain why leaving them out is an example of least privilege.

---

## 6. Part 3 — Groups, Users, and MFA

### 6.1 Create a group

1. Go to **IAM** → **User groups** → **Create group**.
2. Name: `s3-readonly-group`.
3. Under "Attach permissions policies," search for and select `AllowS3ReadAccess`.
4. Click **Create group**.

### 6.2 Create a user and add them to the group

1. Go to **IAM** → **Users** → **Create user**.
2. Name: `forge-learner`.
3. Check **Provide user access to the AWS Management Console**.
4. Choose **I want to create an IAM user** and set a custom password. Uncheck "Require password reset" only if this is a personal sandbox account.
5. On the permissions step, choose **Add user to group** and select `s3-readonly-group`.
6. Complete user creation and **download or copy the sign-in credentials** — you cannot retrieve the password again later.

### 6.3 Enable MFA for the new user

1. Sign out of root, sign back in as `forge-learner` using the account ID/alias sign-in link and the credentials from the previous step.
2. Go to **IAM** → **Users** → (your own user, `forge-learner`) → **Security credentials** tab.
3. Under "Multi-factor authentication (MFA)," click **Assign MFA device**.
4. Set it up with your authenticator app, the same way you did for root.

**✅ Verify:** `forge-learner` now requires both a password and an MFA code to sign in.

**Checkpoint question:** `forge-learner` was granted permissions by joining a group rather than by attaching the policy directly to the user. What is the practical advantage of managing permissions this way once you have many users?

---

## 7. Part 4 — Test Authorization

While signed in as `forge-learner`:

1. Go to **S3** and open your test bucket.
2. Confirm you **can** view and download the test file (`s3:GetObject` and `s3:ListBucket` are allowed).
3. Try to **upload** a new file to the bucket. This should **fail** with an access denied error, since `s3:PutObject` was never granted.
4. Try to **delete** the bucket. This should also fail.

**✅ Verify:** Read access works, write and delete access are both denied.

**Checkpoint question:** The user was authenticated successfully (correct password + MFA) but still couldn't upload a file. Which of the two concepts, authentication or authorization, blocked the upload? Explain the distinction in your own words.

---

## 8. Part 5 — IAM Roles: Granting Access to a Resource, Not a Person

Roles let an AWS service (like EC2) access other AWS services securely, without embedding long-lived credentials on the instance.

### 8.1 Create the role

1. Go to **IAM** → **Roles** → **Create role**.
2. Trusted entity type: **AWS service**.
3. Use case: **EC2** → **Next**.
4. Attach the `AllowS3ReadAccess` policy you created earlier.
5. Name the role `forge-ec2-s3-role` → **Create role**.

### 8.2 Attach the role to an EC2 instance

1. Go to **EC2** → **Launch instance** (or select an existing instance you own).
2. Under **Advanced details** → **IAM instance profile**, select `forge-ec2-s3-role`.
3. Launch (or, for an existing stopped instance: **Actions** → **Security** → **Modify IAM role**).

### 8.3 Test it from the instance

Connect to the instance (via SSH or EC2 Instance Connect) and run:

```bash
aws s3 ls s3://cloudforge-iam-lab-<yourname>
```

This should succeed **without you ever configuring an access key or secret key on the instance** — the temporary credentials come from the role.

**Checkpoint question:** Why is using an IAM role generally more secure for an EC2 instance than creating an IAM user and storing that user's access keys directly on the instance?

---

## 9. Part 6 — Identity-Based vs Resource-Based Policies (Conceptual + Optional Hands-On)

You have already worked with an **identity-based** policy (`AllowS3ReadAccess`, attached to a group and a role). Now attach a matching **resource-based** policy directly to the bucket to see the other side of policy evaluation.

1. Go to your S3 bucket → **Permissions** tab → **Bucket policy** → **Edit**.
2. Add a bucket policy that allows your account's `forge-learner` user (or role) read access, referencing the bucket's ARN. Use the IAM policy generator or write it manually, following the same `Effect`/`Action`/`Resource` structure as the policy from Part 2.
3. Save.

**Checkpoint question:** If an identity-based policy allows an action but a resource-based policy on the same resource does not explicitly allow it (and there's no explicit deny), does AWS grant or deny access? Look this up as part of the exercise and note the general evaluation logic in your own words.

---

## 10. Knowledge Check (20 XP)

1. In the shared responsibility model, who is responsible for patching the guest operating system on an EC2 instance: AWS or the customer?
2. What is the difference between an IAM user and an IAM role in terms of how long their credentials last?
3. Why should permissions usually be assigned to groups rather than directly to individual users?
4. What two things does MFA require to verify identity, beyond a password?
5. In the JSON policy from this lab, what does the `"Effect": "Allow"` field control, and what other value could it take?

---

## 11. Troubleshooting Guide

| Symptom | Likely Cause |
|---------|--------------|
| `forge-learner` can't sign in to the console | Wrong sign-in URL/account ID, or console access wasn't enabled during user creation |
| MFA setup QR code won't scan | Try "Enter code manually" instead, using the secret key AWS displays |
| User has no S3 access at all | Group not attached to the user, or policy not attached to the group, or policy JSON has an ARN typo |
| User can read but you expected them to also write | This is expected — the policy intentionally excludes `s3:PutObject`; that's least privilege working correctly |
| EC2 instance gets "Unable to locate credentials" | IAM instance profile/role was not attached, or the AWS CLI isn't picking up instance metadata correctly (check instance metadata service settings) |
| Resource-based policy edits are rejected | ARN in the bucket policy doesn't match the bucket name exactly, or the principal ARN is malformed |

---

## 12. Cleanup (Recommended)

To avoid leftover access in your sandbox account:

1. Detach `AllowS3ReadAccess` from `forge-ec2-s3-role`, then delete the role.
2. Remove `forge-learner` from `s3-readonly-group`, then delete the user.
3. Delete `s3-readonly-group`.
4. Delete the `AllowS3ReadAccess` policy.
5. Empty and delete the test S3 bucket.
6. Terminate any EC2 instance launched purely for this lab.

**✅ Final check:** Confirm no test users, roles, or buckets from this lab remain in your account.

---

## 13. Stretch Goals (Bonus XP)

- Write a policy that grants access only during specific hours using a `Condition` block (`aws:CurrentTime`)
- Create a second group with a **deny** policy for a specific S3 action and observe how explicit denies always override allows
- Set up an IAM policy that only allows access from your current IP address using `aws:SourceIp`
- Compare the permissions boundary feature to a regular policy and write a short note on when you'd use each

---

*Cloud FORGE — AWS Student Builders, Egerton University*
