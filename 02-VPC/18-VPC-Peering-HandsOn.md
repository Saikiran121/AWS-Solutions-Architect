# Hands-On: Peering VPCs for Private Connectivity

In this lab, we will connect two isolated networks: the **Default VPC** and our custom **Dev-VPC**. This will allow instances in different VPCs to communicate using their **Private IP addresses**.

---

### Step 0: The Setup (Pre-requisites)

To see the peering in action, ensure you have the following:

1.  **BastionHost** (in `Dev-VPC`): Running Nginx on Port 80.
2.  **JenkinsServer** (in `Default VPC`): Running Jenkins on Port 8080.

---

### Step 1: The "Isolation" Test (Connectivity Failure)

Before we peer, let's prove they can't talk to each other.

1.  SSH into the **BastionHost**.
2.  Try to reach Jenkins in the Default VPC:
    ```bash
    curl <Jenkins-Private-IP>:8080
    ```
3.  **Result:** The command hangs and eventually says `Connection Timed Out`. This is because there is no path between these two VPCs.

---

### Step 2: Creating the Peering Connection

1.  Go to the **VPC Dashboard** -> **Peering connections**.
2.  Click **Create peering connection**.
3.  **Name tag:** `Dev-Peering`.
4.  **VPC (Requester):** Select your `Default VPC`.
5.  **VPC (Accepter):** 
    *   **Account:** My account.
    *   **Region:** This region.
    *   **VPC ID:** Select your `Dev-VPC`.
6.  Click **Create peering connection**.

---

### Step 3: Accepting the Request

Since both VPCs are in your account, the status will show "Pending Acceptance."

1.  Select `Dev-Peering`.
2.  Click **Actions** -> **Accept request**.
3.  The status will change to `Active`.

---

### Step 4: The "Most Forgotten" Step (Route Tables)

Even though they are peered, the routers in each VPC don't know about it yet. You must add routes in **BOTH** VPCs.

#### 1. Update Dev-VPC Route Table
*   Go to **Route Tables** -> Select `Dev-Public-RT`.
*   Click **Routes** tab -> **Edit routes**.
*   Click **Add route**.
*   **Destination:** Enter the **CIDR of the Default VPC** (usually `172.31.0.0/16`).
*   **Target:** Select **Peering Connection** -> `Dev-Peering`.
*   Click **Save changes**.

#### 2. Update Default VPC Route Table
*   Repeat the same process for your **Default VPC's Route Table**.
*   **Destination:** Enter the **CIDR of the Dev-VPC** (`10.0.0.0/16`).
*   **Target:** `Dev-Peering`.
*   Click **Save changes**.

---

### Step 5: Final Verification (Success!)

1.  Go back to your SSH session on the **BastionHost**.
2.  Try the `curl` command again:
    ```bash
    curl <Jenkins-Private-IP>:8080
    ```
3.  **Result:** You should now see the Jenkins HTML output in your terminal! 

**Congratulations!** You have successfully enabled private cross-account communication between two VPCs.
