# Amazon CloudWatch Metrics

At its core, **Amazon CloudWatch** is a metrics repository. An AWS service—such as Amazon EC2 or an Application Load Balancer—puts metrics into the repository, and you retrieve statistics based on those metrics.

A **Metric** is simply a time-ordered set of data points. For example, the CPU utilization of a specific EC2 instance tracked every minute for a week is a single metric.

---

### 1. The Anatomy of a Metric

To understand CloudWatch, you have to understand how AWS organizes millions of data points across thousands of customers. They do this using Namespaces and Dimensions.

#### A. Namespaces (The Container)
A **Namespace** is a container for CloudWatch metrics. It essentially acts as a top-level folder to ensure that metrics from different AWS services don't accidentally mix together.
*   There is no default namespace.
*   AWS native services use namespaces that begin with `AWS/` (e.g., `AWS/EC2`, `AWS/S3`, `AWS/ApplicationELB`).
*   When you create a custom metric (from your own application code), you give it a custom namespace name (e.g., `MyEcommerceApp/Purchases`). You cannot use the `AWS/` prefix for custom metrics.

#### B. Dimensions (The Unique Identifier)
A **Dimension** is a name/value pair that is part of the identity of a metric. You can have up to 30 dimensions assigned to a single metric.
*   *Why do we need them?* If you just look up the metric `CPUUtilization` in the `AWS/EC2` namespace, CloudWatch won't know *which* server you are talking about.
*   You use Dimensions to filter the data. 
    *   Example Dimension: `InstanceId = i-1234567890abcdef0`
    *   Example Dimension: `AutoScalingGroupName = WebServersASG`
*   If you change the Dimensions, you are essentially creating or looking at an entirely different, unique metric.

---

### 2. Standard vs. High-Resolution Metrics

This is a critical concept, especially when tying Metrics to Auto Scaling Groups.

*   **Standard Resolution:** By default, several AWS services (like EC2) publish metrics every **5 minutes** (300 seconds). For custom metrics, you can publish down to every **1 minute** (60 seconds).
*   **High-Resolution:** If your application experiences massive traffic spikes in seconds, waiting 1 to 5 minutes for CloudWatch to trigger an Auto Scaling alarm is too slow. You can publish High-Resolution Custom Metrics that report data every **1 second**.
    *   *Note: High-Resolution metrics cost more money.*

---

### 3. The Ultimate Exam Trap: Default vs. Custom EC2 Metrics

This is one of the most consistently tested concepts on the SAA-C03 exam.

When you boot up a standard Amazon EC2 instance, AWS automatically provides **Host-Level Metrics** to CloudWatch out of the box. Because AWS manages the hypervisor and the underlying physical server hardware (the Host), it can easily "see" certain things without needing your permission:
*   **CPU Utilization**
*   **Network Utilization (In/Out)**
*   **Disk Read/Write (I/O)**
*   **Status Checks** (Is the underlying hardware broken?)

#### The Catch: AWS Cannot See Inside Your OS
Because of the AWS Shared Responsibility Model, AWS has absolutely zero access to the inside of your Operating System (Linux/Windows). Therefore, **AWS DOES NOT provide the following metrics by default:**
*   **Memory (RAM) Utilization**
*   **Available Disk Storage Space (e.g., "Is the C: drive full?")**
*   **Swap Usage**

#### The Solution: The CloudWatch Agent
To get Memory or free disk space metrics, you must install a piece of software called the **CloudWatch Agent** inside your EC2 instance's Operating System. 
1. The Agent runs inside the OS.
2. It looks at the RAM/Disk usage.
3. It pushes that data to CloudWatch as a **Custom Metric**.

---

### 4. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A company has migrated their on-premises database to an Amazon EC2 instance. The Database Administrator has requested that an alarm be triggered if the memory (RAM) utilization of the instance exceeds 85%. When the Solutions Architect goes to configure the CloudWatch alarm, they cannot find the Memory metric for the EC2 instance. What must the Architect do to resolve this?**
*   **A)** Enable Detailed Monitoring on the EC2 instance via the AWS Management Console to expose the Memory metric.
*   **B)** Install and configure the unified CloudWatch Agent on the EC2 instance to push Memory utilization as a custom metric.
*   **C)** Change the instance type, as Memory metrics are only available by default on Memory-Optimized (R-family) instance types.
*   **D)** Use AWS CloudTrail to track the internal RAM events.
*   **Answer:** **B**. By default, Amazon EC2 does not provide Memory (RAM), free disk space, or Swap usage metrics to CloudWatch because AWS cannot see inside the guest Operating System. You must install the CloudWatch Agent to capture OS-level metrics and push them as Custom Metrics. (Enabling "Detailed Monitoring" only changes the frequency of default metrics from 5 minutes to 1 minute, it does not add RAM).

**Q2: A high-frequency trading application running on Amazon EC2 requires extremely rapid scaling to handle millisecond fluctuations in market data. The current Auto Scaling Group relies on a Custom Metric published every 1 minute, but the scaling response is too slow. How can a Solutions Architect improve the scaling reaction time?**
*   **A)** Enable Detailed Monitoring on the EC2 instances.
*   **B)** Transition the backend instances to a Network Load Balancer (NLB) instead of an Application Load Balancer.
*   **C)** Publish the Custom Metric as a High-Resolution metric with a 1-second interval, and attach a high-resolution CloudWatch Alarm.
*   **D)** Use the `AWS/EC2` namespace for the custom metric instead of a custom namespace.
*   **Answer:** **C**. Standard custom metrics have a minimum resolution of 1 minute (60 seconds). For sub-minute scaling requirements, you must publish the metric as a High-Resolution metric, which supports a 1-second interval, enabling much faster Auto Scaling alarm triggers.

**Q3: A developer has created a custom application that counts the number of times a user clicks a "Donate" button. They are writing code using the AWS SDK to push this data to Amazon CloudWatch as a metric. Which of the following is a VALID namespace the developer can use for this metric?**
*   **A)** `AWS/Custom`
*   **B)** `AWS/Donations`
*   **C)** `MyOrganisation/App/Donations`
*   **D)** `AWS/EC2/Donations`
*   **Answer:** **C**. You cannot use the `AWS/` prefix for custom namespaces. The `AWS/` prefix is strictly reserved for AWS native services (like `AWS/EC2`, `AWS/S3`). Therefore, `MyOrganisation/App/Donations` is the only valid option listed.

---

> [!CAUTION]
> **Exam Strategy:** The moment you see the words **"Memory"** or **"RAM"** tracking for an EC2 Instance on the SAA exam, your brain should immediately lock onto the answer containing the words **"CloudWatch Agent"** or **"Custom Metric."**
