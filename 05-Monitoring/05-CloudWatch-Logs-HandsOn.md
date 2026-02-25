# Hands-On Lab: Streaming EC2 Ubuntu Logs to CloudWatch

This lab demonstrates the heavily-tested process of extracting application or system logs from a running Amazon EC2 instance and streaming them centrally into Amazon CloudWatch Logs. 

By default, an EC2 instance does *not* send its local hard drive logs to AWS. This hands-on will show you exactly how to bridge that gap using the unified **CloudWatch Agent**.

---

### Step 1: Create the IAM Role

For the software on the EC2 instance to talk to the CloudWatch API, it needs permissions.

1.  Navigate to **IAM** in the AWS Console.
2.  Click **Roles** -> **Create role**.
3.  **Trusted entity type:** Select **AWS service**.
4.  **Use case:** Select **EC2**, then click Next.
5.  **Permissions policies:** Search for and select the AWS-managed policy named `CloudWatchAgentServerPolicy`. Click Next.
6.  **Role name:** Name it `EC2-CloudWatch-Role`.
7.  Click **Create role**.

---

### Step 2: Launch the EC2 Instance

Now we launch the server and attach the permission role we just created.

1.  Navigate to **EC2** -> **Launch instance**.
2.  Give it a name (e.g., `Log-Streaming-App`).
3.  Select the **Ubuntu** AMI.
4.  Scroll down to **Advanced details**.
5.  Find the **IAM instance profile** dropdown and select the `EC2-CloudWatch-Role` you created in Step 1.
6.  Click **Launch instance**.

*(Note: Ensure your Security Group allows SSH access so you can log in to perform the next steps).*

---

### Step 3: Install the CloudWatch Agent

SSH into your newly launched Ubuntu EC2 instance and follow these steps to download and install the agent software.

1.  **Update the system:**
    ```bash
    sudo apt update -y
    ```

2.  **Download the CloudWatch Agent package:**
    ```bash
    wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
    ```

3.  **Install the downloaded package:**
    ```bash
    sudo dpkg -i amazon-cloudwatch-agent.deb
    ```

4.  **Verify the Installation:**
    Run the generic help command to verify the agent CLI tool is installed:
    ```bash
    amazon-cloudwatch-agent-ctl -h
    ```
    *If installed correctly, you will see a list of usage help commands. The agent binaries are now installed at `/opt/aws/amazon-cloudwatch-agent/`.*

---

### Step 4: Create the CloudWatch Agent Configuration

The software is installed, but it doesn't know *what* logs to look for. We need to create a JSON configuration file telling it to read the Ubuntu system log (`/var/log/syslog`).

1.  Open a new file using the `nano` text editor:
    ```bash
    sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
    ```

2.  Paste the following JSON configuration into the editor:
    ```json
    {
      "logs": {
        "logs_collected": {
          "files": {
            "collect_list": [
              {
                "file_path": "/var/log/syslog",
                "log_group_name": "/ec2/syslog",
                "log_stream_name": "{instance_id}"
              }
            ]
          }
        }
      }
    }
    ```
    *(**Explanation:** This tells the agent to open the `/var/log/syslog` file, send its contents to a CloudWatch Log Group named `/ec2/syslog`, and dynamically name the Log Stream after the specific EC2 Instance ID using the `{instance_id}` variable).*

3.  Save and exit `nano` (Press `CTRL+O`, `Enter`, then `CTRL+X`).

---

### Step 5: Start the Agent

Now we tell the agent to start running in the background and use the configuration file we just built.

1.  **Start the agent** using the `fetch-config` command:
    ```bash
    sudo amazon-cloudwatch-agent-ctl \
    -a fetch-config \
    -m ec2 \
    -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
    -s
    ```

2.  **Verify it is running** using `systemd`:
    ```bash
    sudo systemctl status amazon-cloudwatch-agent
    ```
    *You should see the text `active (running)` highlighted in green.*

---

### Step 6: Verify in the AWS Console

Let's prove the text from the hard drive actually made it to the cloud.

1.  Navigate to **CloudWatch** in the AWS Console.
2.  On the left-hand menu, click **Logs** -> **Log groups**.
3.  You should automatically see a new group named `/ec2/syslog` appear. (The agent created it automatically thanks to the IAM role we attached in Step 2).
4.  Click on `/ec2/syslog`.
5.  Under **Log streams**, you will see a stream named exactly like your EC2 Instance ID (e.g., `i-0123abcd5678ef90`).
6.  Click the stream. You will see the live, real-time Linux system logs streaming directly to your screen!

*(You could now create a **Metric Filter** to scan this stream for words like "error" or "failed" and attach a CloudWatch Alarm).*
