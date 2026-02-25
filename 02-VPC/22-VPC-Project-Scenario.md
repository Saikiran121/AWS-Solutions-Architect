# Project: Secure 3-Tier Enterprise Architecture

In this project, we will apply everything we have learned about VPC components to design a production-ready, highly available, and secure network architecture.

---

### 1. Business Requirement
A financial services company needs to host a web application with the following requirements:
*   **High Availability:** Must survive the failure of a single Availability Zone (AZ).
*   **Security:** Database and Application servers must never be exposed to the public internet.
*   **Management:** Admins need a secure way to SSH into private instances.
*   **Efficiency:** High-speed, private access to S3 for database backups.
*   **DevOps Integration:** The Production VPC must connect to a Shared Services VPC (where the Jenkins server is located).

---

### 2. Architecture Overview (The 3-Tier Design)

We will use **Multi-AZ Architecture** (spanning across AZ-1 and AZ-2).

#### Tier 1: Public Tier (Web/External)
*   **Components:** Application Load Balancer (ALB) and a **Bastion Host**.
*   **Subnets:** Public Subnets (AZ-1 & AZ-2).
*   **Routing:** Direct route to the **Internet Gateway (IGW)**.

#### Tier 2: Application Tier (Private)
*   **Components:** EC2 instances running the application code.
*   **Subnets:** Private Subnets (AZ-1 & AZ-2).
*   **Routing:** Outbound traffic goes through a **NAT Gateway** (for patching); S3 traffic goes through a **Gateway Endpoint**.

#### Tier 3: Database Tier (Private/Restricted)
*   **Components:** RDS (MySQL/Aurora) Multi-AZ.
*   **Subnets:** Private Data Subnets (AZ-1 & AZ-2).
*   **Routing:** **NO Internet access.** Purely internal connectivity.

---

### 3. Networking Logic (The "Glue")

#### A. Secure Inbound Flow
1.  **User** connects to the **ALB** (HTTPS on Port 443).
2.  **ALB** forwards traffic to the **App Instances** (Port 80/8080).
3.  **Security Group Check:** App instances only allow traffic *from* the ALB's Security Group.

#### B. Secure Management Flow (Bastion)
1.  **Admin** SSHs into the **Bastion Host** (Port 22). 
2.  **Admin** then SSHs from the Bastion into the **Private App Instances**.
3.  **Security Group Check:** Private instances only allow SSH (Port 22) *from* the Bastion's Security Group.

#### C. Cross-VPC Connectivity (Peering)
1.  A **VPC Peering Connection** is established between **Production-VPC** and **Shared-VPC**.
2.  **Jenkins** (in Shared-VPC) can now deploy code directly to the **App Tier** in Production using private IP addresses.

---

### 4. Defense-in-Depth Checklist

| Layer | Component | Security Strategy |
| :--- | :--- | :--- |
| **Edge** | IGW | Only attached to Public subnets. |
| **Subnet** | NACLs | Deny high-risk ports (e.g., 3389 RDP) at the entry point. |
| **Instance** | Security Groups | Statefully restrict traffic to specific sources (SG Chaining). |
| **API** | S3 Endpoint | Ensure S3 traffic stays on the AWS backbone, bypassing the NAT. |

---

### 5. Why this is "Industry Standard"?
1.  **Isolation:** Even if the Web ALB is compromised, the Database is two layers deep.
2.  **No Single Point of Failure:** Spanning across two AZs ensures 99.9% availability.
3.  **Cost Optimized:** Using Gateway Endpoints for S3 saves significant NAT Gateway data processing fees.
4.  **Least Privilege:** Every component (ALB, App, DB) has its own Security Group with the absolute minimum ports open.

---

> [!TIP]
> **Implementation Tip:** When building this, always create the **VPC and Subnets** first, followed by the **IGW/NAT**, then the **Route Tables**, and finally the **Security Groups** before launching any EC2 instances.
