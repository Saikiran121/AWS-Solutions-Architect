# Hands-On: Building the Subnet Architecture

Now that we have our `dev-vpc` container, it's time to carve out the internal network blocks: the **Subnets**. We will build a highly-available architecture with 4 subnets across 2 Availability Zones.

---

### Step 1: Creating the Subnets

1.  **Navigate to Subnets Dashboard:** In the VPC Console, click on **Subnets** in the left sidebar.
2.  **Start Creation:** Click the **Create subnet** button.
3.  **Select VPC:** In the **VPC ID** dropdown, select your `dev-vpc`.
4.  **Configure Subnets:** Use the **Add new subnet** button at the bottom to configure all 4 subnets at once.

| # | Subnet Name | Availability Zone | IPv4 CIDR Block | Capacity |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `dev-vpc-Public-Subnet-A` | `ap-south-1a` | `10.0.0.0/24` | 251 Usable IPs |
| **2** | `dev-vpc-Public-Subnet-B` | `ap-south-1b` | `10.0.1.0/24` | 251 Usable IPs |
| **3** | `dev-vpc-Private-Subnet-A` | `ap-south-1a` | `10.0.16.0/20` | 4,091 Usable IPs |
| **4** | `dev-vpc-Private-Subnet-B` | `ap-south-1b` | `10.0.32.0/20` | 4,091 Usable IPs |

5.  **Finalize:** Once all 4 subnets are configured, click **Create subnet**.

---

### Key Information Explained

#### Why `/24` for Public and `/20` for Private?
In a typical industrial architecture:
*   **Public Subnets (/24):** Are usually small. They only host ELBs, NAT Gateways, and maybe a few Bastion hosts. They don't need thousands of IPs.
*   **Private Subnets (/20):** Are large. They host your actual application servers, microservices, and containers. As you scale with Auto Scaling Groups, you need a large pool of addresses to avoid "IP Exhaustion."

#### Why two different Availability Zones?
By placing subnets in both `ap-south-1a` and `ap-south-1b`, we ensure **High Availability**. If an entire data center (AZ) in Mumbai goes down, our resources in the other AZ will continue to function.

---

### Verification
*   Filter the list by your VPC ID.
*   You should see all 4 subnets with the state **Available**.
*   **Total usable IPs:** Notice that AWS has automatically subtracted **5 IPs** from each range (e.g., 256 - 5 = 251).

> [!NOTE]
> **What about the "Public" label?** 
> Even though we named them "Public," they are currently **Statistically Private**. They will only become truly "Public" once we attach an Internet Gateway and update their Route Tables in the next steps.
