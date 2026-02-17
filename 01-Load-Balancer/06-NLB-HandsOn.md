# Hands-On: Setting Up a Network Load Balancer (NLB)

This guide takes you through the deployment of a high-performance Network Load Balancer (NLB) to handle backend web traffic using the TCP protocol.

---

### Step 1: Launch EC2 Instances
We will start by launching two backend servers that will serve our web content.

1.  **Launch Template/Instance:** Launch **2 EC2 instances** using the **Amazon Linux 2023 AMI**.
2.  **Security Group:** Create or select a Security Group for the instances that allows:
    *   **HTTP (Port 80)** from anywhere (0.0.0.0/0).
    *   **HTTPS (Port 443)** from anywhere (0.0.0.0/0).
3.  **User Data:** Under "Advanced Details," paste the following script:

```bash
#!/bin/bash
# Update the system
yum update -y
# Install Apache
yum install -y httpd
# Start and enable the service
systemctl start httpd
systemctl enable httpd
# Create a simple homepage with the instance hostname
echo "<h1> Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

---

### Step 2: Create the Network Load Balancer (NLB)

Now, we will configure the NLB, incorporating the new AWS feature that allows NLBs to have their own Security Groups.

#### 1. Security Group for NLB
*   Go to **Security Groups** -> **Create security group**.
*   **Name:** `demo-sg-nlb`.
*   **Inbound Rules:** Allow **HTTP (Port 80)** from **Anywhere (0.0.0.0/0)**.

#### 2. Target Group Configuration
*   Go to **Target Groups** -> **Create target group**.
*   **Target Type:** `Instances`.
*   **Name:** `demo-tg-nlb`.
*   **Protocol:** **TCP** on Port **80**.
*   **Health Checks:**
    *   Expand **Advanced health check settings**.
    *   Set **Timeout** to `5 seconds`.
    *   Set **Interval** to `5 seconds`. (This ensures very fast detection of failures).
*   **Register Targets:** Select both EC2 instances and click **Include as pending below**.
*   **Create target group**.

#### 3. Creating the Load Balancer
*   Go to **Load Balancers** -> **Create load balancer**.
*   **Select:** **Network Load Balancer**.
*   **Name:** `DemoNLB`.
*   **Scheme:** **Internet-facing**.
*   **Network Mapping:** Select your VPC and **all 3 Availability Zones** (AZs).
*   **Security Groups:** Attach the `demo-sg-nlb` created earlier.
*   **Listeners and Routing:**
    *   **Protocol:** TCP
    *   **Port:** 80
    *   **Default Action:** Forward to `demo-tg-nlb`.
*   **Review and Create:** Click **Create load balancer**.

---

### Step 3: Verification
1.  Wait for the NLB to become **Active**.
2.  Copy the **DNS Name** from the NLB dashboard.
3.  **The Result:** Paste the DNS name into your browser. You will see the "Hello World" page.
4.  **Observation:** Unlike ALB, NLB operates at Layer 4. Notice how the connection feels snappier and how the NLB preserves the connection flow to a single instance more strictly than an ALB might in simple tests.

---

### Step 4: Securing the Backend (NLB Security Group Chaining)
Force all traffic through your Network Load Balancer to ensure a secure, hardened architecture.

1.  **Identify the NLB Security Group:** Go to **Load Balancers** -> Select `DemoNLB` -> Find the **Security Group ID** (e.g., `sg-0xyz...`).
2.  **Edit the EC2 Security Group:**
    *   Go to **Instances** -> Select your EC2 instance -> **Security** -> Click the **Security Group**.
    *   Click **Edit inbound rules**.
3.  **Implement Least Privilege:**
    *   **Delete** the existing `HTTP` rule that allows `0.0.0.0/0`.
    *   **Add a new rule:** 
        *   **Type:** `HTTP`.
        *   **Port:** `80`.
        *   **Source:** Select the **Security Group ID of the Network Load Balancer** (`demo-sg-nlb`).
4.  **Save Rules:** Click **Save rules**.

---

### Verification: Can I still access my instance via its Public IP?

**No.** Just like with the ALB, any direct connection attempt to the EC2 Public IP will now time out.

*   **Why?** The instance's Security Group is now locked. It only recognizes the NLB's Security Group as a valid traffic source. 
*   **The Benefit:** Your instances are now "shielded" from the public internet. Even if an attacker knows your server's IP, they cannot reach it directly. Every request must pass through the NLB (and its security layers) first.
