# Hands-On: Setting Up an Application Load Balancer (ALB)

This guide provides step-by-step instructions for creating a classic ALB architecture with two backend EC2 instances serving a "Hello World" page.

---

### Step 1: Launch EC2 Instances
To provide a backend for our load balancer, we need to create two identical web servers.

1.  **Launch Template/Instance:** Launch **2 EC2 instances** using the **Amazon Linux 2023 AMI**.
2.  **Security Group:** Create or select a Security Group that allows inbound traffic on:
    *   **HTTP (Port 80)** from anywhere (0.0.0.0/0).
    *   **HTTPS (Port 443)** from anywhere (0.0.0.0/0).
3.  **User Data:** Under "Advanced Details," paste the following script to automatically install and start the Apache web server:

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

### Step 2: Create a Target Group
The Target Group tells the load balancer where to send the traffic.

1.  **Navigate:** Go to the EC2 Dashboard -> **Target Groups** -> **Create target group**.
2.  **Type:** Select **Instances**.
3.  **Name:** Set the name to `demo-tg-alb`.
4.  **Protocol & Port:** Select **HTTP** on Port **80**.
5.  **Health Checks:** Use the default settings (Path: `/`).
6.  **Register Targets:**
    *   Select the two EC2 instances you launched in Step 1.
    *   Click **Include as pending below**.
    *   Click **Create target group**.

---

### Step 3: Create the Application Load Balancer (ALB)
Now, we create the entry point for our users.

1.  **Navigate:** Go to EC2 Dashboard -> **Load Balancers** -> **Create load balancer**.
2.  **Type:** Select **Application Load Balancer**.
3.  **Basic Configuration:**
    *   **Name:** `DemoALB`.
    *   **Scheme:** **Internet-facing**.
    *   **IP address type:** IPv4.
4.  **Network Mapping:**
    *   Select your VPC.
    *   Select **all 3 Availability Zones** (AZs) to ensure high availability.
5.  **Security Groups:** Select the Security Group we created earlier (allowing HTTP/HTTPS).
6.  **Listeners and Routing:**
    *   **Protocol:** HTTP
    *   **Port:** 80
    *   **Default Action:** Forward to target group `demo-tg-alb`.
7.  **Review and Create:** Click **Create load balancer**.

---

### Step 4: Verification
Once the Load Balancer state changes from `Provisioning` to `Active`:

1.  Copy the **DNS Name** of the ALB (e.g., `DemoALB-12345678.us-east-1.elb.amazonaws.com`).
2.  Paste it into your browser.
3.  **The Result:** You should see the "Hello World" message along with the hostname of one of your instances.
4.  **Test Balancing:** Refresh the page several times. You should see the hostname change as the ALB balances the load between the two instances.

---

### Step 5: Testing High Availability (Failure & Recovery)
Now, let's observe how the ALB handles a real-world server failure.

1.  **Stop an Instance:** Go to the EC2 Dashboard -> **Instances** -> Select one of your two instances -> **Instance state** -> **Stop instance**.
2.  **Monitor Health Status:**
    *   Go to **Target Groups** -> Select `demo-tg-alb`.
    *   Click the **Targets** tab.
    *   After a minute, you should see the status of the stopped instance change from `Healthy` to `Unhealthy` (or eventually `unused`).
3.  **Verify Traffic Flow:**
    *   Go back to your browser tab with the ALB DNS Name.
    *   Refresh the page multiple times.
    *   **The Result:** You will notice that you only see the hostname of the **running** instance. The ALB has automatically detected the failure and stopped sending traffic to the stopped server.
4.  **Recover the Instance:**
    *   Go back to the EC2 Dashboard and **Start** the stopped instance.
5.  **Observe Recovery:**
Watch the **Targets** tab in the Target Group. The instance will move to `initial` and then back to `Healthy` once its web server is up and health checks pass.
    *   Refresh your browser again.
    *   **The Result:** The ALB automatically resumes balancing traffic across **both** instances. This demonstrates the "Self-Healing" capability of the Load Balancer.

