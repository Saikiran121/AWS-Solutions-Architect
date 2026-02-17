# Hands-On: Mastering Network ACLs (Subnet Security)

In this lab, we will learn how to manually control traffic flow at the subnet level. We will use a custom **NACL** to block and then allow access to a web server (Nginx).

---

### Step 1: Creating a Custom NACL

1.  Navigate to the **VPC Dashboard** -> **Network ACLs**.
2.  Click **Create network ACL**.
3.  **Name tag:** `Dev-NACL`.
4.  **VPC:** Select your `dev-vpc`.
5.  Click **Create network ACL**.

---

### Step 2: Subnet Association

By default, a new NACL isn't protecting anything. We must attach it to a subnet.

1.  Select your new `Dev-NACL` from the list.
2.  Click the **Subnet associations** tab -> **Edit subnet associations**.
3.  Select the subnets you want to protect (e.g., your Public Subnets).
4.  Click **Save changes**.

> [!IMPORTANT]
> A subnet can only be associated with **one** NACL at a time. When you associate the `Dev-NACL`, the subnet will automatically drop its connection to the Default NACL.

---

### Step 3: The "Deny" Test (Blocking Nginx)

Let's say you have an Nginx server running on an instance in your public subnet. To block all web traffic to it:

1.  With `Dev-NACL` selected, click **Inbound rules** -> **Edit inbound rules**.
2.  Click **Add new rule**.
3.  **Rule number:** `100` (Lower numbers have higher priority!).
4.  **Type:** `HTTP (80)`.
5.  **Source:** `0.0.0.0/0`.
6.  **Action:** Select **DENY**.
7.  Click **Save changes**.

**Result:** Your Nginx server is now completely inaccessible from the outside world on Port 80, even if the Security Group allows it!

---

### Step 4: Configuring Outbound Traffic

NACLs are **Stateless**. If you want a request to be fully successful, you must also allow the response to leave.

1.  Click the **Outbound rules** tab -> **Edit outbound rules**.
2.  Click **Add new rule**.
3.  **Rule number:** `100`.
4.  **Type:** `HTTP (80)`.
5.  **Destination:** `0.0.0.0/0`.
6.  **Action:** Select **ALLOW**.
7.  Click **Save changes**.

> [!CAUTION]
> **The Stateless Trap:** Even after Step 4, your website might still fail. Why? Because you allowed Port 80 Outbound, but the client's browser is likely waiting for a response on a random **Ephemeral Port** (1024-65535). 
> 
> To fix this for real production, you would need an Outbound rule allowing **Custom TCP** on ports **1024-65535**.
