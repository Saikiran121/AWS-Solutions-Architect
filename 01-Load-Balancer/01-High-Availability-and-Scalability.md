# 01. High Availability and Scalability

## 1. AWS Scalability: An In-Depth Guide

Scalability is the ability of a system to handle increased load by adding resources to the infrastructure. In AWS, scalability is not just about growing; it's about the ability to grow (and shrink) seamlessly without impacting performance or reliability.

### What is AWS Scalability?
Scalability in AWS refers to the ability of your application to accommodate growth in usage, such as more users, more data, or more transactions, by dynamically adjusting the underlying computing capacity. AWS provides a suite of services (like Auto Scaling, ELB, and RDS Read Replicas) that allow infrastructure to scale automatically based on demand.

### How It Works: The Mechanics of Scaling
AWS Scalability relies on several core components working in harmony:

1.  **Monitoring (Amazon CloudWatch):** Collects metrics like CPU utilization, memory usage, or request counts.
2.  **Logic (Auto Scaling Groups):** Defines the "policies" for when to scale (e.g., "Add an instance if CPU exceeds 70% for 5 minutes").
3.  **Distribution (Elastic Load Balancing):** Ensures that the new traffic is evenly distributed across the newly created resources.
4.  **Resource Provisioning:** AWS automatically launches or terminates EC2 instances, or adjusts capacity for services like DynamoDB or Lambda.

---

### Types of Scalability: Vertical vs. Horizontal

Understanding the difference between these two is critical for architecting resilient systems.

#### A. Vertical Scalability (Scaling "Up")
Vertical scaling involves increasing the power of an existing resource (e.g., upgrading an EC2 instance from a `t3.micro` to a `m5.large`).

*   **How it works:** You replace the existing hardware with a more powerful one (more CPU, RAM, or I/O).
*   **Pros:** Simple to implement; no architectural changes to the application are usually required.
*   **Cons:**
    *   **Hard Limit:** You eventually hit the maximum size of the largest available instance.
    *   **Downtime:** Usually requires a reboot to change instance types (though some services minimize this).
    *   **Single Point of Failure:** You still have only one instance.
*   **Best Use Case:** Small databases or applications that aren't designed to run across multiple servers.

#### B. Horizontal Scalability (Scaling "Out")
Horizontal scaling involves adding more resources of the same size to your pool (e.g., adding 5 more `t3.micro` instances).

*   **How it works:** You distribute the load across a fleet of instances.
*   **Pros:**
    *   **Limitless Growth:** You can theoretically add an infinite number of instances.
    *   **High Availability:** If one instance fails, others continue to serve traffic.
    *   **Cost-Effective:** You can use many small, cheap instances rather than one massive, expensive one.