---

### Step 6: Securing the Backend (Security Group Chaining)
Currently, your EC2 instances are accessible via the Load Balancer AND their Public IPs. In a production environment, we want to force all traffic through the Load Balancer for security and logging.

1.  **Identify the Load Balancer SG:** Go to **Load Balancers** -> Select `DemoALB` -> Find the **Security Group ID** (e.g., `sg-0abc123...`).
2.  **Edit the EC2 Security Group:**
    *   Go to **Instances** -> Select your EC2 instance -> **Security** -> Click the **Security Group**.
    *   Click **Edit inbound rules**.
3.  **Implement Least Privilege:**
    *   **Delete** the existing `HTTP` rule that allows `0.0.0.0/0`.
    *   **Add a new rule:** 
        *   **Type:** `HTTP`.
        *   **Port:** `80`.
        *   **Source:** Select the **Security Group ID of the Load Balancer** (type `sg-` and select it from the dropdown).
4.  **Save Rules:** Click **Save rules**.

---

### Critical Question: Can I still access my EC2 instance with its Public IP?

**No.** After performing Step 6, if you try to paste the **Public IP** of your EC2 instance directly into your browser, the request will **time out**.

*   **Why?** The Security Group on the EC2 instance now specifically says: "I only accept traffic if it comes from the Load Balancer's Security Group." Since your laptop/browser is not the Load Balancer, the traffic is dropped.
*   **Is this good?** Yes! This is the industry-standard way to secure web servers. It ensures that attackers cannot bypass your Load Balancer's security features (like WAF or SSL termination) by attacking your instances directly via their IP addresses.

---

### Step 7: Mastering Advanced Routing with ALB Rules
In this step, we will move beyond simple load balancing and explore how to use **Listener Rules** to route traffic based on the content of the request.

#### What is an ALB Rule?
A rule consists of three parts:
1.  **Priority:** The order in which rules are evaluated (e.g., 1, 2, 3).
2.  **Conditions:** The "If" statement (e.g., *If* path is `/images/*`).
3.  **Actions:** The "Then" statement (e.g., *Then* forward to `images-tg`).

---

#### Hands-On Demo: Practical Conditions

1.  **Open Rules Editor:**
    *   Go to **Load Balancers** -> Select `DemoALB`.
    *   Click the **Listeners and rules** tab.
    *   Under the HTTP:80 listener, click **Manage rules**.
2.  **Add a Path-Based Rule:**
    *   Click **Add rule**.
    *   **Condition:** Select **Path pattern**. Enter `/app1/*`.
    *   **Action:** Select **Forward to target groups** -> `demo-tg-alb`.
    *   **Priority:** Give it a priority (e.g., 1).
    *   *Demo:* If you visit `ALB-DNS-Name/app1/index.html`, you will reach your app.
3.  **Add a Host-Based Rule:**
    *   **Condition:** Select **Host header**. Enter `demo.example.com`.
    *   **Action:** Select **Fixed response**. Enter Response code `200` and Text `Welcome to the Demo Domain!`.
    *   *Demo:* This rule only triggers if the browser sends a specific "Host" header.
4.  **Add a Query-String Rule:**
    *   **Condition:** Select **Query string**. Key: `version`, Value: `v2`.
    *   **Action:** Select **Fixed response**. Enter Response code `200` and Text `You are viewing Version 2`.
    *   *Demo:* Visit `ALB-DNS-Name/?version=v2` to see the custom response!

---

#### Technical Troubleshooting: The "Default Rule"
Every ALB has a **Default Rule** at the bottom (Priority: Last).
*   **Behavior:** If none of your custom conditions (Path, Host, Query, etc.) match the request, the ALB falls back to this rule.
*   **Best Practice:** Usually, the default rule forwards to your main application target group or returns a 404/Fixed Response.

**Verification:** In the Rules list, you can see your Priority 1, 2, and 3 rules stacked above the Default rule. The ALB evaluates them from top to bottom and stops at the first match.
