# AWS Bastion Hosts: Secure Administrative Access

A **Bastion Host** (also known as a **Jump Box**) is a specialized server that sits in a **Public Subnet** and acts as a gateway to your servers sitting in **Private Subnets**. It is designed to withstand attacks and provides a secure single point of entry for administrators.

---

### 1. Why We Need Them
In a production-ready VPC, your application servers and databases are kept in **Private Subnets** (no internet access) to protect them from hackers. However, administrators still need a way to log in to these servers for maintenance or troubleshooting.

*   **Security:** Instead of opening SSH (Port 22) or RDP (Port 3389) on every server to the entire internet, you only open it on the Bastion Host.
*   **Auditability:** You can log and monitor all administrative activity at a single point.

---

### 2. The Architecture (Security Group Chaining)

The Bastion Host pattern relies on a security technique called **Security Group Chaining**:

1.  **Bastion Security Group:**
    *   **Inbound:** Allow SSH (Port 22) only from **your specific IP address**.
2.  **Private Instance Security Group:**
    *   **Inbound:** Allow SSH (Port 22) **only from the Bastion Host's Security Group**.

**The Benefit:** It is physically impossible for anyone on the internet to talk to the private instances directly. They *must* go through the Bastion first.

---

### 3. Best Practices for Hardening
*   **Minimal OS:** Use a lightweight Linux distribution with only the bare essentials installed.
*   **Static Private IP:** Ensure the Bastion has a predictable IP for internal routing.
*   **No Data:** Never store sensitive data, keys, or source code on the Bastion.
*   **MFA:** Always use multi-factor authentication for logging into the Bastion.

---

### 4. Bastion Host vs. NAT Gateway
This is a very common point of confusion:
*   **Bastion Host:** Used by **Adults (Administrators)** to get **INTO** the VPC for management.
*   **NAT Gateway:** Used by **Servers** to get **OUT** of the VPC for software updates.

---

### 5. Modern Alternative: AWS Systems Manager (SSM)
While Bastion Hosts are classic, AWS now recommends **SSM Session Manager**.
*   **Why?** It allows you to log into instances (Public or Private) via the AWS Console or CLI **without** needing a Bastion Host, an IGW, or even opening Port 22. It is more secure and easier to manage.

---

### 6. Solution Architect Exam Questions (SAA-C03 Level)

**Q1: You need to allow an administrator to SSH into an EC2 instance located in a private subnet. Which architectural component should you place in a public subnet to facilitate this safely?**
*   **A)** NAT Gateway
*   **B)** Bastion Host
*   **C)** VPC Endpoint
*   **Answer:** **B**. A Bastion Host is designed for administrative jump-access. NAT Gateways are for outbound traffic only.

**Q2: Which Security Group configuration represents the best practice for a 3-tier application using a Bastion Host?**
*   **A)** Private instances allow Port 22 from `0.0.0.0/0`.
*   **B)** Bastion Host allows Port 22 from `0.0.0.0/0`.
*   **C)** Private instances allow Port 22 only from the security group of the Bastion Host.
*   **Answer:** **C**. This is called Security Group Chaining and ensures only the Bastion can reach the private servers.

**Q3: A company wants to eliminate the overhead of managing Bastion Hosts and SSH keys while maintaining secure access to private instances. What should a Solution Architect recommend?**
*   **A)** Use a NAT Instance.
*   **B)** Use AWS Systems Manager (SSM) Session Manager.
*   **C)** Move the instances to the public subnet temporarily.
*   **Answer:** **B**. SSM Session Manager provides secure access without the need for Bastions or managing SSH keys in the traditional way.