*   **Cons:** Requires the application to be **stateless** (sessions shouldn't be stored locally on the server).
*   **Best Use Case:** Web applications, microservices, and modern distributed systems.

---

### Summary Comparison Table

| Feature | Vertical Scalability (Up) | Horizontal Scalability (Out) |
| :--- | :--- | :--- |
| **Strategy** | Increase capacity of one instance | Increase the number of instances |
| **Effort** | Low (Change instance type) | Moderate (Requires Load Balancer) |
| **Redundancy** | Low (Single instance) | High (Multiple instances) |
| **Limits** | Hardware Upper Bound | Virtually Unlimited |
| **Downtime** | Often Required | Zero Downtime |

---

### Industrial Examples of Scalability

#### 1. E-Commerce: The "Black Friday" Scenario (Horizontal)
An e-commerce giant like Amazon or Flipkart experiences a 10x spike in traffic during a 24-hour sale.
*   **Implementation:** They use **AWS Auto Scaling** with **Horizontal Scaling**. As the request count per second (RPS) increases, CloudWatch triggers the Auto Scaling group to launch hundreds of additional EC2 instances. Once the sale ends, the system automatically terminates those instances to save costs.

#### 2. Streaming Services: Netflix (Horizontal)
Netflix uses AWS to serve millions of concurrent video streams globally.
*   **Implementation:** Their microservices architecture relies heavily on Horizontal Scalability. Each component (recommendation engine, login service, playback service) scales independently based on its specific load, ensuring that a surge in one region doesn't affect others.

#### 3. Legacy Banking Systems (Vertical)
A traditional bank running a monolithic core banking application on a large database.
*   **Implementation:** Because the application is stateful and difficult to distribute, they might use **Vertical Scaling**. When the end-of-month reporting starts, they might scale the RDS instance type from an `r5.4xlarge` to an `r5.12xlarge` to handle the massive I/O and memory requirements for those few hours.

#### 4. Gaming Industry (Horizontal)
Online multiplayer games like *Fortnite* or *PUBG*.
*   **Implementation:** When a new season starts, millions of players log in simultaneously. Horizontal scaling allows the game servers to spin up thousands of specialized EC2 instances (often using G-series for GPU or C-series for Compute) to host individual game sessions, then shut them down as players log off.

---

## 2. High Availability (HA): Ensuring Maximum Uptime

While Scalability handles *load*, **High Availability (HA)** handles *failure*. It ensures that your system remains operational and accessible to users even when individual components or entire data centers (Availability Zones) fail.

### The "Rules of 9s"
Availability is usually measured as a percentage of uptime over a year.

| Availability | Yearly Downtime | Business Impact |
| :--- | :--- | :--- |
| **99% (Two 9s)** | 3.65 days | Standard websites, non-critical apps. |
| **99.9% (Three 9s)** | 8.77 hours | Most SaaS and consumer apps. |
| **99.99% (Four 9s)** | 52.56 minutes | E-commerce, Banking, Critical Cloud services. |
| **99.999% (Five 9s)** | 5.26 minutes | Telecommunications, Emergency services. |

---

### How High Availability Works in AWS
AWS achieves HA by eliminating **Single Points of Failure (SPOF)**.

1.  **Multi-AZ Deployments:** Spreading resources (EC2, RDS) across multiple Availability Zones. If one AZ goes down due to a power outage or natural disaster, the others continue to serve traffic.
2.  **Elastic Load Balancing (ELB):** Constantly performs **Health Checks**. If an instance fails, ELB automatically stops sending traffic to it and reroutes it to healthy instances in other AZs.
3.  **Amazon Route 53:** Provides DNS-level failover. If an entire region or primary endpoint is unreachable, Route 53 can reroute users to a secondary standby site.
4.  **Auto Healing:** Auto Scaling groups don't just scale; they also replace "unhealthy" instances. If an instance crashes, ASG terminates it and launches a fresh one to maintain the "desired capacity."

---

### Crucial Distinctions: HA vs. Fault Tolerance vs. Scalability

*   **Scalability:** The ability to handle *more* users (Performance).
*   **High Availability:** The ability to stay *online* despite failures (Uptime).
*   **Fault Tolerance (FT):** A higher (and more expensive) standard than HA. FT implies **zero downtime** and **zero data loss** during a failure. HA might involve a few seconds or minutes of interruption during a failover, whereas FT is seamless.

---

### Disaster Recovery: RTO and RPO
When discussing HA, two metrics are essential:
*   **Recovery Time Objective (RTO):** How quickly must the system be back online? (Targeting time).
*   **Recovery Point Objective (RPO):** How much data loss can the business tolerate? (Targeting data).

---

### Industrial Examples of High Availability

#### 1. Investment Banking (HA + Fault Tolerance)
Large banks like Goldman Sachs or JPMorgan require High Availability for their trading platforms.
*   **Implementation:** They use **Multi-Region** architectures with **synchronous data replication**. If a primary region (e.g., US-East-1) fails, the secondary region (US-West-2) is already "hot" and takes over immediately. They aim for "Five 9s" because every second of downtime equals millions of dollars in lost trades.

#### 2. Healthcare: Electronic Health Records (EHR)
Systems like Epic or Cerner that hold patient records in hospitals.
*   **Implementation:** These systems use **Amazon RDS Multi-AZ**. If the primary database instance fails, AWS automatically fails over to a standby replica in a different AZ. This ensures doctors can always access life-saving patient data even during infrastructure glitches.

#### 3. Content Delivery: News Outlets (BBC, CNN)
During major global events, traffic spikes (Scalability) and uptime (Availability) are both critical.
*   **Implementation:** They use **Amazon CloudFront** (CDN) to cache content globally. Even if the origin web servers are struggling or undergoing maintenance in one region, the "Edge Locations" continue to serve the cached news articles to millions of readers.

#### 4. Payment Gateways (Stripe, PayPal)
*   **Implementation:** Payment processors use **Active-Active** configurations across multiple regions. Traffic is balanced globally. If one region's connectivity is degraded, the global load balancer (Route 53 + Global Accelerator) transparently shifts the load to the nearest healthy region, ensuring customers don't see "Payment Failed" errors.

---

## 3. High Availability and Scalability for EC2

Amazon EC2 (Elastic Compute Cloud) provides specialized features to ensure individual server instances can handle load and recovered from failure automatically.

### Scaling Strategies for EC2

1.  **Vertical Scaling (Resizing):** You can change the instance type (e.g., from `t2.micro` to `c5.large`) to give the server more CPU and RAM.
    *   **Pro:** Immediate performance boost for a single process.
    *   **Con:** Requires stopping the instance (downtime) and has a hardware ceiling.
2.  **Horizontal Scaling (Auto Scaling Groups):** The industry standard for modern apps. You add more identical instances to share the workload.

---

### Auto Scaling Groups (ASG) in Depth
An ASG is a collection of EC2 instances treated as a single logical unit. It uses a **Launch Template** (standard image, instance type, security groups) to spin up new servers.

*   **Capacity Management:**
    *   **Minimum Capacity:** The absolute fewest instances your app needs to run.
    *   **Desired Capacity:** The number of instances AWS tries to maintain at all times.
    *   **Maximum Capacity:** The upper limit to prevent runaway costs during a spike.

*   **Scaling Policies:**
    *   **Target Tracking Scaling:** The simplest method. You set a goal (e.g., "Keep average CPU at 40%") and AWS does the rest.
    *   **Step Scaling:** More granular. You define steps (e.g., "If CPU is 50-70%, add 1 instance; if 70-90%, add 3").
    *   **Scheduled Scaling:** Predictable. "Increase capacity to 20 instances every Friday at 4 PM for the weekend rush."
    *   **Predictive Scaling:** Uses Machine Learning to forecast traffic based on historical patterns and scales *before* the traffic arrives.

---

### High Availability Features for EC2

#### 1. Elastic Load Balancing (ELB) Integration
ELB works with ASG to distribute incoming traffic. It performs **Health Checks** on each instance. If an instance stops responding or returns 5xx errors, ELB stops sending traffic to it, and ASG automatically replaces it.

#### 2. Placement Groups
You can control where EC2 instances are physically located relative to each other:
*   **Cluster Placement Group:** Instances are packed in the same rack. Used for high-speed HPC (High Performance Computing). (High Performance, Low HA).
*   **Spread Placement Group:** Ensures each instance is on a separate rack/hardware. Used for small, critical sets of instances that must not fail together. (High HA).
*   **Partition Placement Group:** Divides instances into logical "partitions." Instances in one partition do not share hardware with other partitions. Common for HDFS and Cassandra.

#### 3. EC2 Instance Recovery
AWS monitors the hardware hosting your EC2. If the underlying hardware fails, a CloudWatch alarm can trigger an **Automated Recovery**. The instance is moved to new hardware while keeping its ID, Private IP, and EBS volumes intact.

---

### Industrial Example: Scalable Microservices Fleet

A modern fintech startup running their API on EC2:
*   **Architecture:** They use **Horizontal Scaling** with an ASG spanning 3 Availability Zones (HA).
*   **Cost Optimization:** They use a mix of **On-Demand** instances (for the Minimum Capacity) and **Spot Instances** (for the scaling capacity), saving up to 70% in costs.
*   **Scaling Policy:** They use **Target Tracking** based on "Request Count Per Target" to ensure the API never slows down regardless of how many users are logging in.
*   **Placement Strategy:** They use **Spread Placement Groups** for their core Kafka brokers to ensure a single hardware failure doesn't cause a data cluster outage.
