# S3 Bucket Policy: Centralized Access Control

An **S3 Bucket Policy** is a resource-based policy written in **JSON** that you attach directly to an S3 bucket. It allows you to manage access to the bucket and the objects within it, including cross-account access.

---

### 1. The Anatomy of a Bucket Policy

AWS JSON policies follow a standard structure. Every statement contains:

*   **Sid (Statement ID):** An optional identifier for the statement.
*   **Effect:** Either `Allow` or `Deny`.
*   **Principal:** *Who* is affected (e.g., a specific IAM user, an account, or `*` for everyone).
*   **Action:** *What* they can do (e.g., `s3:GetObject`, `s3:PutObject`).
*   **Resource:** The target of the action (e.g., the bucket ARN or `arn:aws:s3:::bucket-name/*` for objects).
*   **Condition (Optional):** When the policy is in effect (e.g., only from a specific IP).

---

### 2. Common Industrial Scenarios

#### Scenario A: Public Read Access (Static Websites)
Used when you want anyone in the world to be able to view objects (like images or HTML files).
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-website-bucket/*"
    }
  ]
}
```

#### Scenario B: Enforcing Encryption (HTTPS Only)
Industry standard for security compliance. This policy denies any request that doesn't use SSL/TLS.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EnforceHTTPS",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::my-secure-bucket/*",
      "Condition": {
        "Bool": { "aws:SecureTransport": "false" }
      }
    }
  ]
}
```

#### Scenario C: Restricting to Corporate IP
Ensures that data can only be accessed from your office network.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyOfficeIP",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::confidential-bucket/*",
      "Condition": {
        "NotIpAddress": { "aws:SourceIp": "203.0.113.0/24" }
      }
    }
  ]
}
```

---

### 3. IAM Policy vs. S3 Bucket Policy

| Feature | IAM Policy | Bucket Policy |
| :--- | :--- | :--- |
| **Attached To** | Users, Groups, Roles | The S3 Bucket itself |
| **Used For** | Managing "What can THIS user do?" | Managing "Who can access THIS bucket?" |
| **Max Size** | ~2 KB to 10 KB | **20 KB** (Allows for complex logic) |
| **Cross-Account** | Difficult | **Primary way to allow other accounts** |

---

### 4. Solution Architect Exam Questions (SAA-C03 Level)

**Q1: A company wants to host a static website on S3. They have uploaded the files, but visitors get a 403 Forbidden error. Which component should be used to grant public read access to all objects?**
*   **A)** IAM Policy attached to the developer.
*   **B)** S3 Bucket Policy with a Principal of "*".
*   **C)** S3 ACL on each individual object.
*   **Answer:** **B**. Bucket policies are the most efficient way to grant bucket-wide public access.

**Q2: A security auditor requires that all data uploaded to an S3 bucket must be encrypted in transit. How can you automate this enforcement?**
*   **A)** Enable default encryption on the bucket.
*   **B)** Use a Bucket Policy with a "Deny" effect and the "aws:SecureTransport" condition.
*   **C)** Manually encrypt objects before uploading.
*   **Answer:** **B**. Denying non-HTTPS traffic is the standard way to enforce encryption in transit.

**Q3: Which JSON element in an S3 Bucket Policy defines the ARN of the bucket or objects?**
*   **A)** Principal
*   **B)** Action
*   **C)** Resource
*   **Answer:** **C**. The Resource element defines the ARN.

---

> [!IMPORTANT]
> **Explicit Deny wins!** If an IAM policy says "Allow" but a Bucket Policy says "Deny", the request will be **DENIED**. In AWS, an explicit Deny always overrides an Allow.
