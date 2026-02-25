# Cross-Zone Load Balancing

By default, an ELB distributes incoming traffic evenly across its enabled **Availability Zones (AZs)**, and then the load balancer node in that AZ distributes the traffic only to the instances *within that specific AZ*.

**Cross-Zone Load Balancing** changes this behavior. When enabled, each load balancer node distributes traffic across all registered targets in **all enabled Availability Zones**.

---

### 1. The "Imbalance" Scenario (Why We Need It)

Imagine an Auto Scaling Group spanning two AZs, but due to scaling policies or manual intervention, the instance count is unbalanced.
*   **Availability Zone A:** 2 EC2 Instances
*   **Availability Zone B:** 8 EC2 Instances
*   **Total:** 10 Instances

#### Scenario A: Cross-Zone Load Balancing is DISABLED
1.  The DNS resolves to the Load Balancer, and traffic is split **50/50** between AZ-A and AZ-B.
2.  **AZ-A receives 50% of the traffic.** It only has 2 instances, so each instance gets **25%** of the total load. They are severely overloaded!
3.  **AZ-B receives 50% of the traffic.** It has 8 instances, so each instance gets a mere **6.25%** of the load. They are practically idle.

#### Scenario B: Cross-Zone Load Balancing is ENABLED
1.  Traffic hits the Load Balancer globally.
2.  The Load Balancer looks at *all 10 instances across both AZs* and routes traffic evenly among them.
3.  **Every single instance** (whether in AZ-A or AZ-B) receives exactly **10%** of the total load.

**The Benefit:** Cross-Zone Load Balancing guarantees even traffic distribution, preventing individual instances from being overwhelmed when AZ capacities are unequal.

---

### 2. ALB vs. NLB vs. GWLB (Exam Essentials)

The default behavior and the *pricing* for Cross-Zone Load Balancing differ dramatically depending on the type of load balancer you use.

| Load Balancer | Enabled by Default? | Inter-AZ Data Transfer Charges? |
| :--- | :--- | :--- |
| **Application Load Balancer (ALB)** | **YES** | **FREE**. No charges for data crossing AZ lines. |
| **Network Load Balancer (NLB)** | **NO** | **YOU PAY.** You incur standard AWS data transfer charges between AZs. |
| **Gateway Load Balancer (GWLB)** | **NO** | **YOU PAY.** You incur standard AWS data transfer charges between AZs. |

*Note: For Classic Load Balancer (CLB), it is disabled by default, but free if enabled. (Rarely tested on the modern SAA exam).*

---

### 3. How to Enable Cross-Zone Load Balancing

For an **ALB**, you usually don't have to do anything—it is on by default and cannot be disabled.

For an **NLB**, if you want to enable it (keeping the costs in mind):
1.  Go to the **EC2 Dashboard** -> **Load Balancers**.
2.  Select your NLB.
3.  Click the **Attributes** tab -> **Edit**.
4.  Toggle **Cross-Zone Load Balancing** to `On`.

---

### 4. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: An application is running on EC2 instances across two Availability Zones (AZ-A and AZ-B) behind a Network Load Balancer (NLB). AZ-A currently has 2 instances and AZ-B has 8 instances. The sysadmin notices that the instances in AZ-A have 100% CPU utilization, while the instances in AZ-B hover around 10%. What is the MOST likely cause of this issue?**
*   **A)** The EC2 instances in AZ-B are failing their health checks.
*   **B)** Sticky Sessions are enabled on the NLB.
*   **C)** Cross-Zone Load Balancing is disabled on the NLB.
*   **D)** The NLB does not support routing to multiple Availability Zones.
*   **Answer:** **C**. By default, an NLB splits traffic evenly by AZ, not by instance. AZ-A gets 50% of the traffic dumped onto just 2 instances, causing the overload. Enabling Cross-Zone load balancing would distribute the load evenly across all 10 instances.

**Q2: A company is designing a high-throughput video streaming service using a Network Load Balancer (NLB) across three Availability Zones. To ensure perfect capacity utilization, the architect wants to enable Cross-Zone Load Balancing on the NLB. What cost considerations should the architect be aware of?**
*   **A)** Enabling Cross-Zone Load Balancing requires upgrading the NLB to a higher service tier.
*   **B)** There are no additional costs; AWS includes cross-zone data transfer for free on all load balancers.
*   **C)** The company will incur standard regional data transfer charges for traffic moving between instances in different Availability Zones.
*   **D)** The NLB hourly charge will double.
*   **Answer:** **C**. Unlike ALBs (where cross-zone traffic is free), enabling Cross-Zone Load Balancing on an NLB or GWLB incurs standard inter-AZ data transfer costs for the traffic routed across zones.

**Q3: An e-commerce site recently migrated from a Network Load Balancer (NLB) to an Application Load Balancer (ALB) to take advantage of path-based routing. The infrastructure team forgot to enable the "Cross-Zone Load Balancing" setting during the ALB creation. What will happen to the traffic distribution if the Auto Scaling Group has 2 instances in AZ-1 and 6 instances in AZ-2?**
*   **A)** The instances in AZ-1 will receive significantly more traffic per instance than AZ-2.
*   **B)** The traffic will be distributed evenly to all 8 instances automatically.
*   **C)** The ALB will fail to route traffic until the setting is explicitly enabled.
*   **Answer:** **B**. For Application Load Balancers (ALB), Cross-Zone Load Balancing is **enabled by default at the load balancer level**. Therefore, traffic will be distributed perfectly evenly across all 8 instances without any explicit action required locally.

---

> [!TIP]
> **Exam Strategy:** If you see a question detailing an **unfair/imbalanced CPU load** where one zone is dying while another is idle (especially with an NLB), the answer is almost always **enable Cross-Zone Load Balancing**. Remember: **ALB = Free & Default**, **NLB = You Pay & Disabled by Default**.
