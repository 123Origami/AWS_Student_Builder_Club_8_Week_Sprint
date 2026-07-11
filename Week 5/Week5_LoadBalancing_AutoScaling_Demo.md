# Cloud FORGE - Week 5 Demo
## Load Balancing and Auto Scaling in the Arena

This demo walks through building a highly available, self-healing web tier on AWS: a launch template, an Auto Scaling Group, and an Application Load Balancer working together.

**What you'll need:** An AWS account with EC2 access, and a default VPC (or a VPC with at least 2 subnets in different AZs).

---

## Architecture Overview

By the end of this demo, traffic will flow like this:

```
Internet -> Application Load Balancer -> Target Group -> EC2 Instances (in an Auto Scaling Group, spread across 2+ AZs)
```

The ASG keeps a healthy number of instances running at all times. The ALB only sends traffic to instances that pass health checks. If an instance dies, the ASG replaces it, and the ALB automatically starts routing to the replacement once it's healthy.

---

## Part 1: Launch Template

The launch template is the blueprint the ASG uses every time it spawns a new instance.

1. Go to **EC2 > Launch Templates > Create launch template**
2. Name it `cloudforge-web-template`
3. Choose an **Amazon Linux 2023** AMI
4. Instance type: `t2.micro` (or `t3.micro` if in a region without t2 free tier)
5. Key pair: select an existing pair or skip if using Session Manager
6. Network settings: leave subnet blank (the ASG will assign this), but select or create a security group that allows:
   - HTTP (port 80) from anywhere (0.0.0.0/0)
   - SSH (port 22) from your IP only
7. Under **Advanced details > User data**, paste the following so every instance auto-installs a simple web server on boot:

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

8. Click **Create launch template**

This user data script is what makes health checks pass later - each instance will serve a simple page identifying itself, so you can visually confirm traffic is being distributed.

---

## Part 2: Target Group

The target group defines where the load balancer sends traffic, and how it checks if a target is healthy.

1. Go to **EC2 > Target Groups > Create target group**
2. Target type: **Instances**
3. Name it `cloudforge-web-tg`
4. Protocol: HTTP, Port: 80
5. VPC: select your VPC
6. Health check settings:
   - Protocol: HTTP
   - Path: `/`
   - Healthy threshold: 2
   - Unhealthy threshold: 2
   - Timeout: 5 seconds
   - Interval: 15 seconds
7. Skip "Register targets" for now (the ASG will register instances automatically)
8. Click **Create target group**

---

## Part 3: Auto Scaling Group

This is the piece that keeps your fleet alive and at the right size.

1. Go to **EC2 > Auto Scaling Groups > Create Auto Scaling group**
2. Name it `cloudforge-web-asg`
3. Select the launch template you created in Part 1
4. VPC and subnets: select at least 2 subnets in different Availability Zones
5. Under **Load balancing**, choose "Attach to an existing load balancer" and select the target group from Part 2
6. Health checks: enable **ELB health checks** in addition to EC2 status checks (this is what allows the ASG to replace an instance the load balancer has marked unhealthy, not just one that's technically stopped)
7. Group size:
   - Desired capacity: 2
   - Minimum capacity: 2
   - Maximum capacity: 4
8. Skip scaling policies for now, we'll add one in Part 5
9. Click **Create Auto Scaling group**

Watch the **Activity** tab. Within a few minutes you should see two instances launch and pass health checks.

---

## Part 4: Application Load Balancer

1. Go to **EC2 > Load Balancers > Create load balancer > Application Load Balancer**
2. Name it `cloudforge-web-alb`
3. Scheme: **Internet-facing**
4. VPC: select your VPC, and the same subnets used by the ASG
5. Security group: allow HTTP (80) from anywhere
6. Listener: HTTP on port 80, forwarding to `cloudforge-web-tg`
7. Click **Create load balancer**

Once it's provisioned (a couple of minutes), copy the **DNS name** from the load balancer's details page and paste it into your browser. Refresh a few times - you should see the hostname in the response change, confirming the ALB is distributing traffic across both instances.

---

## Part 5: Scaling Policy

Now let's make the ASG respond automatically to load instead of sitting at a fixed size.

1. Go to your ASG > **Automatic scaling** tab > **Create dynamic scaling policy**
2. Policy type: **Target tracking scaling**
3. Metric type: **Average CPU utilization**
4. Target value: `50`
5. Instances need: 60 seconds warm up before included in metrics
6. Click **Create**

This tells the ASG: "keep average CPU across the fleet close to 50 percent - scale out if it climbs above that, scale in if it drops well below it."

---

## Part 6: Testing Failure and Scaling

**Test self-healing:**
1. Pick one instance in the target group and manually stop or terminate it from the EC2 console
2. Watch the ASG Activity tab - it should launch a replacement automatically within a minute or two
3. Refresh the ALB DNS name in your browser - traffic keeps flowing throughout, since the second instance was still healthy

**Test scale-out (optional, requires generating load):**
1. Connect to one of the instances via Session Manager or SSH
2. Install a load generator: `sudo amazon-linux-extras install epel -y && sudo yum install -y stress`
3. Run: `stress --cpu 2 --timeout 300`
4. Watch the CloudWatch CPU metric climb, and watch the ASG launch additional instances once the target tracking policy kicks in
5. Once the stress command ends and CPU drops, watch the ASG scale back in after the cooldown period

---

## Cleanup (important, to avoid ongoing charges)

Delete resources in this order to avoid dependency errors:

1. Delete the **Application Load Balancer**
2. Delete the **Auto Scaling Group** (this terminates its instances automatically)
3. Delete the **Target Group**
4. Delete the **Launch Template**
5. Confirm in **EC2 > Instances** that nothing is still running

---

## Key Takeaways

- A launch template is the blueprint, an ASG is the fleet manager, and an ALB is the traffic director. Each does one job well, and they're wired together through the target group.
- ELB health checks matter more than EC2 status checks alone - they let your ASG replace instances that are technically running but not actually serving traffic correctly.
- Target tracking scaling policies let you set a goal (like 50 percent CPU) rather than manually defining thresholds, and AWS handles the rest.

#AwsStudentbuilders
