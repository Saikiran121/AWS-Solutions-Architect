# Hands-On: Creating a Secure AWS S3 Bucket

In this lab, we will walk through the process of creating your first **S3 Bucket** using modern AWS security best practices.

---

### Step 1: Navigate to S3
1.  Log in to your **AWS Management Console**.
2.  Search for **S3** in the search bar and select it.
3.  Click the orange **Create bucket** button.

---

### Step 2: General Configuration
1.  **Bucket Type:** Select **General purpose**. (This is the standard for most use cases).
2.  **Bucket Name:** Enter a unique name (e.g., `dev-vpc-assets-saikiran-123`).
    *   *Remember:* It must be **globally unique** and follow the naming conventions (lowercase only).
3.  **AWS Region:** Select the region closest to you (e.g., `us-east-1`).

---

### Step 3: Object Ownership (Modern Security)
In the past, AWS used Access Control Lists (ACLs) to manage object permissions. Today, it is best practice to use **IAM policies** and **Bucket Policies** instead.

1.  **Object Ownership:** Select **ACLs disabled (recommended)**.
2.  By doing this, you ensure that you (the bucket owner) have full control over all objects uploaded to the bucket.

---

### Step 4: Block Public Access settings for this bucket
By default, AWS enables "Block all public access." **Keep this enabled!**

1.  Ensure **Block all public access** is checked.
2.  This acts as a "Guardrail" that prevents anyone from accidentally making your private data public.

---

### Step 5: Remaining Settings
1.  **Bucket Versioning:** Select **Disable** (You can enable this later if needed for deletion protection).
2.  **Default Encryption:** Leave as **Server-side encryption with Amazon S3 managed keys (SSE-S3)**.
3.  **Advanced settings:** Leave everything as default.

---

### Step 6: Create Bucket
1.  Scroll to the bottom and click **Create bucket**.

**Result:** Your secure, private bucket is now ready! You can now start uploading files.

---

### Step 7: Uploading Objects & Folders

Now that your bucket is created, let's add some data.

1.  Click on your bucket name from the list.
2.  Click the **Upload** button.
3.  Click **Add files** and select a file from your computer.
    *   *Observation:* You can see the **File Size** and **Type** before uploading.
4.  Click **Create folder**, name it `images`, and click **Create**.
5.  Open the `images` folder and upload an image there.
    *   *Observation:* Notice the **Destination** path updates to `s3://your-bucket-name/images/`.

---

### Step 8: Understanding Object Properties

1.  Click on an uploaded object (e.g., `logo.png`).
2.  In the **Properties** tab, you will see:
    *   **Owner:** Your AWS Account.
    *   **Last modified:** Timestamp of upload.
    *   **Size:** Exact bytes.
    *   **Key:** The full path (e.g., `images/logo.png`).
    *   **S3 URI:** The internal AWS identifier (e.g., `s3://my-bucket/logo.png`).
    *   **Object URL:** The public HTTP address.

---

### Step 9: The "Open" vs. "Object URL" Mystery

**The Problem:** 
*   If you click the **Open** button, the file opens perfectly in your browser.
*   If you copy the **Object URL** and paste it into a new tab, you get: `<Code>AccessDenied</Code>`.

**The Explanation:**
1.  **"Open" uses a Pre-signed URL:** When you click the S3 console's "Open" button, AWS automatically generates a temporary, secure link that includes your credentials. It's like a one-time "VIP Pass."
2.  **"Object URL" is the Public Door:** This is the standard, permanent path to the file. Since we enabled **Block all public access** in Step 4, S3 is doing its job and locking the door to everyone without a "VIP Pass."

---

### Step 10: Where do we use the S3 URI?

The **S3 URI** (`s3://bucket/key`) is not for browsers. It is used for:
1.  **AWS CLI:** `aws s3 cp s3://my-bucket/file.txt .`
2.  **AWS SDKs:** In Python (Boto3) or Java code to identify which file to process.
3.  **Service Integrations:** Connecting S3 to other AWS services like **Athena**, **Redshift**, or **SageMaker**.
