# Auto Scaling Groups (ASG)

In the real world, application traffic is rarely constant. A website might have 100 users at 3 AM but 10,000 users at 3 PM. 

An **Auto Scaling Group (ASG)** is a logical grouping of EC2 instances that automatically adjusts its size—adding or removing instances—based on the real-time demands of your application.

---

### 1. What is an Auto Scaling Group? (The Core Rules)

At its absolute core, an ASG ensures that you have the correct number of EC2 instances running to handle your load. It operates based on three defined rules:

*   **Minimum Capacity:** The absolute lowest number of instances that must be running (e.g., 2). *Even if traffic is zero, the ASG will maintain 2 instances.*
*   **Maximum Capacity:** The absolute highest number of instances allowed to run (e.g., 10). *This protects you from a massive unexpected AWS bill.*
*   **Desired Capacity:** The number of instances you *want* to be running right now (e.g., 4). The ASG will immediately launch or terminate instances to match this number. (*Note: Min <= Desired <= Max*).

---

### 2. Why and Where Do We Use It? (The Scenarios)

**Why do we use ASGs?**
1.  **Cost Optimization (Scaling In):** If traffic drops at midnight, the ASG automatically terminates idle EC2 instances. You stop paying for servers you aren't using.
2.  **Performance (Scaling Out):** If a marketing campaign goes viral, the ASG automatically launches new EC2 instances to handle the traffic spike before the website crashes.
3.  **High Availability & Fault Tolerance:** If an EC2 instance crashes (fails a health check), the ASG automatically terminates the broken instance and launches a brand new one to replace it, without any human intervention.

**Where do we use ASGs?**
ASGs span across **Availability Zones (AZs)**. A best practice is to deploy your ASG across at least 2 or 3 Multi-AZ subnets. If an entire AWS data center (AZ) goes offline, the ASG will automatically recreate the lost instances in the remaining healthy AZs.

---

### 3. How Does It Work? (The Architecture)

An ASG doesn't "guess" how to build a server. It relies on a few core components:

#### A. The Launch Template
Before an ASG can build a new server, it needs a blueprint. A **Launch Template** provides this blueprint:
*   Which AMI (Operating System) to use?
*   Which Instance Type (e.g., `t3.micro`)?
*   Which Security Groups to attach?
*   What User Data script to run on boot (e.g., install Nginx, download code)?

#### B. CloudWatch Alarms
ASGs are usually managed by **CloudWatch Alarms**. 
*   *Scenario:* You create an alarm for "CPU > 70% for 5 minutes". If that alarm triggers, it tells the ASG: "Scale Out!"
*   *Scenario:* "CPU < 20% for 15 minutes". It tells the ASG: "Scale In!"

#### C. Load Balancer Integration
ASGs are almost always deployed *behind* a Load Balancer (ALB/NLB). 
*   When the ASG launches a new instance, it **automatically registers** that instance with the Load Balancer's Target Group.
*   When it terminates an instance, it **automatically deregisters** it so the Load Balancer stops sending it traffic.
*   **Exam Tip:** The ASG can use the Load Balancer's (ELB) Health Checks to determine if an instance is healthy, rather than just relying on the basic EC2 status checks.

---

### 4. Types of Scaling Policies (How we trigger scaling)

To automate the "Desired Capacity," you attach a Scaling Policy to the ASG.

#### A. Target Tracking Scaling (The Easiest & Most Common)
*   **Concept:** You pick a metric, and the ASG does the math to keep it there. It works exactly like a thermostat in your house.
*   **Example:** "Keep the Average CPU Utilization of my ASG at exactly 50%." If CPU hits 70%, it adds instances. If CPU hits 30%, it removes instances.

#### B. Step/Simple Scaling (The "If-This-Then-That" Rule)
*   **Concept:** You define specific steps based on CloudWatch Alarms.
*   **Example:** 
    *   If CPU > 60%, add 1 instance.
    *   If CPU > 85%, add 3 instances.

#### C. Scheduled Scaling (The Predictable Event)
*   **Concept:** You know *exactly* when traffic will hit based on time and date.
*   **Scenario:** You run a Payroll application that gets slammed every Friday at 5:00 PM. You set a schedule: Every Friday at 4:30 PM, set Minimum Capacity to 10. Every Friday at 8:00 PM, set Minimum to 2.

#### D. Predictive Scaling (Machine Learning)
*   **Concept:** AWS uses Machine Learning to analyze your historical traffic patterns spanning weeks or months, and automatically predicts and scales *ahead* of the curve.

---

### 5. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A company's website is hosted on EC2 instances behind an Application Load Balancer (ALB). The instances are part of an Auto Scaling Group (ASG) deployed across three Availability Zones. During a massive marketing event, traffic suddenly spikes 500% in a matter of seconds, causing the website to crash before the ASG can launch new instances. How can a Solutions Architect prevent this in the future during *planned* marketing events?**
*   **A)** Change the scaling policy to use Target Tracking for CPU utilization.
*   **B)** Create a Scheduled Scaling policy to increase the baseline capacity 30 minutes before the marketing event.
*   **C)** Use Predictive Scaling to analyze the traffic.
*   **D)** Switch from an Application Load Balancer to a Network Load Balancer.
*   **Answer:** **B**. If an event is *planned* (you know the exact date and time), the best approach is **Scheduled Scaling**. It proactively launches instances *before* the traffic hits, completely avoiding the boot-up delay that caused the crash.

**Q2: A legacy application running on EC2 instances frequently locks up due to memory leaks. The EC2 instance status checks report the instances as "Healthy" (because the OS is running), but the web application actually returns HTTP 500 errors to users. The instances are in an Auto Scaling Group behind an Application Load Balancer. How can you ensure the locked instances are automatically replaced?**
*   **A)** Create a script on the EC2 instances to reboot the server daily.
*   **B)** Configure the Auto Scaling Group to use ELB Health Checks instead of EC2 Health Checks.
*   **C)** Change the ASG scaling policy to Step Scaling based on memory utilization.
*   **D)** Enable Cross-Zone Load Balancing on the ALB.
*   **Answer:** **B**. EC2 health checks only check if the virtual machine is powered on. ELB (Load Balancer) health checks actually "ping" the application (e.g., checking it returns a 200 OK). By tying the ASG to the ELB Health Checks, the ASG will terminate the instance when the app starts returning 500 errors.

**Q3: A Solutions Architect needs to design an architecture that maintains exactly 6 EC2 instances at all times to process background queue jobs. The architecture must be highly available and resilient to an entire Availability Zone failure. Which configuration best meets these requirements?**
*   **A)** Deploy 3 EC2 instances in AZ-A and 3 in AZ-B.
*   **B)** Create an Auto Scaling Group spanning 3 Availability Zones with Min=6, Max=6, and Desired=6.
*   **C)** Deploy an Application Load Balancer distributing traffic to 6 instances in a single AZ.
*   **D)** Use a Scheduled Scaling policy to verify the count of 6 instances every hour.
*   **Answer:** **B**. Setting Min=Max=Desired to 6 guarantees exactly 6 instances will run at all times. Spanning the ASG across multiple AZs provides the highest level of fault tolerance; if an AZ goes down, the ASG automatically shifts and rebuilds the instances in the surviving AZs to maintain the desired count of 6.

---

> [!TIP]
> **Exam Strategy:** On the AWS SAA exam, if a question mentions fixing an "unhealthy or unresponsive" instance automatically without human intervention, the service you need is an **Auto Scaling Group**. If the question asks how to handle a "predictable, known" event (like Black Friday), look for **Scheduled Scaling**.
