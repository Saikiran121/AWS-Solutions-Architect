# AWS Subnets: Designing the Internal Network

A **Subnet** (Sub-network) is a range of IP addresses within a VPC. Subnets allow you to group your resources based on security and operational needs.

---

### 1. Types of Subnets

*   **Public Subnet:** A subnet that has a route to an **Internet Gateway (IGW)**. Resources here (like Load Balancers) can be accessed from the public internet.
*   **Private Subnet:** A subnet that does **not** have a route to an IGW. To access the internet (e.g., for updates), it must use a **NAT Gateway**. Resources here (like Databases) are hidden from the public internet.

---

### 2. The 5 Reserved IPs in AWS
AWS reserves **5 IP addresses** in every subnet for internal networking management. These IPs are **not usable** for your instances.

| IP Address | Role | Description |
| :--- | :--- | :--- |
| **.0** | **Network Address** | Identifies the network itself (e.g., `10.0.0.0`). |
| **.1** | **VPC Router** | The gateway IP for the subnet to communicate with the rest of the VPC. |
| **.2** | **DNS Server** | The Amazon-provided DNS (Route 53 Resolver) for the VPC. |
| **.3** | **Reserved** | Reserved by AWS for future internal use/management. |
| **.255** | **Broadcast** | Traditionally used for broadcasting (AWS VPCs do not support broadcast). |

> [!IMPORTANT]
> **Why are they reserved?** AWS requires these IPs to manage the "Virtual" nature of the cloud network. Without these, routing, DNS resolution, and AWS service communication within the VPC would break.

---

### 3. Sizing Scenario: The 29 EC2 Instance Challenge

**The Requirement:** You need to launch **29 EC2 instances** into a single subnet. What CIDR range should you use?

#### Option A: /27 (32 IPs)
*   **Total IPs:** 2^(32-27) = 32.
*   **Usable IPs:** 32 (Total) - 5 (Reserved) = **27**.
*   **Result:** **FAIL.** You cannot host 29 instances because you only have 27 usable IPs.

#### Option B: /26 (64 IPs)
*   **Total IPs:** 2^(32-26) = 64.
*   **Usable IPs:** 64 (Total) - 5 (Reserved) = **59**.
*   **Result:** **SUCCESS.** You can host your 29 instances and still have room for 30 more in the future.

**Conclusion:** Always calculate your usable IPs as `(2^(32-N)) - 5` to ensure your workload fits.

---

### 4. Industrial Scenarios

#### Scenario A: High Availability (Multi-AZ)
A company like **Netflix** doesn't just put everything in one subnet.
*   **Implementation:** They create identical subnets in **US-East-1a**, **US-East-1b**, and **US-East-1c**.
*   **The Benefit:** If one AWS data center (Availability Zone) goes down, the subnets in the other zones stay online.

#### Scenario B: The Secure "3-Tier" Design
E-commerce platforms like **Shopify** use subnets to create isolation:
*   **Public Subnet (/24):** Only for the Load Balancers (ALB).
*   **Application Subnet (/24):** Private. For the "Logic" servers. No direct internet access.
*   **Database Subnet (/28):** Private. Only for the Database. Very small CIDR to reduce the attack surface.
