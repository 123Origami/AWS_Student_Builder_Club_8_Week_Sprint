# Cloud FORGE — Week 4: Storage and Security in the Arena
## Lab: Attach an EBS Volume & Build Your Own Custom AMI

---

### Quick Facts

| | |
|---|---|
| **Time Estimate** | 30–45 minutes |
| **Difficulty** | 🟢 Beginner (no prior storage experience needed) |
| **AWS Services Used** | EC2, Elastic Block Store (EBS), AMIs |
| **Cost** | Free Tier — just remember to clean up at the end (Step 9) |
| **Prerequisite** | A running EC2 instance from Week 1 (with nginx installed) |

---

## Before You Start: What Are You Actually Building?

Think of your EC2 instance as an apartment. It comes with some built-in storage (the root volume), but sometimes you need extra closet space — that's what an **EBS volume** is: a separate, attachable hard drive you can plug in, unplug, and move between instances.

In this lab you will:

1. **Create extra storage** (a new EBS volume) and attach it to your instance
2. **Format and mount it** so the operating system can actually use it
3. **Freeze a snapshot of your whole instance** into a reusable template (an **AMI**)
4. **Launch a brand-new instance** from that template and prove it works — with zero manual setup

By the end, you'll understand two of the most-tested AWS concepts: **block storage** and **machine images**.

---

## ⚠️ Read This First — The #1 Beginner Mistake

The original AWS docs (and a lot of older tutorials) say your new volume will show up as `/dev/xvdf`. **This is often wrong on modern instances.**

Here's why: AWS instances built on newer "Nitro" hardware (this includes most current instance types, like `t3.micro`) name attached volumes differently — as `/dev/nvme1n1`, `/dev/nvme2n1`, and so on, instead of the classic `/dev/xvdf`.

**Don't guess the device name. Always check first with `lsblk`.** Step 3 below shows you exactly how to read the output so you use the *correct* name for your instance, whatever it turns out to be.

> 💡 **Cohort tip:** If your device shows up as `nvme1n1`, your commands use `/dev/nvme1n1` — not `/dev/xvdf` — for every step in Part A. Copy-pasting commands from an old tutorial without checking this first is the most common error in this lab.

---

## Part A — Attach and Use an EBS Volume

### Step 1 — Create a New EBS Volume

