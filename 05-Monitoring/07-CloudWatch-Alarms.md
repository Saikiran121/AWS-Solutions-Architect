# Amazon CloudWatch Alarms

If **Metrics** are the "eyes" of AWS (watching what happens) and **Logs** are the "memory" (recording what happened), then **Alarms** are the "muscles."

A CloudWatch Alarm watches a single CloudWatch Metric. If that metric crosses a specific threshold you define, the alarm triggers an automated action.

---

### 1. The Three Alarm States

At any given moment, a CloudWatch Alarm is in exactly one of three states:

1.  **`OK`**: The metric is within the defined, acceptable threshold. (e.g., CPU is currently at 40%, and the alarm threshold is 80%).
2.  **`ALARM`**: The metric has breached the defined threshold. (e.g., CPU has spiked to 95%). This is the state that triggers an action.
3.  **`INSUFFICIENT_DATA`**: The alarm does not have enough information to make a decision. This usually happens when:
    *   The alarm was *just* created and hasn't collected its first data point yet.
    *   The EC2 instance being monitored is turned off.
    *   It is a custom metric (like 404 Error counts), and there simply hasn't been any traffic to generate data.

---

### 2. Period and Datapoints to Alarm (Anti-Flapping)

Nobody wants to be woken up at 3:00 AM because a server's CPU spiked to 90% for exactly two seconds and then immediately returned to normal. To prevent this "flapping" (false alarms), CloudWatch forces you to define *duration*.

When configuring an alarm, you define:
1.  **Period:** How often the metric is evaluated (e.g., Every 1 minute, or every 5 minutes).
2.  **Datapoints to Alarm:** How many of those consecutive periods must be in breach before the alarm actually fires.

*Example:* You set the alarm to trigger if CPU > 80% for **3 out of 3** consecutive **5-minute** periods. 
*   If the CPU spikes to 100% for 9 minutes and then drops back down, the alarm will stay in the `OK` state. It *must* stay elevated for the full 15 minutes (3 periods of 5 minutes) to trigger the `ALARM` state.

---

### 3. Alarm Actions (What happens when it triggers?)

When an alarm enters the `ALARM` state (or even if it enters the `OK` or `INSUFFICIENT_DATA` states), it can automatically trigger specific AWS services. For the SAA-C03 exam, you must memorize these three primary native targets:

#### A. Amazon SNS (Simple Notification Service)
This is the most common action. The alarm sends a message to an SNS Topic. SNS takes that message and fan-outs notifications to human engineers (via Email or SMS) or triggers an AWS Lambda serverless function to attempt an automated code fix.

#### B. Amazon EC2 Auto Scaling
Alarms are the invisible engine behind Auto Scaling.
*   **Scale Out:** An alarm watching for CPU > 80% triggers an Auto Scaling policy to add 2 new EC2 instances to the fleet.
*   **Scale In:** An alarm watching for CPU < 20% triggers a policy to terminate 2 instances to save money.

#### C. EC2 Actions & Deep-Dive into EC2 Recovery (🚨 Highly Tested!)
Instead of scaling or texting a human, CloudWatch Alarms can execute actions directly on a specific EC2 instance: `Stop`, `Terminate`, `Reboot`, or `Recover`.

**The EC2 `Recover` Action Deep-Dive:**
EC2 Recovery is one of the most frequently tested concepts on the SAA-C03 exam regarding high availability for legacy applications.

*   **The Problem:** AWS physical servers (hosts) occasionally break. Power supplies fail, motherboards fry. When this happens, your EC2 instance fails exactly 1 of its 2 status checks: the **System Status Check** (`StatusCheckFailed_System`), meaning the underlying AWS hardware is broken (it is not a problem with your OS or code).
*   **The CloudWatch Solution:** You can create a CloudWatch Alarm watching the `StatusCheckFailed_System` metric. If it triggers `ALARM`, you configure the action to **Recover this instance**.
*   **The Magic:** AWS will automatically "teleport" your EC2 instance from the broken physical hardware to a brand-new, healthy rack in the data center.

**Why is Recover different from Auto Scaling? (The Crucial Exam Distinction)**
If an Auto Scaling Group replaces a broken instance, it terminates the old one and launches a *brand new instance*. The new instance gets a *new* Instance ID and a *new* Private IP address.

When CloudWatch **Recovers** an instance, the rebooted instance **RETAINS EVERYTHING**:
1.  Identical **Instance ID** (`i-1234567890abcdef0`).
2.  Identical **Private IP Address** and Elastic IP Address.
3.  Identical **EBS Volume Data** (because EBS volumes are network-attached drives, AWS simply detaches the drive from the broken physical host and reattaches it to the new physical host).
*(Note: Data on temporary "Instance Store" volumes is lost because it was physically attached to the broken host).*

**Real-World Scenario: Legacy Software Licensing**
*   **Scenario:** Your company uses an expensive, 15-year-old accounting software. The vendor tied the software license permanently to the server's specific MAC Address and Private IP address. If the Private IP changes, the software breaks and the company shuts down.
*   **How to protect it:** You cannot use an Auto Scaling Group, because scaling would change the IP and void the license. Instead, you create a CloudWatch Alarm to `Recover` the instance. If the AWS hardware dies in the middle of the night, CloudWatch reboots the instance on healthy hardware with the *exact same IP*, keeping the legacy license perfectly intact.

---

### 4. Composite Alarms & Deep-Dive (Reducing Alert Fatigue)

