# ELB SSL/TLS Certificates

An **SSL/TLS Certificate** allows traffic between your clients (users) and your load balancer to be encrypted in transit using the HTTPS protocol. 

In AWS, managing these certificates across fleets of servers can be tedious, which is why the Elastic Load Balancer (ELB) provides native features to handle it for you.

---

### 1. SSL Termination (Offloading)

The primary reason to attach an SSL certificate to a Load Balancer rather than directly to your EC2 instances is a concept called **SSL Termination** (or SSL Offloading).

**How it works:**
1.  **Client to ELB (Encrypted):** The client opens a secure HTTPS connection with the Load Balancer. The Load Balancer uses its attached SSL certificate to encrypt/decrypt this traffic.
2.  **ELB to EC2 (Unencrypted):** Once the Load Balancer receives the traffic, it decrypts the packet. It then forwards the plain, unencrypted HTTP traffic to the backend EC2 instances over the private VPC network.

**Why is this a best practice?**
*   **CPU Savings:** Encrypting and decrypting SSL traffic is very CPU intensive. By "offloading" this work to the Load Balancer, your EC2 instances save their CPU cycles for actual application processing.
*   **Centralized Management:** You don't have to install and renew SSL certificates on 100 individual EC2 instances; you manage one certificate on the ELB.

---

### 2. AWS Certificate Manager (ACM)

**AWS Certificate Manager (ACM)** is the native AWS service used to provision, manage, and deploy public and private SSL/TLS certificates.

*   **Free for ELB:** Public SSL certificates provisioned through ACM are **100% free** when attached to an Application Load Balancer (ALB) or Network Load Balancer (NLB).
*   **Auto-Renewal:** ACM automatically handles the dreaded chore of certificate renewals before they expire.
*   *(Exam Tip: You can also upload your own third-party SSL certificates to ACM or IAM if you bought them elsewhere).*

---

### 3. Server Name Indication (SNI)

**This is a critical topic for the SAA-C03 Exam.**

Historically, if you had a load balancer, it could only hold *one* SSL certificate. If you wanted to host `www.cats.com` and `www.dogs.com` securely, you needed to pay for and manage *two separate load balancers*.

**SNI (Server Name Indication)** solves this problem. SNI is an extension to the TLS protocol that allows a client to indicate which hostname it is trying to connect to at the very beginning of the handshake process.

**The SNI Benefit in AWS:**
*   You can attach **MULTIPLE SSL Certificates** to a single Application Load Balancer (ALB) or Network Load Balancer (NLB).
*   When a request comes in, the ALB looks at the SNI header from the client and "serves" the correct SSL certificate (`cats.com` vs `dogs.com`).
*   **Cost Savings:** This allows you to host dozens of different secure websites on a single ALB, saving significant infrastructure costs.

**Supported Services:**
*   Application Load Balancer (ALB) - **YES**
*   Network Load Balancer (NLB) - **YES**
*   CloudFront (CDN) - **YES**
*   Classic Load Balancer (CLB) - **NO** (Older generation, does not support SNI).

---

### 4. Hands-On Scenario: 4 Domains, 4 Target Groups, 1 ALB

If you have 4 different domains (e.g., `web.cats.com`, `api.cats.com`, `web.dogs.com`, `api.dogs.com`) and want to route them to 4 distinct Target Groups, you combine **SNI** with **Host Header Routing** on a single Application Load Balancer.

**The Setup Steps:**

1.  **Request Certificates (ACM):** Request a public certificate in AWS Certificate Manager (ACM) for `*.cats.com` and another one for `*.dogs.com`.
2.  **Attach to ALB (SNI):** Attach both certificates to the HTTPS (Port 443) listener on your ALB. Thanks to SNI, the ALB now knows how to decrypt traffic for both cats and dogs.
3.  **Create 4 Target Groups:** Create `WebCats-TG`, `ApiCats-TG`, `WebDogs-TG`, and `ApiDogs-TG` and register the respective EC2 instances.
4.  **Configure ALB Listener Rules (Host Routing):**
    On the ALB HTTPS Listener, you create 4 specific routing rules based on the "Host header":
    *   **Rule 1:** IF `Host Header` is `web.cats.com` -> THEN Forward to `WebCats-TG`.
    *   **Rule 2:** IF `Host Header` is `api.cats.com` -> THEN Forward to `ApiCats-TG`.
    *   **Rule 3:** IF `Host Header` is `web.dogs.com` -> THEN Forward to `WebDogs-TG`.
    *   **Rule 4:** IF `Host Header` is `api.dogs.com` -> THEN Forward to `ApiDogs-TG`.
5.  **Default Action:** Create a "Default Rule" that returns a fixed response (e.g., HTTP 404 "Not Found") if the user requests an unrecognized domain.

*Result: You have successfully secured and routed 4 completely different applications using just one load balancer, saving costs while maintaining logic separation.*

---

### 5. Hands-On: Requesting SSL (Hostinger) & Attaching to ELB

This section covers how to request a free public SSL certificate from AWS using a domain purchased on a third-party registrar like **Hostinger**, and how to attach it to both an ALB and NLB.

