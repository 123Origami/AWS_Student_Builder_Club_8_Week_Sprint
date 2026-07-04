# Cloud FORGE — Week 3 Lab Guide
## Networking in the Cloud: Building Your First AWS VPC

**Track:** AWS Student Builders — Egerton University
**Based on:** Week 3 session slides (Networking: AWS VPC)
**Estimated time:** 90–120 minutes
**XP available:** 100 points

---

## 1. Lab Overview

In this lab you will design and build a custom Amazon VPC from scratch, the same way real cloud engineers architect isolated networks in AWS. By the end, you will have a working two-tier network with a public subnet reachable from the internet and a private subnet that is shielded from direct inbound access but can still reach the internet outbound through a NAT Gateway.

This lab turns the Week 3 concepts (CIDR notation, subnets, gateways, route tables, Security Groups, and NACLs) into a hands-on build you can show off in your portfolio.

### What you will build

```
                        Internet
                            |
                    [Internet Gateway]
                            |
        VPC: cloudforge-vpc (10.0.0.0/16)
        ┌─────────────────────────────────────────┐
        │  Public Subnet (10.0.1.0/24) — AZ-a      │
        │   ├─ Bastion/Web EC2 instance             │
        │   └─ NAT Gateway (with Elastic IP)        │
        │                                           │
        │  Private Subnet (10.0.2.0/24) — AZ-a      │
        │   └─ App EC2 instance (no public IP)      │
        └─────────────────────────────────────────┘
```

### Learning objectives

By completing this lab, you will be able to:

1. Explain the purpose of a VPC and calculate CIDR block sizes
2. Create a custom VPC with public and private subnets across an Availability Zone
3. Attach and configure an Internet Gateway for public internet access
4. Deploy a NAT Gateway to give private resources outbound-only internet access
5. Configure route tables to control subnet traffic flow
6. Launch EC2 instances into the correct subnets
7. Secure resources using Security Groups and Network ACLs
8. Validate the design by testing connectivity end-to-end

---

## 2. Prerequisites

- [ ] An active AWS account (Free Tier is sufficient)
- [ ] IAM user or role with permissions for VPC, EC2, and EIP management
- [ ] A key pair (`.pem` file) downloaded, or one created during the lab
- [ ] Basic familiarity with the AWS Management Console
- [ ] Completion of the Week 3 session content (VPC, CIDR, subnets, gateways, NACLs, Security Groups)

> **Cost note:** NAT Gateways and Elastic IPs are **not** Free Tier eligible and incur hourly charges. Complete the Cleanup section (Part 9) as soon as you finish to avoid unexpected billing.

---

## 3. Key Concepts Recap

| Term | Definition |
|------|------------|
| **VPC** | A logically isolated virtual network you define within a single AWS Region |
| **CIDR Block** | The IP address range assigned to your VPC/subnet (e.g. `10.0.0.0/16`). Ranges from `/16` (max) to `/28` (min) for a VPC |
| **Subnet** | A segment of your VPC's IP range tied to a single Availability Zone |
| **Public Subnet** | A subnet whose route table sends `0.0.0.0/0` traffic to an Internet Gateway |
| **Private Subnet** | A subnet with no direct route to an Internet Gateway |
| **Internet Gateway (IGW)** | A horizontally scalable, regional VPC component that allows traffic in/out of the internet |
| **NAT Gateway** | Lets instances in a private subnet initiate outbound internet traffic without being reachable from the internet |
| **Elastic IP (EIP)** | A static public IPv4 address you can attach to instances, NAT Gateways, or network interfaces |
| **Route Table** | A set of rules that determine where network traffic is directed |
| **Security Group** | A stateful, instance-level virtual firewall (allow rules only) |
| **NACL** | A stateless, subnet-level firewall (requires explicit allow **and** deny rules) |

---

## 4. Part 1 — Plan Your CIDR Blocks

Before touching the console, plan your addressing on paper. This is the step most learners skip, and it's the one that separates a rushed build from a real architecture decision.

| Resource | CIDR Block | Usable IPs (approx.) |
|----------|------------|------------------------|
| VPC | `10.0.0.0/16` | 65,536 |
| Public Subnet (AZ-a) | `10.0.1.0/24` | 251 |
| Private Subnet (AZ-a) | `10.0.2.0/24` | 251 |

**Checkpoint question:** Why can't two subnets in the same VPC share overlapping CIDR ranges? Write a one-sentence answer in your lab notes before continuing.

---

## 5. Part 2 — Create the VPC

