# Hands-On Lab: The "Jump" - Accessing Private Instances via Bastion

In this lab, we will implement the **Bastion Host** pattern. You will learn how to securely log into an instance that has no internet access by using a "Jump Box."

---

### Step 1: Resource Creation

1.  **Launch Bastion Host (Public):**
    *   **Name:** `Bastion-Host`
    *   **Subnet:** `dev-vpc-Public-Subnet-A`
    *   **Public IP:** Enabled.
    *   **Key Pair:** `deploy.pem` (Download and save this!).
2.  **Launch Private Server (Private):**
    *   **Name:** `private-server`
    *   **AMI:** Ubuntu 22.04 LTS.
    *   **Instance Type:** t3.micro.
    *   **Subnet:** `dev-vpc-Private-Subnet-A`.
    *   **Public IP:** **Disabled**.

---

### Step 2: Security Group Chaining (The Secret Sauce)

1.  **Bastion SG:** Allow Port 22 from **Your IP**.
2.  **Private-SG:** 
    *   Go to **Inbound Rules** -> **Edit**.
    *   **Type:** SSH (Port 22).
    *   **Source:** Search and select the **Security Group ID of the Bastion Host**.
    
> [!IMPORTANT]
> This "chain" ensures that the private server only recognizes traffic coming from the Bastion's bodyguard.

---

### Step 3: Connecting to the Bastion

1.  Select the `Bastion-Host` in the EC2 Dashboard.
2.  Click **Connect** -> **EC2 Instance Connect** -> **Connect**.
3.  You are now logged into the Bastion Host terminal in your browser.

---

### Step 4: Key Management on the Bastion

To connect from the Bastion to the Private Server, the Bastion needs the private key (`deploy.pem`).

1.  **Create the file:**
    ```bash
    nano deploy.pem
    ```
2.  **Paste content:** Open your downloaded `deploy.pem` on your local computer, copy everything, and paste it into the terminal.
3.  **Save:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`.
4.  **Verify:** `cat deploy.pem` (Ensure it looks correct).
5.  **Set Permissions:**
    ```bash
    chmod 0400 deploy.pem
    ```
    *(Note: 0400 means read-only for the owner. SSH will refuse to use a key that is "too open").*

---

### Step 5: The "Jump" connection

Now, run the command to jump into the private server:

```bash
ssh ubuntu@<PRIVATE_IP_OF_PRIVATE_SERVER> -i deploy.pem
```

**Result:** You will see the prompt change to `ubuntu@ip-10-x-x-x`. **Yipee!** You are now securely connected to an instance that is invisible to the public internet.

---

### Step 6: Testing Outbound Connectivity (The Surprise)

Now that you are inside the `private-server`, try to reach the internet:

```bash
ping google.com
```

**Result:** The command will hang or return "Destination Host Unreachable."

#### Why does this happen?
It's a common misconception that if you can SSH into a server, the server can talk to the internet. 
*   **The Bastion Connection:** This is **Inbound** traffic. You are "Jumping" into the VPC.
*   **The Ping Command:** This is **Outbound** traffic. 
*   **The Verdict:** Your private server is in a private subnet. It has **no NAT Gateway** and **no route to the Internet Gateway**. It is completely isolated from the outside world.

> [!TIP]
> To "fix" this outbound connectivity so the server can download updates, we would need to implement a **NAT Gateway** in the public subnet (Stay tuned for the next lab!).

---

### Organizational Usage & Best Practices

In large enterprises like **CitiBank** or **NASA**, Bastion Hosts are managed with extreme care:

1.  **Centralized Logging:** Every command typed on the Bastion is recorded and sent to a security tool (like CloudWatch Logs or Splunk) for auditing.
2.  **Identity Integration:** Users don't use shared keys. They log in with their corporate credentials (AD/LDAP) via **IAM Roles**.
3.  **Limited Window:** Access to the Bastion is often governed by a "Just-In-Time" (JIT) access request. The port is only opened for 1-2 hours after an approved ticket.
4.  **The Modern Shift:** Organizations are moving away from Bastions toward **AWS Systems Manager (SSM) Session Manager** to eliminate the need for Port 22 and PEM files entirely.
