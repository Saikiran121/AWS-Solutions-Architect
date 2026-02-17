# Hands-On: Building a Custom VPC Architecture

This lab guide will walk you through the process of building a highly secure and scalable Virtual Private Cloud (VPC) from scratch.

---

### Step 1: Creating the VPC (Logical Container)

The first step is to create the VPC itself. This acts as the isolated "boundary" for all your network resources.

1.  **Navigate to VPC Dashboard:** Log in to the AWS Management Console and search for **VPC**.
2.  **Start Creation:** Click the **Create VPC** button.
3.  **Configure Settings:**
    *   **Resources to create:** Select **VPC only**. (This ensures we build the components manually for learning purposes).
    *   **Name tag:** Enter `dev-vpc`.
    *   **IPv4 CIDR block:** Select **IPv4 CIDR manual input**.
    *   **IPv4 CIDR:** Enter `10.0.0.0/16`.
    *   **IPv6 CIDR block:** Select **No IPv6 CIDR block**.
    *   **Tenancy:** Select **Default**.
4.  **Tags:** Ensure a tag exists with:
    *   **Key:** `name`
    *   **Value:** `dev-vpc`
5.  **Finalize:** Click **Create VPC**.

---

### Verification
Once created, you will see your `dev-vpc` in the VPC list. 
*   **State:** Should be `Available`.
*   **CIDR:** Should be `10.0.0.0/16`.

> [!IMPORTANT]
> At this stage, your VPC is just an empty container. It has no subnets, no internet access, and no security groups. It is essentially a "network in a box" waiting for you to define its interior.

---

### Step 2: Expanding Your Network (Multiple CIDR Blocks)

AWS allows you to grow your VPC's address space by adding secondary CIDR blocks. This is useful if you run out of IP addresses and don't want to create a whole new VPC.

1.  **Select your VPC:** Go to the VPC list and select `dev-vpc`.
2.  **Open Edit Menu:** Click the **Actions** button and select **Edit CIDRs**.
3.  **Add New Range:**
    *   Click the **Add new IPv4 CIDR** button.
    *   **IPv4 CIDR:** Enter a new range, such as `10.1.0.0/16`. (Note: This must not overlap with your existing CIDR or any peered VPCs).
4.  **Save:** Click **Save**.

**The Result:** Your VPC now has a much larger pool of addresses. You can create new subnets using this secondary range just like you would with the primary range.

---

### Technical Note: CIDR Block Limits
*   **Default Limit:** By default, you can add up to **5 IPv4 CIDR blocks** to a single VPC (1 Primary + 4 Secondary).
*   **Soft Limit:** This is a soft limit. You can request an increase from AWS Support to add up to **50 CIDR blocks** per VPC.
*   **IPv6:** You can associate exactly **one** IPv6 CIDR block with a VPC.
