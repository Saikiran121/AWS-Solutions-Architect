# AWS VPC Peering: Connecting Isolated Networks

**VPC Peering** is a networking connection between two VPCs that enables you to route traffic between them using **Private IPv4** or IPv6 addresses. Instances in either VPC can communicate as if they are within the same network.

---

### 1. Key Capabilities

*   **Cross-Account Peering:** Connect your VPC with a VPC in a different AWS account.
*   **Inter-Region Peering:** Connect VPCs across different AWS Regions (e.g., `us-east-1` to `ap-south-1`). Traffic is encrypted and stays on the AWS global backbone (never touches the public internet).

---

### 2. The Golden Rules (The "Exam Watch" Section)

AWS strictly enforces several rules that frequently appear in SAA-C03 exam questions:

#### A. No Transitive Peering (The Most Famous Rule)
If VPC A is peered with VPC B, and VPC B is peered with VPC C, **VPC A cannot talk to VPC C**. 
*   **The Logic:** `A -> B` and `B -> C` does **NOT** equal `A -> C`.
*   **The Fix:** You must create a direct peering connection between VPC A and VPC C, or use a **Transit Gateway**.

#### B. No Overlapping CIDRs
You cannot create a peering connection if the VPCs have overlapping IP address ranges (CIDR blocks). 
*   *Example:* If VPC A is `10.0.0.0/16` and VPC B is `10.0.0.0/16`, peering will **FAIL**.

#### C. Route Tables are NOT Automatic
Creating the peering connection is only 50% of the work. You **MUST** manually update the Route Tables in **BOTH** VPCs to point traffic to the Peering Connection ID (`pcx-xxxx`).

### 3. Referencing Security Groups Across Peering

One of the most powerful features of VPC Peering is the ability to reference a Security Group (SG) from a peered VPC as a **Source** or **Destination** in your own SG rules. This allows for fine-grained security without managing IP lists.

#### A. Same Account & Same Region
If both VPCs are in your account and region:
- **Syntax:** Simply select/paste the **Security Group ID** (e.g., `sg-1a2b3c4d`) of the instance in the other VPC.

#### B. Cross-Account Peering (Same Region)
If the peered VPC belongs to a different AWS account:
- **Syntax:** You must use the format **`AccountID/SecurityGroupID`**.
- *Example:* `123456789012/sg-1a2b3c4d`
- **Why:** This tells AWS exactly which account owns that "badge" (SG badge).

#### C. The Inter-Region Limitation (CRITICAL)
> [!WARNING]
> **Inter-Region Limitation:** You **CANNOT** reference Security Groups across Inter-Region VPC Peering connections.
> 
> *   **The Fix:** You must use the **CIDR block** (IP range) of the instances in the remote region instead.

---

### 4. Implementation Workflow

1.  **Request:** Owner of VPC A sends a peering request to VPC B.
2.  **Accept:** Owner of VPC B accepts the request (the status changes to `Active`).
3.  **Update Routes:** 
    *   In VPC A's Route Table: Add `VPC B CIDR` -> `pcx-xxxx`.
    *   In VPC B's Route Table: Add `VPC A CIDR` -> `pcx-xxxx`.
4.  **Update Security Groups:** Allow the CIDR or the Security Group ID of the peered VPC.

---

### 4. Industrial Scenarios

*   **Shared Services VPC:** A central VPC containing monitoring tools (CloudWatch, LDAP, AD) that every other VPC in the company needs to access.
*   **Corporate Mergers:** Connecting the networks of two different companies (accounts) without using VPNs or the Public Internet.

---

### 5. Solution Architect Exam Questions (SAA-C03 Level)

**Q1: You have three VPCs: A, B, and C. VPC A is peered with VPC B, and VPC B is peered with VPC C. VPC A needs to access a database in VPC C. What is the most efficient solution?**
*   **A)** Enable "Transitive Route" in VPC B's configuration.
*   **B)** Create a direct VPC Peering connection between VPC A and VPC C.
*   **C)** Route traffic through an Internet Gateway.
*   **Answer:** **B**. Peering is non-transitive.

**Q2: You have successfully created and accepted a VPC peering connection between VPC A (10.1.0.0/16) and VPC B (10.2.0.0/16). However, instances still cannot ping each other. Which step did you most likely forget?**
*   **A)** Enabling DNS resolution on the peering connection.
*   **B)** Updating the Route Tables in both VPCs.
*   **C)** Peering only works for instances in the same Availability Zone.
*   **Answer:** **B**. Manual route table updates are mandatory for peering traffic to flow.

**Q3: Can you peer two VPCs that both have the CIDR block `172.31.0.0/16`?**
*   **A)** Yes, if they are in different regions.
*   **B)** Yes, if they are in different accounts.
*   **C)** No, CIDR blocks must be unique for peering.
*   **Answer:** **C**. Overlapping CIDRs are a hard limitation.
