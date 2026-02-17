# Application Load Balancer (ALB): Deep Dive

The Application Load Balancer (ALB) is a high-performance, Layer 7 load balancer that operates at the **Application Layer** of the OSI model. It is designed to handle HTTP and HTTPS traffic with advanced routing capabilities, making it the ideal choice for modern web applications and microservices architectures.

---

### How ALB Works: The Architecture
ALB consists of three core components that work together to route traffic:

1.  **Listeners:** A process that checks for connection requests using a protocol and port (e.g., HTTPS on port 443).
2.  **Rules:** Logic that defines how the load balancer routes requests to targets. Rules consist of a priority, one or more actions, and one or more conditions.
3.  **Target Groups:** A logical grouping of targets (EC2 instances, microservices, Lambda functions) where the request is finally sent.

---

### Advanced Routing Features
ALB is famous for its "Intelligent Routing," which allows you to route traffic based on the content of the request.

*   **Path-Based Routing:** Route traffic based on the URL path.
    *   `/api/*` -> Sent to API Target Group.
    *   `/static/*` -> Sent to Storage/CDN Target Group.
*   **Host-Based Routing:** Route traffic based on the host name in the HTTP header.
    *   `app.example.com` -> Sent to Application Target Group.
    *   `blog.example.com` -> Sent to Blog Target Group.
*   **Query-String/Parameter Routing:** Route based on query parameters (e.g., `?version=v2`).
*   **HTTP Header-Based Routing:** Route based on specific headers like `User-Agent` to serve different content to mobile vs. desktop users.

---

### Key Features of ALB

1.  **SSL/TLS Termination:** Offload the compute-intensive task of encryption/decryption to the ALB. You can manage certificates via **AWS Certificate Manager (ACM)**.
2.  **WebSockets & HTTP/2 Support:** Native support for real-time, bi-directional communication and improved performance.
3.  **Sticky Sessions (Session Affinity):** Ensures a user's requests are consistently routed to the same target using cookies.
4.  **WAF Integration:** Direct integration with **AWS Web Application Firewall** to block malicious requests (SQLi, XSS) at the load balancer level.
5.  **Redirects & Fixed Responses:** You can configure the ALB to automatically redirect HTTP to HTTPS or return a custom 404/503 message directly without hitting the backend.

---

### Industrial Examples of ALB

#### 1. E-commerce Giant: Amazon.com
A massive platform like Amazon uses ALB to manage its microservices.
*   **Implementation:** They use **Path-Based Routing**.
    *   When you visit `/gp/cart`, the ALB routes you to a fleet of "Cart" microservices.
    *   When you go to `/checkout`, it routes you to a highly secure "Payments" fleet.
This ensures that a failure in the cart service doesn't stop people from searching for products.

#### 2. Multi-Tenant SaaS: Slack or Shopify
SaaS companies often provide "Custom Domains" for their clients.
*   **Implementation:** They use **Host-Based Routing**.
    *   `client-a.slack.com` and `client-b.slack.com` both hit the same set of Load Balancers, but the ALB uses the "Host" header to route them to the specific application cluster dedicated to that tenant.

#### 3. Modern Tech Startup: Blue/Green Deployments
Startups like **Monzo** or **Revolut** deployment frequently and need zero downtime.
*   **Implementation:** They use **Weighted Target Groups**.
    *   During a release, they might route 90% of traffic to the "Old Blue" cluster and 10% to the "New Green" cluster. As confidence grows, they shift the weight until 100% of traffic is on the new version.

#### 4. Video Platforms: Netflix (MetaData Side)
While the raw video stream might use NLB, the UI and API use ALB.
*   **Implementation:** Netflix uses ALB to serve the "Recommendation Engine" and "User Profile" APIs. They use **Header-Based Routing** to ensure that users on older Smart TVs receive a specific lightweight API response compared to users on a high-end web browser.

---

## 2. Target Groups: The Logical Routing Destinations

A **Target Group** is a logical grouping of resources that you want your Load Balancer to route traffic to. It acts as a bridge between the Load Balancer's **Listener Rules** and the actual backend servers or functions.

### Core Concept
When you define a rule on your ALB, you don't send traffic directly to a server; you send it to a **Target Group**. The Target Group then decides which specific "target" (instance, IP, etc.) is healthy and ready to receive the request.

---

### In-Depth: Target Types
AWS allows you to define targets in four different ways, providing immense architectural flexibility:

1.  **Instance ID:** You register specific EC2 instances by their ID. The LB uses the instance's primary private IP.
2.  **IP Address:** You route traffic to specific private IP addresses.
    *   **Pro Tip:** This allows you to route traffic to resources **outside** the current VPC, such as on-premises servers via VPN/Direct Connect or resources in peered VPCs.
3.  **Lambda Function:** Allows you to use an ALB as a front-end for serverless logic.
4.  **Application Load Balancer:** You can chain one ALB to another. This is common in complex microservices or when using PrivateLink.

---

### Critical Configuration Parameters

