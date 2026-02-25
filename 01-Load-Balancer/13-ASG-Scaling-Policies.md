# Auto Scaling Group (ASG) Scaling Policies

While an Auto Scaling Group defines *what* to scale (the EC2 instance type via a Launch Template) and *where* to scale (the Subnets), **Scaling Policies** define exactly **WHEN** and **HOW MUCH** to scale.

There are several scaling strategies you can employ depending on the predictability and jaggedness of your application traffic.

---

### 1. Dynamic Scaling Policies (Reacting to Traffic)

Dynamic scaling policies rely on Amazon CloudWatch alarms to continuously monitor metrics and adjust capacity on the fly.

#### A. Target Tracking Scaling (The Recommended Default)
*   **How it works:** You select a target metric, and AWS does the math for you. It acts exactly like a thermostat in a house. You set the thermostat to 72°F; if it gets too hot, the AC turns on. If it gets too cold, the heat turns on.
*   **Common Metrics:** Average CPU utilization, Average Network In/Out, or `RequestCountPerTarget` (how many requests the Application Load Balancer is sending to each instance).
*   **Scenario:** You set Target Tracking to keep Average CPU at **50%**.
    *   Traffic spikes -> CPU hits **75%** -> ASG calculates how many instances it needs to add to bring the average back down to 50% -> It scales out.
    *   Traffic drops -> CPU hits **20%** -> ASG calculates how many it can terminate while keeping the average near 50% -> It scales in.

#### B. Step Scaling (For Jagged/Spiky Workloads)
*   **How it works:** You define specific "steps" based on alarm thresholds. It allows you to respond more aggressively to massive spikes than you would to minor bumps in traffic.
*   **Scenario:**
    *   *Step 1:* If CPU > 60%, add **1** instance.
    *   *Step 2:* If CPU > 85%, add **5** instances immediately! (Because a massive spike means the site is about to crash).
*   **The Advantage:** Step scaling allows you to define complex, custom multi-tiered responses.

#### C. Simple Scaling (Legacy)
*   **How it works:** Extremely basic. If a CloudWatch alarm breaches, perform exactly one action (e.g., if CPU > 80%, add 1 instance).
*   **The Problem:** Simple Scaling triggers a **Cooldown Period** (see below) where the ASG *locks up* and completely ignores any new alarms until the cooldown expires. **Step Scaling is heavily preferred over Simple Scaling.**

---

### 2. Predictable Scaling Policies

What if you *know* when traffic is going to hit? Waiting for CPU to spike (Dynamic Scaling) means your users will experience slow response times while the new EC2 instances boot up.

#### A. Scheduled Scaling (For Known Events)
*   **How it works:** You configure a schedule (one-time or recurring via Cron jobs) to change the Minimum, Maximum, or Desired capacity ahead of time.
*   **Scenario (The Black Friday Event):** You run an e-commerce store with an upcoming Black Friday sale at midnight.
    *   You schedule an action for 11:30 PM on Thursday to set the `Desired Capacity` to **50**.
    *   By the time midnight hits, the 50 servers are already booted up, initialized, and ready to handle the stampede of users. Zero latency.
*   **Scenario (The 9-to-5 Office App):** Scale out the ASG at 8:00 AM every weekday. Scale it all the way down to Min=1 at 6:00 PM to save costs over the night and weekend.

#### B. Predictive Scaling (Machine Learning)
*   **How it works:** You don't know the exact schedule, but AWS does. Predictive Scaling uses Machine Learning to analyze minimum 2 weeks of historical CloudWatch traffic data. It recognizes daily or weekly patterns and automatically schedules scaling actions *ahead* of the predicted curve.
*   *Note: Predictive Scaling is often combined with Target Tracking. Predictive handles the expected spikes, and Target Tracking handles any sudden, unexpected bursts.*

---

### 3. Crucial Exam Concept: The Cooldown Period

A major issue in auto-scaling is **"Thrashing."** 
1. CPU hits 80%.
2. The ASG launches 2 instances.
3. *One minute later,* CPU is still at 80% (because the new instances are still installing software and haven't started processing requests yet). 
4. The ASG panics and launches 2 *more* instances.

**The Solution is the Cooldown Period (Default: 300 seconds / 5 minutes).**
*   After an ASG launches or terminates an instance, it enters a Cooldown Period.
*   During this time, it pauses and waits for the metrics to stabilize. It basically says, *"I just added servers; let's give them 5 minutes to boot up and reduce the load before I decide if I need to add even more."*

---

### 4. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A news publishing website gets moderate traffic most days. However, when a major breaking news story is published, traffic spikes incredibly fast, and the web servers often become overwhelmed before the Target Tracking scaling policy can add new instances. The DevOps team wants a scaling solution that reacts faster and more aggressively when CPU utilization suddenly spikes past 90%. Which scaling policy should they implement?**
*   **A)** Simple Scaling
*   **B)** Step Scaling
*   **C)** Scheduled Scaling
*   **D)** Predictive Scaling
*   **Answer:** **B**. Step Scaling allows you to define "steps"—meaning you can tell the ASG to add just 1 instance if CPU hits 70%, but to aggressively launch 5 instances immediately if CPU hits 90%. (Scheduled and Predictive won't work for unpredictable breaking news, and Simple Scaling is too basic and susceptible to cooldown lockups).

**Q2: A company's internal payroll application runs on EC2 instances in an Auto Scaling Group. The application experiences almost no traffic throughout the week, but every Friday between 3:00 PM and 5:00 PM, thousands of employees log in simultaneously to submit their timesheets. Employees frequently complain that the application is extremely slow right at 3:00 PM. How can a Solutions Architect fix this issue most cost-effectively?**
*   **A)** Use Target Tracking to keep Average CPU Utilization at 40%.
*   **B)** Keep the Minimum Capacity of the ASG at 20 instances 24/7.
*   **C)** Create a Scheduled Scaling policy to increase the Desired Capacity to 20 instances every Friday at 2:30 PM.
*   **D)** Use Predictive Scaling backed by Machine Learning.
*   **Answer:** **C**. Because this is a *known, highly predictable* event tied to a specific time, a Scheduled Action to scale out 30 minutes *before* the rush ensures the servers are booted up and ready before any employees actually log in, completely eliminating the 3:00 PM slowness. Keeping 20 instances running 24/7 (B) would be a massive waste of money.

**Q3: An Auto Scaling Group (ASG) uses Simple Scaling. A CloudWatch alarm triggers a scale-out event, and a new EC2 instance is launched. Two minutes later, the CPU utilization is still above the alarm threshold, but the ASG does not launch any additional instances. What is the most likely reason for this behavior?**
*   **A)** The ASG has reached its Desired Capacity limit.
*   **B)** The ASG has reached its Minimum Capacity limit.
*   **C)** The instance is still initializing and the ASG is waiting for the ELB Health Check to pass.
*   **D)** The ASG is currently in its Cooldown Period.
*   **Answer:** **D**. The Cooldown Period (default 300 seconds) prevents the ASG from launching or terminating additional instances until the previous scaling activity has taken effect and the metrics have stabilized. Simple Scaling policies are subject to this cooldown.
