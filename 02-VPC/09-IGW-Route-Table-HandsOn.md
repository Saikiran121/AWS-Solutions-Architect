# Hands-On Lab: Troubleshooting Internet Connectivity

This lab demonstrates why simply launching an instance in a custom VPC isn't enough for internet access. We will perform a step-by-step "Fix-it" mission to bridge the gap between our isolated network and the world.

---

### Step 1: The Initial Problem (No Reachability)

**Scenario:**
*   You launched an EC2 instance in a public subnet of our custom `dev-vpc`.
*   **Security Groups:** You allowed Port 22 (SSH) and Port 80 (HTTP).
*   **User Data:** You included a script to install and start `nginx`.

**The Result:**
*   You try to access the public IP:80 in your browser, but it **Times Out**.
*   You try to connect via **EC2 Instance Connect**, but it fails with a "Network Connectivity" error.

**Why?** Because even though the instance has a front door (Security Group), there is no road (Route Table) leading out of the community and no main gate (Internet Gateway) at the perimeter.

---

### Step 2: Creating the Gate (Internet Gateway)

1.  Navigate to **Internet Gateways** in the VPC Dashboard.
2.  Click **Create internet gateway**.
3.  **Name tag:** Enter `Dev-IGW`.
4.  Click **Create internet gateway**.

---

### Step 3: Attaching the Gate

Created IGWs are initially in a `detached` state. They are useless until attached to a specific VPC.

1.  Select `Dev-IGW`.
2.  Click **Actions** -> **Attach to VPC**.
3.  **VPC:** Select your `dev-vpc`.
4.  Click **Attach internet gateway**.

---

### Step 4: Re-Verification (Still Failing!)

Even though the IGW is attached, you **still cannot SSH into your instance**.
**The Reason:** The subnets don't know the IGW exists yet. They are using the "Local" route only.

---

### Step 5: Creating the Navigators (Route Tables)

We need separate "Maps" for our Public and Private subnets.

1.  **Public Route Table:**
    *   Navigate to **Route Tables** -> **Create route table**.
    *   **Name:** `Dev-Public-RT`.
    *   **VPC:** Select `dev-vpc`.
    *   Click **Create route table**.
2.  **Private Route Table:**
    *   Click **Create route table**.
    *   **Name:** `Dev-Private-RT`.
    *   **VPC:** Select `dev-vpc`.
    *   Click **Create route table**.

---

### Step 6: Associating Subnets (Linking Maps to Blocks)

1.  **Public Association:**
    *   Select `Dev-Public-RT`.
    *   Go to **Subnet associations** tab -> **Edit subnet associations**.
    *   Select your **Public Subnets** (A and B).
    *   Click **Save association**.
2.  **Private Association:**
    *   Select `Dev-Private-RT`.
    *   Go to **Subnet associations** tab -> **Edit subnet associations**.
    *   Select your **Private Subnets** (A and B).
    *   Click **Save association**.

---

### Step 7: Final Check (Still No Internet?)

Believe it or not, you **still can't reach the instance**.
**The Reason:** Your "Public" map currently only knows how to find things inside the same neighborhood (the Local route). It doesn't have directions for "everywhere else" (`0.0.0.0/0`).

---

### Step 8: The "Magic Fix" (The Default Route)

1.  Select `Dev-Public-RT`.
2.  Go to the **Routes** tab -> **Edit routes**.
3.  Click **Add route**.
4.  **Destination:** Enter `0.0.0.0/0`.
5.  **Target:** Select **Internet Gateway** and pick `Dev-IGW`.
6.  Click **Save changes**.

---

### Step 9: Final Success!

*   **Browser:** Refresh your Public IP. You should now see the "Welcome to nginx!" page.
*   **SSH:** You can now successfully connect to your instance via EC2 Instance Connect or your terminal.

**Summary of the Chain:**
`Instance` -> `Security Group` -> `Subnet` -> `Route Table (0.0.0.0/0)` -> `Internet Gateway` -> `Public World`.
If any link in this chain is missing, connectivity breaks.
