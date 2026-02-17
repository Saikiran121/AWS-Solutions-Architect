# Network Load Balancer (NLB): The Performance Powerhouse

The Network Load Balancer (NLB) operates at **Layer 4** (the Connection Layer) of the OSI model. It is designed to handle millions of requests per second while maintaining ultra-low latency. Unlike the ALB, which looks at the content of the message, the NLB only cares about the network connection itself (IPs and Ports).

---

### 1. What is a Network Load Balancer?
An NLB is a high-performance load balancer that functions at the TCP, UDP, and TLS levels. 
*   **Speed:** It can handle sudden, volatile traffic patterns that would overwhelm an ALB.
*   **Latency:** It offers millisecond latency, making it the choice for real-time applications.
*   **Scale:** It is capable of scaling to millions of requests per second automatically.

---

### 2. When to Use NLB?
You should choose an NLB over an ALB in these specific scenarios:

*   **Non-HTTP Protocols:** If your application uses TCP, UDP, or specialized protocols like MQTT or SIP.
*   **Static IP Addresses:** NLB provides one **Static IP (or Elastic IP)** per Availability Zone. This is crucial if your clients need to whitelist specific IPs in their firewalls.
*   **Extreme Performance:** For applications requiring the highest throughput and lowest possible latency.
*   **Preserving Source IP:** If your backend servers absolutely need to see the original client's IP address without using headers like `X-Forwarded-For`.

---

### 3. How Traffic Flows (The Flow Hash)
NLB uses a high-performance **Flow Hash Algorithm** to route traffic.

1.  **The Selection:** When a request arrives, the NLB selects a target from the target group based on a hash of the 5-tuple: (Source IP, Source Port, Destination IP, Destination Port, and Protocol).
2.  **Flow Stickiness:** Once the connection is established, all packets for that specific flow are routed to the same target for the life of the connection.
3.  **Source IP Preservation:** For TCP and UDP traffic, the NLB preserves the client's source IP and sends it directly to the backend. The backend sees the client's IP as if the load balancer wasn't there.

---

### 4. Target Groups in NLB
Target Groups for NLB differ slightly from ALB in terms of protocols and behavior.

*   **Protocols Supported:** TCP, TLS, UDP, and TCP_UDP.
*   **Target Types:**
    *   **Instances:** Register EC2 instances directly.
    *   **IP Addresses:** Route to private IPs (VPC, Peered VPC, or On-Premise).
    *   **ALB:** You can put an ALB behind an NLB (useful if you need a Static IP for an ALB).
*   **Connection Draining:** Like ALB, NLB supports deregistration delay to ensure existing connections are not abruptly terminated.

---

### 5. Health Checks
Because NLB operates at Layer 4, its health checks are focused on connection availability:

*   **TCP Health Check:** The NLB attempts to open a TCP connection to the target on a specific port. If the "handshake" succeeds, the target is `Healthy`.
*   **HTTP/HTTPS Health Check:** Even though it's a Layer 4 balancer, NLB can perform Layer 7 health checks (sending an HTTP request) to ensure the application itself is responsive, not just the network port.

---

### Industrial Examples of NLB

#### 1. Online Gaming: Riot Games (League of Legends)
Gaming companies handle massive amounts of **UDP traffic** for real-time player movement and combat.
*   **Implementation:** They use NLB to distribute millions of player connections across game server fleets. The low latency of NLB ensures that players don't experience "lag" during critical gameplay moments.

#### 2. Financial Trading Platforms: NASDAQ
In high-frequency trading (HFT), microseconds mean millions of dollars.
*   **Implementation:** These platforms use NLBs to handle incoming order requests. The millisecond latency and extreme throughput of NLB allow the system to process trade orders with virtually zero delay.

#### 3. Enterprise Security: Corporate Whitelisting
Many large corporations (like **Goldman Sachs** or **IBM**) have strict firewall policies that only allow outbound traffic to specific, known IP addresses.
*   **Implementation:** A SaaS provider serving these clients will use an NLB with **Elastic IPs**. This allows the clients to whitelist those specific IPs forever. If the SaaS used an ALB (which has changing IPs), the client's firewall would eventually block the connection.

#### 4. Ad-Tech Pipielines: The Trade Desk
Ad-tech platforms must decide which ad to show in less than 100 milliseconds while handling billions of requests daily.
*   **Implementation:** They use NLBs as the front door for their real-time bidding infrastructure. The NLB's ability to handle sudden "bursty" traffic ensures the platform never drops a request during peak internet usage hours.
