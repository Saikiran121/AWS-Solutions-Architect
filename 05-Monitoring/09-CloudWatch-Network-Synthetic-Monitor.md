# Amazon CloudWatch Synthetics (Canaries)

Before CloudWatch Synthetics, AWS monitoring was entirely **Passive**. 
If your website's "Checkout" button broke, CloudWatch Metrics and Logs would only trigger an alarm *after* 100 real customers clicked it, got an error, and failed to give you their money. You were waiting on your customers to report your outages.

**Amazon CloudWatch Synthetics** introduces **Active** monitoring. Instead of waiting for real users to fail, AWS creates fake "synthetic" traffic that continuously tests your site 24/7/365, finding the bugs *before* your customers do.

---

### 1. What are Canaries?

CloudWatch Synthetics uses scripts called **Canaries**. 
A Canary is a lightweight Node.js or Python script that runs in an automated, headless web browser (like Puppeteer or Selenium). You schedule the Canary to run every minute (or 5 mins, or hour). It spins up, navigates to your website, performs a task, records the result, and spins down.

*The name "Canary" comes from the old coal mining technique: miners brought a canary bird into the mine. If toxic gas was present, the highly sensitive bird died first, acting as a warning to the miners before they were harmed.*

---

### 2. Common Canary Blueprints (Use Cases)

You do not need to be a Python/Node programmer to use Canaries. AWS provides pre-built "blueprints" you can launch with a few clicks.

**A. Heartbeat Monitoring (The Ping Test)**
*   The simplest Canary. It loads a single URL (e.g., `https://mycompany.com`) and checks if the server returns an HTTP `200 OK` response within a certain number of seconds. If the page times out or returns a `500 Internal Server Error`, the Canary fails and triggers a standard CloudWatch Alarm.

**B. API Canaries**
*   Instead of loading a whole webpage, this Canary sends HTTP `GET` or `POST` requests directly to your REST APIs (like Amazon API Gateway). It verifies that the JSON response payload is correctly formatted and that the database query latency is acceptable.

**C. GUI Workflow Builder (The E-Commerce Test)**
*   This is where Canaries shine. You can build a multi-step script that simulates a complex user journey.
*   *Scenario:* A Canary is scheduled to run every 5 minutes. First, it goes to `mystore.com`. Then, it clicks the "Shoes" category. Then, it clicks "Add to Cart". Finally, it clicks "Checkout". 
*   If *any* of those specific UI buttons fail to load or respond, the Canary immediately pages the engineering team.

**D. Broken Link Checker**
*   The Canary crawls your webpage, finds every single `<a href>` hyperlink, and clicks them all in the background to ensure none of them return a `404 Not Found` error.

---

### 3. Monitoring Internal VPC Endpoints (Highly Tested Concept)

Canaries are not just for public-facing internet websites. **You can launch Canaries inside your Private VPC Subnets.**

*   **Scenario:** Your company builds an internal microservice running on an EC2 instance in a private subnet. It has no public IP address and cannot be reached from the internet.
*   **The Solution:** You assign a VPC, Subnet, and Security Group to the CloudWatch Canary. AWS will invisibly spin up the Canary *inside* your private network, allowing it to ping your private, internal-only IP addresses and APIs to ensure your backend microservices are healthy.

---

### 4. CloudWatch Network Monitor (Distinction)

*(Note: Do not confuse Synthetics with "CloudWatch Network Monitor").*

*   **CloudWatch Synthetics:** Tests *Application* health (Websites, APIs, User Clicks).
*   **CloudWatch Network Monitor:** Tests *Network Infrastructure* health. It specifically measures network latency and packet loss between your AWS VPC and your physical on-premises data center (usually connected via AWS Direct Connect or Site-to-Site VPN). 

---

### 5. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: An e-commerce company relies on a complex, multi-step checkout process on their main web application. Recently, a minor code deployment broke the "Submit Payment" button, but standard CloudWatch CPU and Database metrics remained perfectly normal. The operations team only discovered the outage 3 hours later after hundreds of customer complaints on social media. What is the MOST operationally efficient way for a Solutions Architect to proactively monitor this specific user journey going forward?**
*   **A)** Install the unified CloudWatch Agent on all web servers and configure it to parse the Nginx `access.log` for 500 errors.
*   **B)** Create a CloudWatch Synthetics Canary using the GUI Workflow builder to systematically click through the checkout process every 5 minutes and alarm if any step fails.
*   **C)** Create a custom Lambda function triggered by Amazon EventBridge every 5 minutes to download a static HTML copy of the home page.
*   **D)** Enable AWS CloudTrail data event logging for the web servers and create a Metric Filter to track failed payment API calls.
*   **Answer:** **B**. CloudWatch Synthetics Canaries using GUI workflows are specifically designed to actively simulate user click-paths. This ensures complex UI interactions (like clicking a specific button) are tested constantly, alerting the team immediately if a specific workflow breaks, even if backend CPU/DB metrics look entirely normal (A). Lambda (C) would only grab the static home page, not test the multi-step checkout UI. CloudTrail (D) is for AWS API auditing, not application monitoring.

**Q2: A company has developed a suite of private microservices hosted on Amazon EC2 instances located exclusively in private subnets within a custom VPC. These microservices do not have public IP addresses and are only accessible by other internal applications. The operations team wants to continuously ping these private REST APIs every minute to verify they are returning the correct JSON responses. How can this be achieved using AWS native services?**
*   **A)** Use Amazon Route 53 Health Checks to ping the private IP addresses of the EC2 instances.
*   **B)** Create a CloudWatch Synthetics API Canary and configure it to run within the private VPC subnets, assigning it appropriate Security Group access.
*   **C)** Install the unified CloudWatch Agent on the EC2 instances and configure the `config.json` file to perform local API health checks.
*   **D)** Use AWS Global Accelerator to create a private endpoint directly to the CloudWatch monitoring service.
*   **Answer:** **B**. You can associate CloudWatch Synthetics Canaries with your custom VPCs and private subnets. This allows the Canary (which runs as a Lambda function under the hood) to securely access and ping private IP addresses and internal resources that are completely isolated from the public internet. Route 53 Health Checks (A) can only check *public-facing* endpoints over the internet (or endpoints connected via Direct Connect), they cannot reach natively into a private subnet without complex routing workarounds.

**Q3: A Solutions Architect needs to continuously monitor the network latency and packet loss between an AWS VPC hosted in `us-east-1` and the company's corporate on-premises data center connected via AWS Direct Connect. Which service is explicitly designed to solve this use case?**
*   **A)** CloudWatch Synthetics Heartbeat Canaries
*   **B)** AWS Transit Gateway Network Manager
*   **C)** CloudWatch Network Monitor
*   **D)** VPC Flow Logs
*   **Answer:** **C**. CloudWatch Network Monitor is explicitly designed to track network packet loss and latency between an AWS VPC subnet and a specific on-premises destination IP address over hybrid connections like Direct Connect or VPN. Synthetics (A) applies to HTTP/application layers. VPC Flow Logs (D) tracks metadata about packet IP sources/destinations, but does not actively measure real-time latency or active packet drop rates.
