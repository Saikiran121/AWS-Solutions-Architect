# AWS VPC Flow Logs: The Network Flight Recorder

**VPC Flow Logs** is a feature that enables you to capture information about the IP traffic going to and from network interfaces in your VPC. Think of it as a "Flight Recorder" for your network.

---

### 1. Capture Levels & Types

You can enable flow logs at three different levels:
*   **VPC Level:** Captures traffic for all subnets and interfaces in the VPC.
*   **Subnet Level:** Captures traffic for all interfaces in a specific subnet.
*   **ENI (Elastic Network Interface) Level:** Captures traffic for a specific network interface (e.g., a specific instance).

**What to capture?**
*   **ALL:** Capture every packet attempt.
*   **ACCEPT:** Capture only traffic permitted by Security Groups and NACLs.
*   **REJECT:** Capture only traffic blocked by Security Groups or NACLs. (Best for troubleshooting "Access Denied" issues).

---

### 2. Log Destinations & Analysis Tools

Where you send your logs depends on how you plan to use them:

| Destination | Best Use Case | Analysis Tool |
| :--- | :--- | :--- |
| **Amazon S3** | Long-term storage, archival, and complex querying. | **Amazon Athena** (SQL queries) |
| **CloudWatch Logs** | Real-time monitoring, creating metric filters, and alarms. | **CloudWatch Logs Insights** |
| **Kinesis Firehose** | Streaming logs to 3rd party tools (Splunk, Datadog). | External Dashboards |

---

### 3. Industrial Scenarios

#### Scenario A: The "Ghost" Traffic
A security auditor notices unusual traffic patterns on a weekend. By enabling Flow Logs at the **VPC level** and sending them to **S3**, they can use **Athena** to find the Source IP and Port of all **REJECTED** traffic to identify a potential port-scanning attack.

#### Scenario B: Debugging Connectivity
Your developer says the application can't reach the Database. 
*   If Flow Logs show **REJECT** entries, it's a **Security Group** or **NACL** issue.
*   If Flow Logs show **nothing**, the traffic isn't even reaching the interface (likely a **Route Table** or **IGW** issue).

---

### 4. Important Limitations (SAA Exam Tips)
*   **Service Traffic:** Traffic to AWS DNS (AmazonProvidedDNS) is not captured.
*   **Instance Metadata:** Traffic to `169.254.169.254` is not captured.
*   **DHCP/Windows Licensing:** Traffic for these infrastructure services is excluded.
*   **No Real-Time Packet Capture:** It captures **metadata** (IPs, Ports, Bytes, Protocol), not the actual contents of the packets.

---

### 5. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A company is experiencing intermittent connectivity issues between two subnets. A developer suspects a Security Group is blocking the traffic. How can a Solution Architect investigate this MOST efficiently?**
*   **A)** Use AWS Trusted Advisor.
*   **B)** Enable VPC Flow Logs at the subnet level and filter for REJECT traffic.
*   **C)** Enable VPC Flow Logs at the VPC level and filter for ACCEPT traffic.
*   **Answer:** **B**. Filtering for REJECT traffic directly shows what Security Groups or NACLs are blocking.

**Q2: You have 100 GB of VPC Flow Logs stored in an S3 bucket. You need to run complex SQL-like queries to find a specific IP address that accessed your network last month. Which tool should you use?**
*   **A)** CloudWatch Logs Insights.
*   **B)** Amazon Redshift.
*   **C)** Amazon Athena.
*   **Answer:** **C**. Athena is the standard, serverless way to query large volumes of S3 logs using SQL.

**Q3: Which of the following is NOT captured by VPC Flow Logs?**
*   **A)** Traffic sent to the Amazon DNS service.
*   **B)** Inbound traffic from an Internet Gateway.
*   **C)** Outbound traffic to an S3 Gateway Endpoint.
*   **Answer:** **A**. AWS DNS and Instance Metadata traffic are specifically excluded from show logs.

---

> [!TIP]
> **Exam Strategy:** If the question mentions **"Querying S3 logs"**, the answer is almost always **Athena**. If it mentions **"Real-time alarms"**, the answer is **CloudWatch**.
