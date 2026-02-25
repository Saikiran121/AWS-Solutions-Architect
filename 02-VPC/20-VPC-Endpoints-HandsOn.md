# Hands-On: Accessing S3 via VPC Gateway Endpoints

In this lab, we will optimize our network by removing the reliance on a NAT Gateway for S3 access. We will move S3 traffic onto the **private AWS backbone** using a **Gateway Endpoint**.

---

### Step 1: Pre-Requisites & Initial State
1.  Ensure you have a **Private EC2 Instance** (e.g., `private-server`) in a private subnet.
2.  Ensure your **Private Route Table** has a route to the **NAT Gateway** (to allow the initial IAM role check and CLI setup).
3.  SSH into your private server (via the Bastion or Jenkins host).

---

### Step 2: Creating the IAM Role for S3 Access
Even with a VPC Endpoint, your EC2 instance needs **permission** to talk to S3.

1.  Go to the **IAM Dashboard** -> **Roles** -> **Create role**.
2.  **Trusted entity type:** AWS Service.
3.  **Use case:** Select **EC2**. Click **Next**.
4.  **Add permissions:** Search for `AmazonS3ReadOnlyAccess`. Check it and click **Next**.
5.  **Role name:** `PrivateServer-EC2-S3ReadOnly`.
6.  Click **Create role**.

---

### Step 3: Attaching the Role to your EC2 Instance
1.  Go to the **EC2 Dashboard** -> **Instances**.
2.  Select your `private-server`.
3.  Click **Actions** -> **Security** -> **Modify IAM Role**.
4.  Select `PrivateServer-EC2-S3ReadOnly` from the dropdown.
5.  Click **Update IAM role**.

---

### Step 4: Verify Public Access (The Control Test)
1.  Back in your terminal (SSH'd into `private-server`):
    ```bash
    aws s3 ls
    ```
2.  **Result:** It works! But currently, this traffic is traveling through your **NAT Gateway** to the public internet and back into AWS. This is expensive and less secure.

---

### Step 5: Breaking the Connection (Removing NAT Gateway)
Before creating the endpoint, let's prove that S3 access currently depends on the NAT Gateway.

1.  Go to the **VPC Dashboard** -> **Route Tables**.
2.  Select your **Private Route Table**.
3.  Click **Routes** tab -> **Edit routes**.
4.  Remove the route with destination `0.0.0.0/0` (pointing to the NAT Gateway).
5.  Click **Save changes**.
6.  Back in your terminal (SSH'd into `private-server`), run:
    ```bash
    aws s3 ls
    ```
7.  **Result:** The command will fail because there is no path to S3.

---

### Step 6: Creating the VPC Endpoint
Now, let's restore access using a private path.

1.  Go to the **VPC Dashboard** -> **Endpoints**.
2.  Click **Create endpoint**.
3.  **Name tag:** `dev-vpc-endpoint`.
4.  **Service category:** AWS services.
5.  **Service name:** Search for `s3` and select the one with **Type: Gateway** (e.g., `com.amazonaws.us-east-1.s3`).
6.  **VPC:** Select your `dev-vpc`.
7.  **Route tables:** Select your **Private Route Table** (e.g., `Dev-Private-RT`).
8.  Leave all other settings (Policy, Tags) as default.
9.  Click **Create endpoint**.

---

### Step 7: Verification (Route Table Entry)
AWS has automatically updated your route table to handle S3 traffic.

1.  Go to **Route Tables** -> Select **Dev-Private-RT**.
2.  Click the **Routes** tab.
3.  You will now see the following entry in the destination list:

| Destination | Target | Status | Propagated | Route Origin |
| :--- | :--- | :--- | :--- | :--- |
| `pl-78a54011` (S3 Prefix List) | `vpce-0e75339622051280e` | **Active** | No | Create Route |

---

### Step 8: Final Validation (The Private Road)
1.  In your `private-server` terminal, run:
    ```bash
    aws s3 ls
    ```
2.  **Result:** It works without a NAT Gateway! 
3.  **Conclusion:** This confirms your traffic is now staying entirely within the Amazon network, saving cost and improving security.