As your AWS infrastructure grows, you might have hundreds of individual CloudWatch Alarms. When a major component fails, it often triggers a cascade of alarms.

Imagine a scenario where a master database crashes.
1.  The Database CPU alarm fires.
2.  The Web Server connection timeout alarm fires.
3.  The API Gateway 500 error alarm fires.
*(Suddenly, the on-call engineer receives 50 text messages at exactly 2:00 AM for a single root cause).*

To prevent this "alert fatigue" and help teams focus on the actual problem, you use **Composite Alarms**.

**How Composite Alarms Work:**
*   A Composite Alarm does *not* watch a metric directly (like CPU or Network). Instead, it watches the state of **other Alarms**.
*   It combines the states of underlying standard alarms using Boolean logic (`AND` / `OR` / `NOT`).

**Real-World Scenario: The E-Commerce Black Friday Outage**
*   **The Problem:** During Black Friday, an e-commerce website experiences a massive surge in traffic. The Web Server CPU naturally spikes to 95%, triggering a "High CPU Alarm." However, the Auto Scaling Group handles the load perfectly, and customers are still ordering successfully. A junior engineer gets the CPU alert, panics, and wakes up the senior team unnecessarily.
*   **The Composite Solution:** The team creates a Composite Alarm to ensure they only panic when a business metric is actually impacted. They define the logic as:
    `Trigger ALARM state IF (WebServer-High-CPU == ALARM) AND (Application-Checkout-Errors == ALARM)`
*   **The Result:** Now, if the CPU spikes but the checkout process is still working fine, the Composite Alarm remains perfectly quiet (`OK` state). The engineers are *only* paged if the CPU spikes AND customers are actually failing to checkout simultaneously. This guarantees that every page they receive is a critical, verified emergency.

**Key Exam Takeaways:**
*   Underlying alarms can be standard metric alarms or even other composite alarms.
*   By aggregating alarms via boolean logic, you drastically reduce the number of Amazon SNS notifications sent, saving operations engineers from "alert fatigue."

---

### 5. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: An application runs on a single Amazon EC2 instance that holds a software license tied permanently to its specific Elastic IP address and Instance ID. The application is highly critical, and if the underlying physical AWS host hardware fails, the application must be brought back online as quickly as possible without losing the IP address. How can a Solutions Architect automate this failover process?**
*   **A)** Place the EC2 instance in an Auto Scaling Group with a minimum and maximum size of 1.
*   **B)** Create an AWS Lambda function triggered by Amazon EventBridge to launch a replacement instance with the same Elastic IP.
*   **C)** Create a CloudWatch Alarm that monitors the `StatusCheckFailed_System` metric and configure an EC2 `Recover` action.
*   **D)** Use AWS Backup to continuously mirror the instance to a secondary Availability Zone.
*   **Answer:** **C**. The EC2 `Recover` action via CloudWatch Alarms is specifically designed for physical hardware failures. It migrates the instance to healthy AWS hardware while retaining the exact same Instance ID, Private IP, Elastic IP, and EBS-backed data. Using an ASG (A) would launch a *brand new* instance with a *new* Instance ID, breaking the software license.

**Q2: A company's operations team is suffering from "alert fatigue." Whenever a minor network blip occurs, three different CloudWatch Alarms (monitoring CPU, Network In, and Application Errors) independently trigger Amazon SNS notifications, burying the team in emails. The team only wants to be notified if ALL THREE alarms trigger simultaneously, indicating a true system outage. How should this be implemented?**
*   **A)** Configure the three standard CloudWatch Alarms to send their notifications to a single, consolidated Amazon SQS queue.
*   **B)** Create a CloudWatch Composite Alarm that uses `AND` logic to combine the states of the three existing alarms. Configure only the Composite Alarm to send the SNS notification.
*   **C)** Group the three metrics into a new CloudWatch Dashboard and configure the dashboard to send anomaly alerts.
*   **D)** Increase the "Datapoints to Alarm" evaluation period on all three alarms to 30 minutes.
*   **Answer:** **B**. Composite Alarms are explicitly designed to reduce alarm noise. By combining multiple existing alarms using boolean logic (like `AND`), you ensure that a notification is only generated when a complex set of conditions are met simultaneously.

**Q3: A Solutions Architect has configured a CloudWatch Alarm to monitor the CPU Utilization of a production EC2 instance. The architect wants to ensure the alarm only triggers an action if the CPU remains above 90% for a sustained period of 15 minutes, ignoring brief, temporary spikes caused by cron jobs. How should the alarm's evaluation parameters be configured to achieve this?**
*   **A)** Set the Threshold to 90%, the Period to 15 minutes, and Datapoints to Alarm to 1 out of 1.
*   **B)** Set the Threshold to 90%, the Period to 1 minute, and Datapoints to Alarm to 1 out of 15.
*   **C)** Set the Threshold to 90%, the Period to 5 minutes, and Datapoints to Alarm to 3 out of 3.
*   **D)** Use a High-Resolution custom metric with a 1-second period and an evaluation period of 900 seconds.
*   **Answer:** **C**. To prevent flapping on brief spikes, you require multiple consecutive evaluation periods. Setting the period to 5 minutes and requiring 3 out of 3 data points to be in breach ensures the CPU has to stay elevated for the full 15 consecutive minutes (3 * 5 = 15) before the alarm state changes. Option A evaluates a single 15-minute average, which could be skewed by a massive 1-minute spike within that window.