1. In the AWS Console, go to **EC2 → Elastic Block Store → Volumes**
2. Click **Create Volume**
3. Configure it:
   - **Type:** `gp3` (this is AWS's default, general-purpose SSD — good enough for almost everything)
   - **Size:** `2 GB`
4. Set the **Availability Zone (AZ)** to match your EC2 instance's AZ **exactly**

   > ⚠️ **This is non-negotiable.** An EBS volume can only attach to an instance sitting in the *same* Availability Zone — think of it like two rooms needing to be in the same building before you can run a cable between them. Check your instance's AZ under **EC2 → Instances → [your instance] → Details panel**.

5. Click **Create Volume**

✅ **Checkpoint:** Your new volume should now show status `available` (not yet attached to anything).

---

### Step 2 — Attach the Volume to Your Instance

1. In the **Volumes** list, select your new volume
2. Click **Actions → Attach Volume**
3. Select your running EC2 instance from the dropdown
4. AWS will suggest a device name (often `/dev/sdf` or `/dev/xvdf`) — **note it, but don't fully trust it yet.** You'll confirm the real name in Step 3.
5. Click **Attach**

✅ **Checkpoint:** The volume's state should change to `in-use`.

---

### Step 3 — Find the Real Device Name, Then Format and Mount

SSH into your instance, then run this first, before anything else:

```bash
lsblk
```

Look for a disk that **doesn't have a mount point** — that's your new volume. It'll look like one of these:

```
# Older instance types:
xvdf          8G  0 disk

# Newer Nitro-based instance types (more common today):
nvme1n1       1G  0 disk
```

**Write down the exact name you see.** From here on, use *that* name everywhere the guide shows a device — this lab will use `DEVICE_NAME` as a placeholder. Wherever you see it, mentally swap in your real value (e.g. `/dev/nvme1n1` or `/dev/xvdf`).

```bash
# Format the volume as ext4
# WARNING: This erases all data on the volume — only do this once, on a brand-new volume
sudo mkfs -t ext4 /dev/DEVICE_NAME
```

```bash
# Create a mount point (a folder that acts as the "door" into your new storage)
sudo mkdir /data

# Mount the volume there
sudo mount /dev/DEVICE_NAME /data
```

```bash
# Verify it worked
df -h | grep /data
```

✅ **Checkpoint:** You should see your device mounted at `/data` with close to 2 GB available.

---

### Step 4 — Write a File and Take Your Screenshot

```bash
echo 'Hello AWS Sprint Week 2' | sudo tee /data/test.txt
cat /data/test.txt
```

**Expected output:**
```
Hello AWS Sprint Week 2
```

> 📸 **Screenshot 1 (required):** Your terminal showing the output of `cat /data/test.txt`.

---

### Step 5 — Make the Mount Survive a Reboot

Right now, if your instance restarts, `/data` will "forget" it's supposed to be connected to your volume. Fix that permanently:

```bash
echo '/dev/DEVICE_NAME /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
```

> **Why `nofail`?** Without it, if this volume is ever missing or detached when the instance boots, the *entire instance* can fail to start. `nofail` tells Linux: "try to mount it, but boot anyway if it's not there."

Verify:

```bash
cat /etc/fstab
```

The last line should show your new entry.

> 💡 **Extra safety check:** Since NVMe device names are assigned by discovery order rather than guaranteed permanently, run `sudo mount -a` after editing `/etc/fstab`. If it returns no errors, your fstab entry is valid and matches a real device.

---

## Part B — Create a Custom AMI

An **AMI (Amazon Machine Image)** is a snapshot of your entire instance — OS, installed software, configuration, all of it — packaged into a reusable template. Instead of manually reinstalling nginx every time you need a new server, you'll launch instances that already have it built in.

### Step 6 — Freeze Your Instance Into an AMI

1. **EC2 → Instances** → select your Week 1 nginx instance
2. **Actions → Image and Templates → Create Image**
3. Fill in:
   - **Name:** `aws-sprint-nginx-ami`
   - **Description:** `Week 2 lab — nginx pre-installed`
4. Leave everything else at default → **Create Image**

> AWS will briefly stop, snapshot, and restart your instance automatically — this is expected, not an error.

5. Go to **EC2 → AMIs** and wait for status to flip from `pending` to `available` (usually 3–5 minutes)

✅ **Checkpoint:** Status shows `available`.

---

### Step 7 — Launch a Fresh Instance From Your AMI

1. In **AMIs**, select `aws-sprint-nginx-ami` → **Launch instance from AMI**
2. Configure:
   - **Instance type:** `t2.micro`
   - **Key pair:** your existing `aws-sprint-key`
   - **Security group:** must allow
     - SSH — Port 22, Source: Anywhere (0.0.0.0/0)
     - HTTP — Port 80, Source: Anywhere (0.0.0.0/0)
3. **Launch instance**

---

### Step 8 — Prove It Worked

Wait for **Running** state, then SSH in using the **new** instance's public IP:

```bash
ssh -i aws-sprint-key.pem ec2-user@<new-instance-public-ip>
```

Once connected — **without installing anything** — run:

```bash
curl localhost
```

**Expected output:** the nginx welcome page HTML, appearing instantly. That's the whole point: nginx was already baked into the AMI.

> 📸 **Screenshot 2 (required):** Terminal output of `curl localhost` on the new instance.

---

## Step 9 — Cleanup (Don't Skip This!)

AWS bills for resources even when you're not actively using them. Clean up in this order:

| # | Action | Why |
|---|---|---|
| 1 | **Terminate** the test instance from Step 7 | You only need one running instance — avoid double compute charges |
| 2 | **Stop** (not terminate!) your original Week 1 instance | Preserves your nginx setup for future weeks |
| 3 | **Detach and delete** the 2 GB EBS volume from Part A | EBS volumes cost money even when detached and unused |
| 4 | **Keep** the AMI (`aws-sprint-nginx-ami`) | You'll reuse it in **Week 5** for the Auto Scaling lab |

**To delete the EBS volume:**
1. **EC2 → Elastic Block Store → Volumes**
2. Select your 2 GB volume
3. If attached: **Actions → Detach Volume**, wait for `available`
4. **Actions → Delete Volume** → confirm

---

## Submission

- Quiz link posted separately in the group chat
- **Deadline:** 48 hours from end of session
- 10 questions, auto-graded, 10 pts each (100 pts total)

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `mkfs` says "no such file or directory" | You probably typed the device name without `/dev/` in front, or used the wrong name — recheck `lsblk` |
| Volume won't attach | Volume and instance must be in the **exact same AZ** |
| `lsblk` doesn't show the device | Wait 30 seconds and rerun; if still missing, reboot the instance |
| `mkfs` says "device is busy" | It may already be mounted — run `sudo umount /dev/DEVICE_NAME` first |
| `curl localhost` shows nothing | Run `sudo systemctl status nginx`; if inactive, `sudo systemctl start nginx` |
| AMI stuck on "pending" | Wait a full 5 minutes — bigger snapshots take longer |
| Can't SSH into new instance | Check security group allows port 22, and confirm you're using the right `.pem` file |
| Mount looks wrong after reboot | Check `/etc/fstab` for typos; test with `sudo mount -a` before rebooting |

---

## Key Concepts — What You Should Walk Away Knowing

### EBS (Elastic Block Store)
- Persistent block storage that attaches to EC2 instances — like a detachable hard drive
- Data survives instance stop/start, and even termination (unless "Delete on Termination" is enabled)
- Must live in the same Availability Zone as the instance it attaches to
- Device naming varies: older instances show `/dev/xvdf`-style names; Nitro-based instances show `/dev/nvme*` — **always confirm with `lsblk`, never assume**

### AMI (Amazon Machine Image)
- A template capturing an instance's OS, software, and configuration at a moment in time
- Every instance launched from an AMI starts identical — no manual setup needed
- Stored in S3 behind the scenes; you're billed for the snapshot storage

### Snapshots
- Point-in-time backups of an EBS volume
- Incremental — only changed blocks are saved after the first snapshot
- The building blocks that AMIs are made from
- Can restore or move data across regions and accounts
