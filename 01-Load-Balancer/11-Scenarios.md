# Load Balancer Troubleshooting Scenarios

Even perfectly designed architectures fail. When AWS Load Balancers fail, you usually don't get an explicit "EC2 Instance 4 is broken" alert; instead, users report that "the website is down."

This guide covers how to systematically troubleshoot complex Load Balancer setups.

---

### Scenario: The Failing 4-Target Group ALB

**The Premise:** You have a single Application Load Balancer (ALB) routing traffic to 4 distinct Target Groups (e.g., `web`, `api`, `auth`, `payment`) using Host Header routing or Path-Based routing. 

Everything was running smoothly, but suddenly, the ALB stops serving traffic correctly. How do you troubleshoot this, and how do you know *which* of the 4 target groups is failing?

---

### Phase 1: Identifying the Failure Source

Your first step is to figure out if the ALB itself is failing, or if a specific backend Target Group is failing.

#### 1. Check CloudWatch Metrics (The Broad View)
Go to the **CloudWatch** console and view the metrics for your specific ALB. Look at two critical metric graphs:
*   `HTTPCode_ELB_5XX_Count`: If this is spiking, the **Load Balancer itself** is generating the errors (e.g., it can't find any targets).
*   `HTTPCode_Target_5XX_Count`: If this is spiking, the **backend EC2 instances** are generating the errors and sending them back to the ALB (e.g., your app code crashed).

#### 2. Check ALB Access Logs (The Narrow View)
To find out *which* of the 4 Target Groups is causing the issue, you must enable **Access Logs** on the ALB (which write to an S3 bucket).
*   You use **Amazon Athena** to query these access logs.
*   The logs explicitly show the `target_group_arn` and the `domain/path` requested. By querying the logs for 5XX errors, you can instantly see if the `payment` Target Group is throwing 500s while the `web` Target Group is fine.

---

### Phase 2: Decoding the HTTP Error Codes

Once you identify the failing Target Group in the logs, look at the specific HTTP Error Code. The ALB uses the 5XX range to tell you *exactly* what went wrong.

#### HTTP 502 (Bad Gateway)
*   **What it means:** The ALB sent the request to the EC2 instance, but the instance returned a malformed or invalid response.
*   **Common Causes:** 
    *   The web server (e.g., Nginx, Apache) on the EC2 instance crashed.
    *   The EC2 instance closed the connection prematurely.
    *   **Crucial:** A Security Group change blocked traffic between the ALB and the EC2 instance.

#### HTTP 503 (Service Unavailable)
*   **What it means:** The ALB has nowhere to send the traffic.
*   **Common Causes:**
    *   **Zero Healthy Instances:** Every single EC2 instance in the specific Target Group has failed its Health Check.
    *   The Target Group is completely empty (no instances registered).
    *   The Target Group itself was accidentally deleted, but the Listener Rule is still trying to route to it.

#### HTTP 504 (Gateway Timeout)
*   **What it means:** The ALB sent the request to the EC2 instance, and then waited... and waited... until it hit its `Idle Timeout` limit (default 60 seconds), and gave up.
*   **Common Causes:**
    *   The backend application is experiencing massive load and is too slow to respond.
    *   The EC2 instance is locked up waiting on a slow Database query.
    *   The backend web server is taking longer to process the request than the ALB's configured Idle Timeout.

---

### Phase 3: The Step-by-Step Troubleshooting Flow

Once you know the Error Code and the failing Target Group, follow this triage path:

1.  **Check Target Group Health:** Go to the EC2 Console -> Target Groups. Look at the specific failing group. Are the registered targets showing `Healthy` or `Unhealthy`?
    *   *If Unhealthy:* Click on the instance to see the "Health Check Status Details" (e.g., "Connection timed out" vs. "HTTP 404 response").
2.  **Verify Security Groups:** 
    *   Check the ALB Security Group to ensure it allows inbound from the Internet (0.0.0.0/0) on 80/443.
    *   Check the EC2 (Target) Security Group to ensure it allows inbound traffic *only* from the ALB's Security Group on the application port (e.g., 8080).
3.  **Check Network ACLs:** Verify that the Subnet NACLs haven't been modified to block inbound traffic on the app port, or outbound traffic on Ephemeral Ports (1024-65535).
4.  **Check Application Logs (SSH/SSM):** If the network is fine and the instances are "Healthy" but returning 500s (`HTTPCode_Target_5XX_Count`), you must SSH into the instance or use CloudWatch Logs to read the actual application stack trace (e.g., "Database connection refused").

---

> [!CAUTION]
> **Exam Trap:** On the SAA-C03 exam, if an application behind an ALB is returning `504 Gateway Timeout` errors, the answer is almost always to **troubleshoot the Web Server or Database performance**, NOT to add more load balancers. The timeout means the backend is too slow, not that the network is broken.