1. Sign in to the **AWS Management Console** and navigate to **VPC**.
2. Select **Your VPCs** → **Create VPC**.
3. Choose **VPC only** (not the "VPC and more" wizard — you're building this manually to reinforce the concepts).
4. Configure:
   - **Name tag:** `cloudforge-vpc`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **Tenancy:** Default
5. Click **Create VPC**.

**✅ Verify:** Your new VPC appears in the VPC list with State = `Available`.

---

## 6. Part 3 — Create Subnets

### 6.1 Public Subnet

1. Go to **Subnets** → **Create subnet**.
2. Select `cloudforge-vpc` as the VPC.
3. Configure:
   - **Subnet name:** `public-subnet-1a`
   - **Availability Zone:** pick any AZ (e.g. `us-east-1a`) and note it down
   - **IPv4 CIDR block:** `10.0.1.0/24`
4. Click **Create subnet**.

### 6.2 Private Subnet

1. Repeat the process:
   - **Subnet name:** `private-subnet-1a`
   - **Availability Zone:** the **same AZ** you chose above
   - **IPv4 CIDR block:** `10.0.2.0/24`
2. Click **Create subnet**.

### 6.3 Enable auto-assign public IP for the public subnet

1. Select `public-subnet-1a` → **Actions** → **Edit subnet settings**.
2. Enable **Auto-assign public IPv4 address**.
3. Save.

**✅ Verify:** You have two subnets, both `10.0.x.0/24`, in the same AZ, mapped to `cloudforge-vpc`.

---

## 7. Part 4 — Gateways

### 7.1 Create and attach an Internet Gateway

1. Go to **Internet Gateways** → **Create internet gateway**.
2. Name it `cloudforge-igw` → **Create**.
3. Select it → **Actions** → **Attach to VPC** → choose `cloudforge-vpc`.

### 7.2 Allocate an Elastic IP

1. Go to **Elastic IPs** → **Allocate Elastic IP address** → **Allocate**.
2. Leave the default settings and confirm.

### 7.3 Create a NAT Gateway

1. Go to **NAT Gateways** → **Create NAT gateway**.
2. Configure:
   - **Name:** `cloudforge-nat`
   - **Subnet:** `public-subnet-1a` (NAT Gateways must sit in a **public** subnet)
   - **Connectivity type:** Public
   - **Elastic IP allocation ID:** select the EIP you just created
3. Click **Create NAT gateway** and wait until its state becomes `Available` (this can take a few minutes).

**Checkpoint question:** Why must a NAT Gateway always be deployed in a public subnet, even though its purpose is to serve private subnets?

---

## 8. Part 5 — Route Tables

### 8.1 Public route table

1. Go to **Route Tables** → **Create route table**.
2. Name: `public-rt` → VPC: `cloudforge-vpc` → **Create**.
3. Select it → **Edit routes** → **Add route**:
   - Destination: `0.0.0.0/0`
   - Target: **Internet Gateway** → `cloudforge-igw`
4. Save routes.
5. Go to the **Subnet associations** tab → **Edit subnet associations** → select `public-subnet-1a` → **Save**.

### 8.2 Private route table

1. Create another route table: `private-rt` → VPC: `cloudforge-vpc`.
2. **Edit routes** → **Add route**:
   - Destination: `0.0.0.0/0`
   - Target: **NAT Gateway** → `cloudforge-nat`
3. Save routes.
4. **Edit subnet associations** → select `private-subnet-1a` → **Save**.

**✅ Verify:** `public-subnet-1a` routes `0.0.0.0/0` → IGW. `private-subnet-1a` routes `0.0.0.0/0` → NAT Gateway.

---

## 9. Part 6 — Security Groups and NACLs

### 9.1 Security Group for the public (bastion/web) instance

1. Go to **Security Groups** → **Create security group**.
2. Name: `public-sg` → VPC: `cloudforge-vpc`.
3. Inbound rules:
   - SSH (22) — Source: **My IP**
   - HTTP (80) — Source: `0.0.0.0/0` (only if you plan to test a web server)
4. Outbound rules: leave the default (all traffic allowed).

### 9.2 Security Group for the private (app) instance

1. Create another security group: `private-sg` → VPC: `cloudforge-vpc`.
2. Inbound rules:
   - SSH (22) — Source: `public-sg` (select the security group itself as the source, not an IP range)
3. Outbound rules: leave the default.

### 9.3 (Optional stretch task) Custom NACL

1. Go to **Network ACLs** → **Create network ACL**.
2. Name: `cloudforge-nacl` → VPC: `cloudforge-vpc`.
3. Associate it with both subnets.
4. Add inbound and outbound rules to allow SSH (22), HTTP (80), and ephemeral ports (1024–65535), remembering that NACLs are **stateless** — every direction needs its own explicit rule.

**Checkpoint question:** A Security Group is described as "stateful" and a NACL as "stateless." In your own words, explain what that means for return traffic.

---

## 10. Part 7 — Launch the EC2 Instances

### 10.1 Public instance (bastion host)

1. Go to **EC2** → **Launch instance**.
2. Name: `bastion-public`.
3. AMI: Amazon Linux 2023 (Free Tier eligible).
4. Instance type: `t2.micro` or `t3.micro`.
5. Key pair: select or create one and download the `.pem` file.
6. Network settings → **Edit**:
   - VPC: `cloudforge-vpc`
   - Subnet: `public-subnet-1a`
   - Auto-assign public IP: **Enable**
   - Security group: `public-sg`
7. Launch the instance.

### 10.2 Private instance (app server)

1. Launch another instance: `app-private`.
2. Same AMI and instance type.
3. Network settings:
   - VPC: `cloudforge-vpc`
   - Subnet: `private-subnet-1a`
   - Auto-assign public IP: **Disable**
   - Security group: `private-sg`
4. Use the **same key pair** as the bastion host.
5. Launch the instance.

**✅ Verify:** `bastion-public` has a public IPv4 address. `app-private` has only a private IP.

---

## 11. Part 8 — Test Connectivity

This is where the architecture proves itself.

### 11.1 SSH into the bastion host

From your local machine:

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<bastion-public-IP>
```

### 11.2 Confirm the bastion has internet access

```bash
ping -c 3 amazon.com
```

You should get replies — this instance is in the public subnet with a route to the IGW.

### 11.3 Hop into the private instance from the bastion

Copy your key onto the bastion (or use SSH agent forwarding, which is the safer option) and connect to the app server's **private** IP:

```bash
ssh -i your-key.pem ec2-user@<app-private-private-IP>
```

### 11.4 Confirm the private instance has outbound-only internet access

From inside `app-private`:

```bash
ping -c 3 amazon.com
```

This should succeed because of the NAT Gateway route, even though nothing on the internet can initiate a connection **into** this instance.

### 11.5 Confirm the private instance is NOT directly reachable

From your local machine, try:

```bash
ssh -i your-key.pem ec2-user@<app-private-private-IP>
```

This should **time out** — the private subnet has no route from the public internet, and it has no public IP for you to even target directly. This failure is the expected, correct result.

**Checkpoint question:** Which specific piece of your configuration is responsible for the private instance being unreachable directly from the internet: the subnet, the route table, or the security group? Explain your reasoning.

---

## 12. Knowledge Check (20 XP)

1. What is the maximum and minimum CIDR block size allowed for a VPC?
2. Why does an Internet Gateway need to be attached to a VPC before a subnet can be "public"?
3. What happens to a subnet's traffic if it is never explicitly associated with a custom route table?
4. Name two AWS resources that can have an Elastic IP attached.
5. List one difference between a Security Group and a NACL besides "stateful vs stateless."

---

## 13. Troubleshooting Guide

| Symptom | Likely Cause |
|---------|--------------|
| Can't SSH into bastion host | Security group missing inbound rule for your current IP; check if your IP changed |
| Bastion has no internet access | Public route table missing `0.0.0.0/0` → IGW route, or subnet not associated with that route table |
| Private instance can't reach the internet | NAT Gateway not `Available` yet, private route table missing `0.0.0.0/0` → NAT route, or NAT Gateway placed in the wrong subnet |
| Can't hop from bastion to private instance | Private security group not allowing SSH from `public-sg`, or key pair not copied/forwarded correctly |
| Private instance unexpectedly reachable from internet | Auto-assign public IP was left enabled, or the "private" subnet is still associated with the public route table |

---

## 14. Part 9 — Cleanup (Required)

NAT Gateways and Elastic IPs bill continuously. Tear down your lab in this order to avoid dependency errors and lingering charges:

1. Terminate both EC2 instances (`bastion-public`, `app-private`).
2. Delete the NAT Gateway (`cloudforge-nat`) — wait for it to fully delete before the next step.
3. Release the Elastic IP.
4. Detach and delete the Internet Gateway (`cloudforge-igw`).
5. Delete both subnets.
6. Delete the custom route tables (`public-rt`, `private-rt`).
7. Delete the security groups (`public-sg`, `private-sg`).
8. Delete the custom NACL if you created one.
9. Delete the VPC (`cloudforge-vpc`).

**✅ Final check:** Confirm no Elastic IPs remain **allocated** in your account (this is the most common source of surprise charges).

---

## 15. Stretch Goals (Bonus XP)

- Add a second Availability Zone with matching public/private subnets and set up a highly available NAT Gateway pair
- Install and run a basic web server (Apache/Nginx) on the bastion host and confirm access over HTTP from your browser
- Replace the bastion SSH workflow with **AWS Systems Manager Session Manager** and remove the SSH inbound rule entirely
- Write a short reflection (150 words) comparing this manual build to what the "VPC and more" console wizard would have generated automatically

---

## 16. Submission Checklist

- [ ] Screenshot of VPC dashboard showing `cloudforge-vpc`
- [ ] Screenshot of both subnets with their CIDR blocks
- [ ] Screenshot of route tables and their routes
- [ ] Screenshot of successful `ping` from the private instance
- [ ] Screenshot of the failed direct SSH attempt to the private instance
- [ ] Answers to all Checkpoint and Knowledge Check questions
- [ ] Confirmation that cleanup was completed

---

*Cloud FORGE — AWS Student Builders, Egerton University*
