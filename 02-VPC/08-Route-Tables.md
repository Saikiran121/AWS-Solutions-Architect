# AWS Route Tables: The GPS of Your Network

A **Route Table** is a set of rules (called routes) that are used to determine where network traffic from your subnet or gateway is directed. Without a route table, your packets wouldn't know how to reach their destination.

---

### 1. Main vs. Custom Route Tables

Every VPC has a **Main Route Table** created by default. However, best practices suggest using **Custom Route Tables** for better control.

*   **Main Route Table:** 
    *   Automatically assigned to any subnet that doesn't have an explicit association.
    *   **The Risk:** If you add a "Public" route (to an IGW) to the Main Route Table, every new subnet you create will automatically become public. This is a major security risk.
*   **Custom Route Table:**
    *   Created by the user.
    *   Allows for granular control. You can have one for "Public Subnets" (linked to IGW) and another for "Private Subnets" (linked to NAT Gateway).

> [!CAUTION]
> **The Common Pitfall: IGW vs. Route Table**
> **Scenario:** I created an Internet Gateway (IGW) and attached it to my VPC. Does my instance have internet connectivity now?
> **Answer:** **NO.** 
> An IGW is just a "door." If your Route Table doesn't have a record (a "map") telling the subnet to use that door for `0.0.0.0/0` traffic, the door stays closed to your instances. You **must** manually add the route to bridge the gap.

---

### 2. How Routing Works (The Rules of the Road)

#### Longest Prefix Match
If multiple routes match a destination, AWS uses the **most specific route** (the one with the longest prefix).
*   **Target A:** `10.0.0.0/16` (Local)
*   **Target B:** `10.0.1.0/24` (Peered VPC)
*   **Packet Destination:** `10.0.1.5`
*   **Outcome:** The packet will go to **Target B** because `/24` is more specific than `/16`.

#### Local Route
Every route table contains a **Local Route**. This allows all subnets within the same VPC to talk to each other.
*   **Crucial Rule:** You **cannot** delete or modify the Local route.

---

### 3. Subnet Associations
*   **Rule:** A subnet can be associated with **exactly one** route table at a time.
*   **Association:** However, a single route table can be associated with **multiple** subnets.

---

### 4. Industrial Scenarios

#### Scenario A: The Multi-Tenant App
A company like **Zendesk** hosts multiple clients.
*   **Implementation:** They use separate route tables for different types of traffic. Client data subnets use a route table that points to a **Virtual Private Gateway** (VPN), while the web frontend subnets use a route table pointing to an **IGW**.
*   **The Benefit:** Physical separation of traffic paths ensures client data never touches the public internet routing path.

#### Scenario B: Firewalled Traffic (Gateway Load Balancer)
A security-conscious company like **Palantir** inspects all traffic.
*   **Implementation:** They use **Ingress Routing**. They point the IGW's route table to a security appliance (Gateway Load Balancer) before traffic ever reaches the subnets.
*   **The Benefit:** 100% of traffic is scanned for threats before entering the application layer.

---

### 5. SAA-C03 Exam Quick-Tips (The "Memory Aid")

**The "GPS / Navigator" Analogy:**
Think of the Route Table as the **GPS** in a car:
*   The **VPC** is the car.
*   The **IGW** is the highway exit.
*   The **Route Table** is the GPS map telling the car: "To get to the Internet (0.0.0.0/0), take the IGW exit."
*   If the GPS doesn't have the map for the highway, you stay on local streets (Local Route).

---

### 6. Solution Architect Exam Questions (SAA-C03 Level)

**Q1: You have a VPC with one public subnet and one private subnet. You created a new subnet but forgot to associate it with a route table. What is the routing behavior of this new subnet?**
*   **A)** It will have no connectivity at all.
*   **B)** it will automatically associate with the Main Route Table.
*   **C)** It will inherit the route table of the oldest subnet in the VPC.
*   **Answer:** **B**. Every subnet must be associated with a route table. If you don't do it explicitly, AWS does it implicitly by linking it to the Main Route Table.

**Q2: A developer is complaining that their instance in a private subnet cannot download security patches from the internet. You have already verified the Security Group and NACL allow the traffic. What should you check in the Route Table?**
*   **A)** A route to the Internet Gateway (IGW) with destination `0.0.0.0/0`.
*   **B)** A route to a NAT Gateway with destination `0.0.0.0/0`.
*   **C)** A route to the VPC Router (`10.0.0.1`).
*   **Answer:** **B**. Private subnets do not use IGWs. They use NAT Gateways to reach the internet. Ensure the Route Table for the private subnet has a route pointing to the NAT Gateway ID.

**Q3: A VPC has a primary CIDR of `10.0.0.0/16`. You add a route to `10.0.0.0/16` targeting a Peering Connection. What will happen?**
*   **A)** Traffic will be balanced between the local VPC and the peered VPC.
*   **B)** ThePeering route will be ignored because the Local route is always primary.
*   **C)** You cannot add a route that perfectly matches the local VPC CIDR.
*   **Answer:** **C**. You cannot override the Local route. AWS will prevent you from adding a route that conflicts with the CIDR of the VPC itself.
