# AWS NACL: The Subnet-Level Firewall

A **Network Access Control List (NACL)** is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more **subnets**.

While Security Groups act at the **Instance level**, NACLs act at the **Subnet level**.

---

### 1. Key Characteristics (The "Bouncer" Analogy)

Think of a NACL as a **Bouncer** standing at the gate of a neighborhood (Subnet). He has a list of rules for who can enter and who can leave.

*   **Rule Numbering (Priority):** Rules are evaluated in order, starting with the lowest numbered rule. 
*   **The "First Match" Rule:** As soon as a packet matches a rule, that rule is applied (ALLOW or DENY), and the NACL **stops looking** at any other rules. 
    *   *Example:* If Rule #10 denies your IP, and Rule #20 allows it, you are **DENIED**. The NACL never even reads Rule #20 because the match at #10 drove the final decision.
*   **The Last Rule (Implicit Deny):** Every NACL has a final rule at the very bottom, usually labeled with an **asterisk (`*`)**. 
    *   **What it does:** It **DENIES** all traffic that didn't match any of your numbered rules. This is AWS's "Catch-All" to ensure your subnet is secure. You cannot delete or change this rule.
*   **Allow & Deny:** Unlike Security Groups (which only support "Allow"), NACLs support both **Allow** and **Deny** rules. This is how you block specific malicious IP addresses.
*   **Subnet Association:** A NACL only affects the subnets it is associated with. A subnet can only be associated with **one** NACL at a time.

---

### 2. Statelessness: The "Forgetful" Firewall

This is the most important concept for the SAA exam. NACLs are **Stateless**.

*   **What it means:** If you allow an inbound request (e.g., Port 80), the NACL **forgets** about it instantly. To let the response go back to the user, you must manually create an **Outbound Rule**.
*   **The Trap:** Even if you allow Port 80 Inbound, you must allow **Ephemeral Ports (1024-65535)** Outbound for the web server's response to actually reach the client.

### 3. Understanding Ephemeral Ports

Since NACLs are stateless, they don't know that an outbound packet is a response to an inbound request. This is where **Ephemeral Ports** (Short-lived ports) become critical.

#### What are they?
When a client (like your browser) connects to a server (on Port 80), the client doesn't use Port 80 on its own side. Instead, it picks a random high-numbered port to receive the reply.

| Operating System | Ephemeral Port Range |
| :--- | :--- |
| **Linux (AWS AMI)** | 32768–65535 |
| **Windows** | 49152–65535 |
| **Elastic Load Balancer** | 1024–65535 |

> [!TIP]
> **Industrial Best Practice:** For the SAA exam, always assume the range is **1024–65535** to cover all bases.

#### The Scenario: "The Missing Webpage"
1.  **Request:** A user on the internet sends a request to your Web Server on **Port 80**.
2.  **Inbound NACL:** You have a rule allowing Port 80. The request gets in.
3.  **Response:** Your server prepares the HTML and tries to send it back. But the user's browser is waiting on a random port (e.g., **Port 49152**).
4.  **Outbound NACL:** If you don't have an Outbound rule allowing traffic to ports **1024–65535**, the NACL will BLOCK the response. 
5.  **Result:** The user sees a "Connection Timed Out" error, even though your server is running perfectly!

---

### 4. NACL vs. Security Group (The "Must-Know" Table)

| Feature | Security Group (Instance Level) | NACL (Subnet Level) |
| :--- | :--- | :--- |
| **State** | **Stateful** (Inbound allows Outbound) | **Stateless** (Must define both) |
| **Rules** | Support **Allow** only | Support **Allow** and **Deny** |
| **Evaluation** | All rules evaluated before decision | Rules evaluated in order (Low to High) |
| **Target** | Attaches to an **ENI/Instance** | Attaches to a **Subnet** |

---

### 4. Default vs. Custom NACLs

#### Visualizing the Default NACL
The Default NACL is designed to keep things working out-of-the-box. It looks like this:

**Inbound Rules:**
| Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **100** | ALL Traffic | ALL | ALL | 0.0.0.0/0 | **ALLOW** |
| **(*)** | ALL Traffic | ALL | ALL | 0.0.0.0/0 | **DENY** |

**Outbound Rules:**
| Rule # | Type | Protocol | Port Range | Destination | Allow/Deny |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **100** | ALL Traffic | ALL | ALL | 0.0.0.0/0 | **ALLOW** |
| **(*)** | ALL Traffic | ALL | ALL | 0.0.0.0/0 | **DENY** |

*   **Why everything works:** Because Rule #100 is evaluated first and allows everything, the asterisk rule (`*`) is never reached.
*   **Default VPC NACL:** By default, it allows **ALL** inbound and outbound traffic. 
*   **Custom NACL:** When you create a new custom NACL, it starts with **ONLY the asterisk rule**. It **DENIES everything** until you manually add Rule #100, #200, etc.

### 6. Multi-Tier Architecture Flow (Web vs. DB)

Let's look at the "Grand Tour" of a request moving through a VPC with **Public Subnet (Web)** and **Private Subnet (Database)**. Because NACLs are stateless, the coordination of ports is like a complex dance.

#### Subnet 1: Public Subnet (Web)
| Type | Rule | Reason |
| :--- | :--- | :--- |
| **Inbound** | Allow Port 80/443 from `0.0.0.0/0` | Users visiting the website. |
| **Inbound** | Allow Ports 1024-65535 from `Private Subnet` | Receiving the DB's response back to the Web server. |
| **Outbound** | Allow Ports 1024-65535 to `0.0.0.0/0` | Sending the HTML response back to the user's browser. |
| **Outbound** | Allow Port 3306 (MySQL) to `Private Subnet` | Web server asking the database for data. |

#### Subnet 2: Private Subnet (Database)
| Type | Rule | Reason |
| :--- | :--- | :--- |
| **Inbound** | Allow Port 3306 from `Public Subnet` | Receiving the request from the Web server. |
| **Outbound** | Allow Ports 1024-65535 to `Public Subnet` | Replying to the Web server. |

---

### 7. Solution Architect Exam Questions (SAA-C03 Level)

**Q1: A company's web server is receiving an attack from a specific IP address (1.2.3.4). Which AWS feature should be used to explicitly block this IP address?**
*   **A)** Security Group
*   **B)** Network ACL
*   **C)** Route Table
*   **Answer:** **B**. Security Groups only support "Allow" rules. To explicitly "Deny" an IP, you must use a NACL.

**Q2: You have allowed Inbound traffic on Port 443 in your custom NACL, but your users still cannot connect to your HTTPS website. What is the most likely cause?**
*   **A)** You forgot to allow Port 443 in the Security Group.
*   **B)** You forgot to add an Outbound rule for Ephemeral Ports (1024-65535) in the NACL.
*   **C)** NACLs do not support Port 443.
*   **Answer:** **B**. Because NACLs are stateless, you must allow the response traffic to leave via ephemeral ports.

**Q3: A NACL has Rule #100: "Allow Port 80 from 0.0.0.0/0" and Rule #50: "Deny Port 80 from 0.0.0.0/0". What happens to incoming web traffic?**
*   **A)** It is allowed because Rule 100 exists.
*   **B)** It is denied because Rule 50 is evaluated first (lowest number).
*   **C)** It is denied because Deny always takes precedence over Allow.
*   **Answer:** **B**. AWS processes rules in numerical order, starting from the lowest number. Since 50 is lower than 100, the traffic is denied immediately.
