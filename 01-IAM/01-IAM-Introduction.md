# AWS IAM: Identity and Access Management

**AWS IAM (Identity and Access Management)** is a web service that helps you securely control access to AWS resources. You use IAM to control who is **authenticated** (signed in) and **authorized** (has permissions) to use resources.

---

### 1. The Core Components of IAM

Think of IAM as the security guard of the AWS cloud. It is a **Global Service** (you don't select a region).

*   **1. Users:** Permanent identities used by people or applications (e.g., `alice`, `jenkins-user`).
*   **2. Groups:** A collection of IAM users. You apply permissions to the group, and users inherit them (e.g., an `Admins` group).
*   **3. Roles:** Temporary identities that don't have a password or access keys. They are intended to be "assumed" by services (like EC2) or users from other accounts.
*   **4. Policies:** JSON documents that define permissions. You attach policies to Users, Groups, or Roles.

---

### 2. The Golden Rule: Principle of Least Privilege
Always grant only the minimum permissions required to perform a task. If a user only needs to read from S3, do not give them `AdministratorAccess`.

---

### 3. Deep-Dive: IAM Users & Groups

#### A. IAM Users
An **IAM User** is an entity that you create in AWS to represent the person or application that uses it to interact with AWS.
*   **Key Characteristic:** They have "Long-term credentials" (Password or Access Keys).
*   **Best Practice:** Never share users. One human = One IAM User. 
*   **Scenario:** You hire a freelancer named `Bob` to update a website. You create an IAM User `Bob-Freelancer` and give him only permissions to the specific S3 bucket for the website.

#### B. IAM Groups
An **IAM Group** is a collection of IAM users. Groups let you specify permissions for multiple users, which can make it easier to manage the permissions for those users.
*   **Key Benefit:** **Scalability.** Instead of attaching a policy to 50 users individually, you put them in a group and attach it once.
*   **Inheritance:** If a user is in three groups (Admins, Devs, Billing), they inherit the permissions of all three.
*   **Caveats:** Groups are NOT an "identity" themselves. You can't assume a group.
*   **Scenario:** You have a "Finance" team. You create a Group called `FinanceGroup`, attach the `Billing` policy to it, and then simply add every new finance hire to that group.

---

### 4. IAM Policy Structure (JSON)

An IAM policy is a JSON document. Here is a typical structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": { "aws:SourceIp": "203.0.113.0/24" }
      }
    }
  ]
}
```

*   **Effect:** `Allow` or `Deny`. (Deny always wins!).
*   **Action:** The specific API call (e.g., `s3:ListBucket`).
*   **Resource:** The ARN (Amazon Resource Name) the action applies to.
*   **Condition:** Optional. Adds extra constraints (like IP address or Time).

---

### 4. Important Concepts for the Exam

#### A. Multi-Factor Authentication (MFA)
You must enable MFA for your **Root User** and highly privileged IAM users. It adds an extra layer of security beyond just a password.

#### B. Access Keys vs. Console Access
*   **Console Access:** Password + MFA.
*   **Programmatic Access:** Access Key ID + Secret Access Key. (Used for CLI/SDKs). **Never share these!**

#### C. Roles for Services (The "EC2 Role" Strategy)
**Scenario:** You have an application on EC2 that needs to upload files to S3.
*   **WRONG way:** Hardcode Access Keys into the application code. (Security risk).
*   **RIGHT way:** Create an IAM Role with S3 permissions and attach it to the EC2 instance. AWS handles temporary credentials automatically.

---

### 5. Industrial Scenarios

#### Scenario 1: The New Employee
A new developer joins. You create an **IAM User**, put them in the **DevGroup**, and that group has a policy allowing access only to specific dev resources. You enforce **MFA** on their first login.

#### Scenario 2: Cross-Account Audit
Company A needs Company B to audit their S3 buckets. Company A creates an **IAM Role** that allows `ReadOnlyAccess` and allows Company B's AWS Account ID to "assume" that role. No passwords are exchanged.

---

### 6. Solution Architect Level Exam Questions (SAA-C03 Level)

**Q1: A developer needs to access an AWS service from an application running on an EC2 instance. What is the MOST SECURE way to provide the required permissions?**
*   **A)** Create an IAM Role with the required permissions and attach it to the EC2 instance.
*   **B)** Create an IAM User with the required permissions and hardcode the access keys in the application.
*   **C)** Use the Root account's access keys for the application.
*   **Answer:** **A**. Using IAM Roles for EC2 avoids the risk of long-term credential exposure.

**Q2: You need to ensure that all users in your AWS account are using Multi-Factor Authentication (MFA). Which tool or feature helps you enforce this?**
*   **A)** IAM Security Token Service (STS).
*   **B)** IAM Policy with a `Condition` requiring MFA.
*   **C)** AWS Shield.
*   **Answer:** **B**. You can add a `Condition` to policies (e.g., `"aws:MultiFactorAuthPresent": "true"`) to deny actions if MFA is not used.

**Q3: An IAM policy has an explicit ALLOW for S3 access, but another policy attached to the same user has an explicit DENY for S3 access. What is the result?**
*   **A)** The user will have access because ALLOW takes precedence.
*   **B)** The user will be denied access because DENY always wins.
*   **C)** AWS will prompt the user to choose.
*   **Answer:** **B**. In AWS IAM, an explicit DENY always overrides any ALLOW.

**Q4: A company has 500 IAM users across different departments. The security team needs to ensure that all users in the 'Developer' department have specific permissions for EC2 and S3. What is the most EFFICIENT way to manage this?**
*   **A)** Attach the required policies to each of the 500 users individually.
*   **B)** Create an IAM Group for the Developer department, attach the policies to the group, and add the users to the group.
*   **C)** Create a single IAM User and share the credentials with all 500 developers.
*   **Answer:** **B**. Groups are the most efficient way to manage repetitive permission sets across many users.

**Q5: Which of the following statements about IAM Groups is TRUE?**
*   **A)** A group can contain other groups (nested groups).
*   **B)** A group is not an identity and cannot be identified as a Principal in a resource-based policy.
*   **C)** There is a limit of 10 users per group.
*   **Answer:** **B**. Groups are purely a management convenience; you cannot define a group as a "Principal" (only Users, Roles, and Accounts can be Principals).

---

> [!TIP]
> **Root User Tip:** The first rule of AWS is: **Stop using the Root User.** Create an IAM user with admin permissions for daily work and keep the Root account locked away with MFA.