#### Step 1: Request a Certificate in ACM
1.  Navigate to **AWS Certificate Manager (ACM)** in the AWS Console.
2.  Click **Request a certificate** -> Select **Request a public certificate**.
3.  **Domain names:** Enter your domain name (e.g., `example.com`). Also, click *Add another name to this certificate* and enter `*.example.com` to secure all subdomains.
4.  **Validation method:** Choose **DNS validation** (Recommended).
5.  Click **Request**. The certificate status will now show as **Pending validation**.

#### Step 2: DNS Validation (in Hostinger)
AWS needs proof that you own the domain. ACM provides a `CNAME` name and a `CNAME` value.
1.  Log in to your **Hostinger** account and navigate to the **hPanel**.
2.  Select your domain and find the **DNS / Nameservers** section (DNS Zone Editor).
3.  Click **Add new record**:
    *   **Type:** `CNAME`
    *   **Name:** Paste the CNAME *name* provided by AWS (Exclude your actual domain name at the end if Hostinger appends it automatically).
    *   **Target (Points to):** Paste the CNAME *value* provided by AWS.
    *   **TTL:** Leave as default (e.g., 14400).
4.  Click **Add Record**. Wait 5-30 minutes for DNS to propagate. Once verified, the ACM status in AWS will change to **Issued**.

#### Step 3: Attaching the Certificate to an ALB
1.  Navigate to **EC2** -> **Load Balancers** -> Select your **ALB**.
2.  Scroll down to **Listeners**.
3.  If you don't have an HTTPS listener, click **Add listener**.
    *   **Protocol:** `HTTPS`
    *   **Port:** `443`
    *   **Default action:** Forward to your target group.
4.  Scroll down to the **Secure listener settings** section.
5.  Under **Default SSL/TLS certificate**, select **From ACM** and choose the exact certificate you just validated (`example.com`).
6.  Click **Add**. Your ALB is now serving HTTPS traffic.

#### Step 4: Attaching the Certificate to an NLB
1.  Navigate to **EC2** -> **Load Balancers** -> Select your **NLB**.
2.  Click **Add listener**.
3.  **Protocol:** `TLS` (Note: NLBs use TLS instead of HTTPS).
4.  **Port:** `443`
5.  **Default action:** Forward to your Target Group.
6.  Under **Secure listener settings**, choose your issued ACM certificate.
7.  Click **Add**. Your NLB is now securely terminating TLS traffic.

---

### 6. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A company hosts five different e-commerce websites (e.g., site1.com, site2.com, etc.), each requiring a unique SSL certificate. Currently, the architecture uses five separate Application Load Balancers (ALBs) to serve these sites securely. The CTO has asked you to redesign the architecture to significantly reduce costs without compromising security. What is the most cost-effective recommendation?**
*   **A)** Replace the five ALBs with five Network Load Balancers (NLBs).
*   **B)** Consolidate all five websites behind a single Application Load Balancer (ALB) and use Server Name Indication (SNI) to attach all five SSL certificates to the same listener.
*   **C)** Move the SSL certificates from the ALBs directly to the backend EC2 instances so the ALBs can be removed.
*   **D)** Migrate all websites to a single Classic Load Balancer (CLB) to support multiple certificates.
*   **Answer:** **B**. Application Load Balancers support SNI, which allows you to bind multiple SSL certificates to a single ALB. This means you can consolidate five load balancers down to one, resulting in massive cost savings. (CLBs do *not* support SNI).

**Q2: A developer is complaining that the EC2 instances in an Auto Scaling Group are experiencing high CPU utilization during traffic spikes. The instances are hosting an HTTPS web application, and the SSL certificates are currently installed directly on the EC2 instances. What architectural change should the Solutions Architect implement to improve CPU performance on the instances?**
*   **A)** Increase the instance type of the EC2 instances.
*   **B)** Enable Sticky Sessions on the Load Balancer.
*   **C)** Move the SSL certificate to the Application Load Balancer (ALB) and configure SSL Termination (Offloading), forwarding traffic to the instances over HTTP.
*   **D)** Enable Cross-Zone Load Balancing on the ALB.
*   **Answer:** **C**. SSL encryption/decryption is computationally expensive. By moving the certificate to the ALB (SSL Termination/Offloading), the ALB handles the heavy lifting, freeing up the EC2 instances' CPU for application logic.

**Q3: A company needs to provision Public SSL certificates for a new web application deployed behind an Application Load Balancer (ALB). They want a solution that is free and handles automatic renewals. Which AWS service should they use?**
*   **A)** AWS Secrets Manager
*   **B)** AWS Identity and Access Management (IAM)
*   **C)** Amazon Route 53
*   **D)** AWS Certificate Manager (ACM)
*   **Answer:** **D**. ACM provisions public SSL certificates for free (when used with AWS services like ALB/CloudFront) and natively handles automatic renewals.

---

> [!TIP]
> **Exam Strategy:** If an exam question mentions hosting **"multiple domains"** or **"multiple SSL certificates"** on a single load balancer to **"save costs,"** the fundamental tech you need to look for in the answer is **SNI (Server Name Indication)**.
