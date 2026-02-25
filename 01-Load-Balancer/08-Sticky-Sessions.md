# Sticky Sessions (Session Affinity)

By default, an Application Load Balancer (ALB) routes each request independently to the registered target with the smallest amount of outstanding requests.

However, sometimes you need the same client (user) to be routed to the **same backend EC2 instance** for the duration of their session. This is called **Sticky Sessions** (or Session Affinity).

---

### 1. The "Shopping Cart" Scenario (Why We Use It)

Imagine you are running a legacy e-commerce application. 
1.  **Request 1:** A user adds a laptop to their cart. The ALB routes this request to **EC2 Server A**. The application saves the cart data in the local RAM of Server A.
2.  **Request 2:** The user goes to checkout. The ALB, doing its normal job, routes this request to **EC2 Server B**.
3.  **The Problem:** Server B doesn't know about the cart data stored on Server A. The user sees an empty cart.

**The Solution:** Enable Sticky Sessions on the ALB's Target Group. Once enabled, after the first request, the ALB will ensure that **Request 2** (and all subsequent requests from that user) are sent back to **EC2 Server A**.

---

### 2. How It Works (The Magic of Cookies)

Sticky sessions work by issuing a cookie to the client's browser. When the client makes subsequent requests, they send the cookie back, and the ALB reads it to know which server the user belongs to.

There are two main types of stickiness:

#### A. Duration-Based Stickiness (ALB Generated)
*   **The Default Method:** The load balancer generates a special cookie named `AWSALB`.
*   **How it works:** You specify a duration (e.g., 1 hour). For that hour, as long as the cookie is present, traffic goes to the same instance.
*   **Best For:** Simple applications where you just need the user to stay on one box.

#### B. Application-Based Stickiness (Custom Generated)
*   **How it works:** Your application code dictates the stickiness by generating its own custom cookie (e.g., `MyAppSessionCookie`).
*   **The Advantage:** The stickiness follows the lifecycle of the *application's* session. If the user logs out and your app deletes the cookie, the stickiness immediately ends.
*   **Note:** The ALB still intercepts this and adds its own `AWSALBApp` cookie to map the custom cookie to the target instance.

---

### 3. The Downside of Sticky Sessions

While sticky sessions solve the "lost session" problem for legacy apps, they introduce new architectural flaws:

1.  **Imbalance:** If one user is downloading massive files or generating huge reports, that specific EC2 instance gets overloaded, while other instances sit completely idle. The load balancer can't redistribute their traffic because they are "stuck."
2.  **Loss of High Availability:** If Server A crashes, the user loses their session entirely because their local data went down with the server.

#### The Modern Alternative: Stateless Architecture
The AWS Best Practice is to design **Stateless Applications**. Instead of storing the shopping cart in Server A's local RAM, the application should write the cart data to a fast, centralized database like **Amazon DynamoDB** or **Amazon ElastiCache (Redis)**. 
*   Now, it doesn't matter if Request 2 goes to Server B; Server B simply retrieves the cart from ElastiCache. No stickiness needed!

---

### 4. Hands-On: How to Enable Sticky Sessions

If you want to enable Sticky Sessions for an Application Load Balancer in the AWS Management Console, follow these steps:

1.  **Navigate to EC2 Dashboard:** Open the AWS Management Console and go to the **EC2** service.
2.  **Access Target Groups:** On the left-hand navigation pane, scroll down to the **Load Balancing** section and click on **Target Groups**.
3.  **Select the Target Group:** Click on the name of the Target Group that is associated with your ALB (e.g., `my-app-tg`).
4.  **Edit Attributes:**
    *   Click on the **Attributes** tab.
    *   Click the **Edit** button.
5.  **Enable Stickiness:**
    *   Find the **Stickiness** section.
    *   Check the box to **Turn on stickiness**.
6.  **Configure Stickiness Type:**
    *   Choose **Load balancer generated cookie** (Duration-based) or **Application-based cookie**.
    *   If using **Load balancer generated cookie**, set the **Stickiness duration** (e.g., 1 day).
    *   If using **Application-based cookie**, enter the exact **Cookie name** that your application generates (e.g., `MyAppSessionID`) and set the duration.
7.  **Save changes.**
    *   Once saved, the ALB will immediately begin issuing the `AWSALB` or `AWSALBApp` cookies and routing users to their previously assigned instances.

---

### 5. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A legacy web application stores user session data locally in the memory of EC2 instances. Users are complaining that they randomly get logged out or lose their shopping cart items while browsing. The instances are behind an Application Load Balancer (ALB). How can a Solutions Architect fix this issue with the LEAST amount of application code changes?**
*   **A)** Migrate the session data to Amazon ElastiCache.
*   **B)** Enable Sticky Sessions (Session Affinity) on the ALB Target Group.
*   **C)** Replace the ALB with a Network Load Balancer (NLB).
*   **D)** Store the session data in an Amazon EFS volume.
*   **Answer:** **B**. Enabling sticky sessions requires zero application code changes and ensures the user is repeatedly routed to the specific instance holding their local session state in memory.

**Q2: A company uses Sticky Sessions on their Application Load Balancer to maintain user state. However, CloudWatch metrics show that the CPU utilization of the EC2 instances in the Auto Scaling Group is highly imbalanced, with one instance running at 95% while others are at 10%. What architectural change should the Solutions Architect recommend to permanently resolve this imbalance?**
*   **A)** Switch to Application-Based cookie stickiness instead of Duration-Based.
*   **B)** Enable Cross-Zone Load Balancing on the ALB.
*   **C)** Redesign the application to be stateless and offload session data to Amazon ElastiCache for Redis.
*   **Answer:** **C**. Sticky sessions inherently cause traffic imbalances (e.g., a "heavy" user gets stuck to one server). Offloading state to a centralized store like ElastiCache makes the application stateless, allowing the ALB to perfectly balance every individual request.

**Q3: A Solutions Architect is configuring Application-Based stickiness for a Target Group attached to an Application Load Balancer. Which cookie name is reserved by the ALB and CANNOT be used as the custom application cookie name?**
*   **A)** `MyAppSession`
*   **B)** `JESSIONID`
*   **C)** `AWSALB`
*   **D)** `UserToken`
*   **Answer:** **C**. `AWSALB` (and `AWSALBApp`, `AWSALBTG`) are reserved cookie names used by the Application Load Balancer itself to manage routing.

---

> [!TIP]
> **Exam Strategy:** If an exam question asks for the **fastest/easiest/least-code** way to fix dropped sessions on an ALB, the answer is **Sticky Sessions**. If the question asks for the **most scalable/highly-available/best-practice** way, the answer is to **make the app stateless using ElastiCache or DynamoDB**.