#### 1. Protocol and Port
Each Target Group has its own protocol (HTTP, HTTPS, GRPC) and port.
*   **Industrial Insight:** Your ALB might listen on port 443 (HTTPS) but talk to your backend Target Group on port 8080 (HTTP). This is called **SSL Offloading**.

#### 2. Health Checks
Target Groups inherit the health check configuration. We've previously discussed how these work, but remember: the LB only sends traffic to targets in the **"Healthy"** state.

#### 3. Deregistration Delay (Connection Draining)
When you remove a target or a target becomes unhealthy, AWS doesn't just "cut the cord."
*   **How it works:** The LB enters a "draining" state. It stops sending *new* requests but allows *existing* requests to finish within a specified timeframe (default 300s).
*   **Why it's needed:** Prevents users from experiencing "Connection Reset" errors during a deployment or scale-in event.

#### 4. Slow Start Mode
When a new instance is added to a Target Group, it might not be ready to handle 100% of the load immediately (e.g., it needs to warm up its cache).
*   **How it works:** ALB gradually increases the number of requests sent to the new target over a specified period.

---

### Industrial Examples of Target Group Usage

#### 1. Legacy to Cloud Migration (IP Target Type)
An enterprise is moving its monolithic app from an on-prem data center to AWS.
*   **Implementation:** They set up an ALB in AWS and create a Target Group of type **IP**. They register the private IPs of their **on-premise servers**.
*   **The Result:** The public-facing traffic hits the modern AWS ALB, which seamlessly routes traffic back to the legacy hardware via Direct Connect until the cloud migration is finished.

#### 2. Event-Driven Microservices (Lambda Target Type)
A travel booking site like **Expedia** uses a mixture of EC2 and Serverless.
*   **Implementation:**
    *   `/search` routes to an **Instance Target Group** (High-performance EC2).
    *   `/subscribe-newsletter` routes to a **Lambda Target Group**.
*   **The Result:** They don't need to keep a server running 24/7 just for a simple newsletter signup; they only pay for the Lambda execution when a user clicks "Submit."

#### 3. High-Security Fintech Environment (LB Chaining)
A payment gateway requires multiple layers of inspection.
*   **Implementation:** They use an Internal ALB as a target for an External ALB.
*   **The Result:** The External ALB handles the public internet and WAF, while the Internal ALB manages the specific routing to isolated, high-security transaction-processing target groups.

---

## 3. ALB Query String and Parameter Routing: The Precision Router

**Query String Routing** allows an Application Load Balancer to route traffic to specific target groups based on the key-value pairs following the `?` in a URL (e.g., `https://example.com/search?type=images`).

### How It Works
When a request hits the ALB, the listener evaluates the URL. If a **Query String Condition** is defined in the rules, the ALB looks for a match in the parameters.

*   **Key-Value Matching:** You can specify that a key (e.g., `version`) must have a specific value (e.g., `v2`).
*   **Wildcards:** ALB supports wildcards:
    *   `*` matches zero or more characters.
    *   `?` matches exactly one character.
*   **Case Sensitivity:** You can configure the match to be case-sensitive or insensitive as per your application's requirements.

---

### Why Use Query String Routing?

1.  **Precision Steering:** Route users to different backend fleets without changing the clean URL path.
2.  **A/B Testing:** Easily split traffic between stable and experimental versions.
3.  **Source Tracking:** Deliver customized content based on where the user came from (e.g., marketing campaigns).

---

### Industrial Examples of Query String Routing

#### 1. Digital Marketing: Campaign Optimization (UTM Parameters)
Companies like **Zillow** or **Expedia** spend millions on diverse marketing channels.
*   **Scenario:** Users coming from an email campaign (`?utm_source=email`) should see a "Welcome Back" landing page, while those from Google Search (`?utm_source=google`) see a generic search page.
*   **Implementation:** The ALB checks the `utm_source` key. If it matches `email`, the request is routed to a target group optimized for high-conversion marketing assets.

#### 2. Feature Flags & Beta Testing
Software companies like **SaaS** providers or **Netflix** often test new features with a small subset of users.
*   **Scenario:** You want to give "Beta" access to users who manually add `?beta=true` to their URL.
*   **Implementation:** An ALB rule checks for the `beta` parameter. If it equals `true`, the user is routed to the **"Beta Feature" Target Group** (Green environment) instead of the standard "Production" Target Group (Blue environment).

#### 3. Mobile Web Optimization
*   **Scenario:** A legacy system needs to serve a completely different, lightweight backend response to users who navigate to the site with a `?mode=lite` parameter.
*   **Implementation:** The ALB detects the `lite` parameter and routes the traffic to a fleet of micro-instances designed specifically for fast, minimal-data responses.

#### 4. Security & Bot Filtering
*   **Scenario:** Identifying and isolating requests that contain suspicious or non-standard parameters often used in automated scraping or attacks.
*   **Implementation:** The ALB can be configured to catch specific patterns in the query string (e.g., `?cmd=...` or `?exec=...`) and route them to a **HoneyPot** target group or a fixed 403 Forbidden response, protecting the main application servers.
