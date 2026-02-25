# AWS Gateway Load Balancer (GWLB)

**Gateway Load Balancer (GWLB)** is a specialized load balancer designed specifically for deploying, scaling, and managing **third-party virtual networking appliances** such as firewalls, Intrusion Detection and Prevention Systems (IDS/IPS), and Deep Packet Inspection systems.

It combines a **transparent network gateway** (acting as a single entry/exit point) with a **load balancer** to distribute traffic across a fleet of virtual appliances.

---

### 1. Why Do We Need Gateway Load Balancers?

Before GWLB, deploying third-party firewalls (like Palo Alto, Fortinet, or Check Point) in AWS was complex. It required intricate route table management, complex failover scripts, and creating bottlenecks.

**The GWLB Solution:**
*   **"Bump-in-the-wire" Architecture:** GWLB is completely transparent to the source and destination. It intercepts traffic, sends it to the firewall for inspection, and then forwards it along to its original destination without altering the packet (source and destination IPs remain identical).
*   **Auto-Scaling Appliances:** You can put your expensive third-party firewalls in an Auto Scaling Group behind the GWLB, allowing them to scale out and in based on network load.
*   **Fault Tolerance:** If a firewall appliance fails its health check, GWLB automatically stops sending traffic to it and reroutes it to healthy appliances.

---

### 2. How It Works: The Magic of GENEVE

GWLB operates at **Layer 3** (Network) and **Layer 4** (Transport) of the OSI model. However, its magic lies in how it communicates with the appliances behind it.

*   **The GENEVE Protocol:** GWLB encapsulates the original network traffic using the **GENEVE (Generic Network Virtualization Encapsulation)** protocol on **UDP port 6081**.
*   **Why Encapsulation?** By putting the original packet inside a GENEVE wrapper, the firewall appliance receives the exact, unmodified original packet (with the original source and destination IP addresses intact) so it can perform accurate inspection.
*   **Return Trip:** Once the firewall inspects and approves the packet, it sends it back to the GWLB (still encapsulated in GENEVE), which then unwraps it and forwards the raw packet to the final destination.

---

### 3. GWLB Architecture Components

A typical GWLB setup involves two main components, often crossing VPC boundaries using PrivateLink mechanics:

1.  **Gateway Load Balancer (GWLB):** This lives in an "Appliance VPC" or "Security VPC" and load balances traffic across the fleet of firewall EC2 instances.
2.  **Gateway Load Balancer Endpoint (GWLBE):** This lives in your "Application VPC". It acts as a routable target in your VPC Route Tables. You update your route tables to send all internet-bound or inter-VPC traffic to this endpoint first.

---

### 4. Industrial Scenarios

#### Scenario 1: Centralized Outbound Internet Inspection (Egress)
A strict financial company requires that all outgoing internet traffic from its private EC2 instances must be inspected for data exfiltration by a 3rd-party firewall.
*   **Implementation:** The Private Subnet Route Table has a default route (`0.0.0.0/0`) pointing to the **GWLBE**. The GWLBE sends traffic over to the GWLB, which distributes it to the firewalls. Once inspected and approved, the firewall sends it back to the GWLBE, which then routes it out the NAT Gateway to the Internet.

#### Scenario 2: East-West Traffic Inspection (VPC-to-VPC)
A company needs to inspect traffic flowing between an HR VPC and a Finance VPC connected via an AWS Transit Gateway.
*   **Implementation:** Traffic leaving the HR VPC towards the Finance VPC is routed first to a GWLBE. The traffic is hauled over to the centralized Security VPC for inspection by the GWLB-managed firewalls before continuing safely to the Finance VPC.

---

### 5. GWLB vs. ALB vs. NLB

| Feature | ALB | NLB | GWLB |
| :--- | :--- | :--- | :--- |
| **Layer** | 7 (HTTP/HTTPS) | 4 (TCP/UDP) | 3/4 (IP packets) |
| **Primary Use Case** | Web applications, microservices | Ultra-high performance, low latency | **3rd-party Virtual Appliances (Firewalls)** |
| **Header Modification** | Modifies headers (adds `X-Forwarded-For`) | Preserves Source IP | **Completely Transparent** (GENEVE encapsulation) |

---

### 6. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A company wants to strictly inspect all internet-bound traffic from its private subnets using a third-party Intrusion Prevention System (IPS) appliance marketplace image. The architecture must be highly available and scale automatically. Which AWS service should the Solutions Architect use?**
*   **A)** Network Load Balancer (NLB)
*   **B)** Gateway Load Balancer (GWLB)
*   **C)** NAT Gateway
*   **D)** VPC Traffic Mirroring
*   **Answer:** **B**. Gateway Load Balancer (GWLB) is specifically designed to deploy, scale, and manage third-party virtual appliances like IPS/IDS systems transparently.

**Q2: A security team is configuring a Gateway Load Balancer to route traffic to a fleet of firewall appliances. Which protocol and port must the firewall appliances support to communicate with the GWLB?**
*   **A)** HTTP on TCP port 80
*   **B)** HTTPS on TCP port 443
*   **C)** GENEVE on UDP port 6081
*   **D)** IPsec on UDP port 500
*   **Answer:** **C**. GWLB uses the GENEVE protocol (UDP port 6081) to encapsulate and pass traffic to and from the virtual appliances while keeping the original IP headers intact.

**Q3: A Solutions Architect is designing an architecture to inspect traffic moving laterally between multiple VPCs (East-West traffic) using centralized Palo Alto firewalls. How should the architect route traffic to these firewalls located in a separate "Security VPC"?**
*   **A)** Point the VPC Route Tables directly to the Elastic IP of the firewalls.
*   **B)** Use VPC Peering and put the firewalls behind an Application Load Balancer.
*   **C)** Deploy Gateway Load Balancer Endpoints (GWLBE) in the application VPCs and point the VPC route tables to these endpoints.
*   **Answer:** **C**. You use Gateway Load Balancer Endpoints (GWLBE) as routed targets in your VPC route tables to seamlessly redirect traffic to the centralized GWLB for inspection.

---

> [!TIP]
> **Exam Strategy:** If a question mentions "third-party firewall," "virtual appliance," or "IDS/IPS" combined with "scaling" or "high availability", the answer is almost universally **Gateway Load Balancer**.
