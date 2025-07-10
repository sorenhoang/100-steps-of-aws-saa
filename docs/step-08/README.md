# Step 08: S3 Bucket Policies vs IAM Policies

## 🎯 Objective
- [x] Master: **S3 Bucket Policies vs IAM Policies**

## 📘 Notes

### **Deep Dive: S3 Bucket Policies vs. IAM Policies**

When controlling access to Amazon S3, AWS provides two primary types of policies: IAM policies (identity-based) and S3 bucket policies (resource-based). While both use a similar JSON structure, they are attached to different entities and serve different purposes. Understanding how they work together is key to securing your data in S3.

### **1. IAM Policies (Identity-Based Policies)** 👤

IAM policies are attached to **IAM principals**—that is, a user, group, or role. They define what actions that specific identity is allowed or denied to perform.

- **Attached To:** A user, a group that a user belongs to, or a role that a user or service assumes.
- **What it Answers:** "What can **this user/role** do?"
- **Scope:** An IAM policy can grant permissions to many different services (EC2, RDS, S3, etc.) all within a single policy document. It is not limited to just S3.
- **Use Cases:**
    - Managing permissions for a large number of users by attaching a policy to an IAM group.
    - Granting an EC2 instance (via its role) permissions to access multiple services, including an S3 bucket.
    - Defining the permissions for a specific human user across the entire AWS environment.

Example IAM Policy:

This policy, when attached to a user John, allows him to get objects from my-example-bucket. It says nothing about what other users can do.

JSON

# 

`{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-example-bucket/*"
        }
    ]
}`

### **2. S3 Bucket Policies (Resource-Based Policies)** 🪣

S3 bucket policies are attached directly to an **S3 bucket** itself. They define who has access to that specific bucket and what actions they can perform on it.

- **Attached To:** An S3 bucket (the resource).
- **What it Answers:** "Who can access **this S3 bucket**?"
- **Scope:** A bucket policy can only grant permissions to the bucket it is attached to and the objects within it. It cannot grant permissions to other resources like EC2 or DynamoDB.
- **Key Feature - Cross-Account Access:** Bucket policies are the primary and simplest way to grant access to users or roles from **other AWS accounts**. You specify the ARN of the external account or user in the `Principal` element.
- **Use Cases:**
    - Granting public read access to a bucket (e.g., for website hosting).
    - Forcing all incoming objects to be encrypted by denying `s3:PutObject` if the encryption header isn't present.
    - Granting another AWS account access to put objects into your bucket (e.g., for receiving logs from another service).

Example S3 Bucket Policy:

This policy, attached to my-example-bucket, allows any user in account 111122223333 to get objects from it.

JSON

# 

`{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": { "AWS": "arn:aws:iam::111122223333:root" },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-example-bucket/*"
        }
    ]
}`

### **3. The Policy Evaluation Logic** ⚖️

When a user tries to access an S3 object, AWS evaluates all applicable policies to make a final decision. The logic is as follows:

1. **Start with "Deny":** By default, all requests are denied.
2. **Check for an Explicit `Deny`:** AWS checks all applicable policies—IAM policies, S3 bucket policy, and even SCPs in AWS Organizations. If **any** of these policies has an `Effect` of `Deny` for the requested action, the request is **immediately denied**. An explicit `Deny` always wins.
3. **Look for an Explicit `Allow`:** If there are no explicit `Deny` statements, AWS then checks for any policy with an `Effect` of `Allow`. For access within the **same account**, an `Allow` in **either** the IAM policy **or** the S3 bucket policy is sufficient to grant access. For **cross-account** access, **both** the IAM policy (in the requesting account) and the S3 bucket policy (in the resource account) must grant the permission.

**Summary of Evaluation:**

| Access Type | IAM Policy (User) | S3 Bucket Policy (Resource) | Result |
| --- | --- | --- | --- |
| **Any** | Explicit `Deny` | *Doesn't Matter* | **DENIED** |
| **Any** | *Doesn't Matter* | Explicit `Deny` | **DENIED** |
| **Same Account** | `Allow` | *Not specified* | **ALLOWED** |
| **Same Account** | *Not specified* | `Allow` | **ALLOWED** |
| **Cross-Account** | `Allow` (in requesting account) | `Allow` (for requesting account) | **ALLOWED** |
| **Cross-Account** | `Allow` (in requesting account) | *Not specified* | **DENIED** |
| **Cross-Account** | *Not specified* | `Allow` (for requesting account) | **DENIED** |
| **Same/Cross Account** | No `Allow` specified | No `Allow` specified | **DENIED** |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. An IAM user in your AWS account is trying to upload an object to an S3 bucket, but the request is being denied. You have checked the user's IAM policies, and they explicitly allow the `s3:PutObject` action on the target bucket. There are no `Deny` statements in the IAM policy. What is the most likely reason for the access denial?**

- A. The S3 bucket is in a different region than the IAM user.
- B. The S3 bucket policy contains an explicit `Deny` statement for the `s3:PutObject` action for that user.
- C. The user has not enabled MFA for their account.
- D. The S3 bucket's Access Control List (ACL) is misconfigured.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An explicit Deny in any applicable policy always overrides an Allow. Even though the user's IAM policy allows the action, a Deny statement in the resource-based S3 bucket policy will take precedence and block the request. While ACLs (D) are another access control mechanism, an explicit Deny in a bucket policy is a more common and powerful reason for this scenario.

</details>
    

---

**2. A company wants to grant read-only access to a specific S3 bucket to an external auditor who has their own AWS account. What is the most secure and direct way to grant this cross-account access?**

- A. Create a new IAM user in your account for the auditor and provide them with the access keys.
- B. Modify the S3 bucket policy to grant `s3:GetObject` permission to the auditor's AWS account ARN.
- C. Make the S3 bucket public and send the URL to the auditor.
- D. Create an IAM role in your account that the auditor can assume, and attach an S3 read-only policy to it.
<details>
<summary>View Answer</summary>

**Answer: B or D**

**Explanation:** Both B and D are valid and secure methods for cross-account access.

- **Option B (Bucket Policy)** is often considered the most direct way to grant access to a specific S3 resource. You add a statement to the bucket policy that specifies the auditor's account ARN as the `Principal`.
- **Option D (IAM Role)** is also a very common and secure best practice. You create a role in your account that has the necessary permissions and a trust policy that allows the auditor's account to assume it.

For the SAA-C03 exam, both are correct patterns. Often, if the question is only about access to a single S3 bucket, the bucket policy is presented as the simplest answer. If access to multiple resources is needed, a role is better.

</details>
        

---

**3. Which type of policy would you use to enforce that all objects uploaded to a specific S3 bucket must be encrypted using server-side encryption (SSE-S3)?**

- A. An IAM policy attached to the users who are uploading objects.
- B. An S3 bucket policy attached to the bucket.
- C. A Service Control Policy (SCP) attached to the account's OU.
- D. An S3 Access Control List (ACL).
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic use case for an S3 bucket policy. You can create a policy with a Deny effect for the s3:PutObject action if the request does not include the x-amz-server-side-encryption header. This effectively forces all uploads to be encrypted, regardless of who is uploading the object. Applying this rule at the resource (the bucket) is more efficient and reliable than managing it for every user (A).

</details>
    

---

**4. A user has an IAM policy that allows `s3:GetObject`. The S3 bucket they are trying to access has a bucket policy that also allows `s3:GetObject` for that user. However, there is an explicit `Deny` for `s3:GetObject` in an IAM permissions boundary attached to the user. What is the result?**

- A. The user can get the object because two policies allow it.
- B. The user cannot get the object because the permissions boundary's explicit `Deny` wins.
- C. The user can get the object because IAM policies are evaluated before permissions boundaries.
- D. The result is unpredictable due to a policy conflict.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The evaluation logic is clear: an explicit Deny in any policy along the evaluation path (SCP, permissions boundary, IAM policy, bucket policy) will always override any Allow statements. In this case, the Deny in the permissions boundary takes precedence, and the request is denied.

</details>
    

---

**5. What is a primary difference between an IAM policy and an S3 bucket policy?**

- A. IAM policies use JSON format, while S3 bucket policies use YAML.
- B. An IAM policy is identity-based (attached to a user/role), while an S3 bucket policy is resource-based (attached to a bucket).
- C. IAM policies cannot grant cross-account access.
- D. S3 bucket policies can grant permissions to other services like EC2 and DynamoDB.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the fundamental distinction. IAM policies are attached to an identity and define what that identity can do. S3 bucket policies are attached to a resource (the bucket) and define who can access that resource. Both use JSON (A is false). IAM policies are used in cross-account access via roles (C is false). S3 bucket policies are limited to S3 permissions only (D is false).

</details>
    

---

**6. An EC2 instance, using an IAM role, needs to access an S3 bucket in a different AWS account. Which of the following MUST be configured for the access to succeed?**

- A. The IAM role must have a policy allowing access to the S3 bucket, AND the S3 bucket policy must grant access to that IAM role.
- B. Only the IAM role's policy needs to be configured.
- C. Only the S3 bucket policy needs to be configured.
- D. The EC2 instance must have an Elastic IP address.
<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** For cross-account access, permissions must be granted on both sides. The identity (the EC2 instance's role in the source account) must have an IAM policy allowing it to perform the action. The resource (the S3 bucket in the destination account) must have a resource-based policy (a bucket policy) that explicitly allows the identity's ARN to perform the action.

</details>
    

---

**7. In an S3 bucket policy, what does the `Principal` element specify?**

- A. The AWS service that the policy applies to.
- B. The specific API actions that are allowed or denied.
- C. The user, account, service, or other entity that is allowed or denied access to the resource.
- D. The conditions under which the policy is in effect.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Principal element is used to identify the identity that the policy statement applies to. This can be an AWS account ARN, an IAM user ARN, an IAM role ARN, a service principal (like ec2.amazonaws.com), or even * for anonymous access. IAM policies do not have a Principal element because the principal is the user/group/role the policy is attached to.

</details>
    

---

**8. An application running in your account needs to access an S3 bucket in your account. A solutions architect has configured the permissions by adding a statement to the S3 bucket policy that allows the application's IAM role. No IAM policy is attached to the role itself. Will the application be able to access the bucket?**

- A. No, because an IAM role must always have its own IAM policy.
- B. Yes, because for access within the same account, an `Allow` in either the identity policy OR the resource policy is sufficient.
- C. No, because S3 bucket policies are only for cross-account access.
- D. Yes, but only if the bucket and the application are in the same region.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a key rule for same-account access. As long as there are no explicit Deny statements, an Allow from either the identity side (IAM policy) or the resource side (S3 bucket policy) is enough to grant permission. Since the bucket policy allows the role, access is granted.

</details>
    

---

**9. When would you choose to use an S3 bucket policy over an IAM policy?**

- A. When you need to manage permissions for a large number of S3 buckets from one central location.
- B. When you need to grant simple cross-account access to a single S3 bucket.
- C. When you need to grant permissions to an IAM user for multiple services like EC2 and S3 simultaneously.
- D. When you are managing permissions for IAM groups.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A bucket policy is the most straightforward way to grant another account access to that specific bucket. It's a single policy on the resource itself. An IAM policy (A) would be better for managing one user's access to many buckets. For multiple services (C) or group management (D), IAM policies are the correct tool.

</details>
    

---

**10. You see the following `Principal` in an S3 bucket policy: `"Principal": "*"`. What does this indicate?**

- A. All IAM users within the same AWS account are granted access.
- B. The policy grants access to a specific AWS service.
- C. The policy grants access to everyone, including anonymous users on the internet.
- D. The policy is invalid and will not work.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The wildcard "*" in the Principal element of a resource-based policy means "everyone" or "anonymous." This is how you configure public access to an S3 bucket. This should be used with extreme caution.

</details>
    

---

**11. An IAM role has a policy attached that grants `s3:GetObject`. This role is assumed by a Lambda function to read data from an S3 bucket. The S3 bucket policy does not mention this role at all. Assuming both the function and bucket are in the same account, will the Lambda function succeed?**

- A. No, the bucket policy must also explicitly allow the Lambda role.
- B. Yes, an `Allow` in the IAM policy is sufficient for same-account access.
- C. No, Lambda functions cannot use resource-based policies.
- D. Yes, because Lambda functions have default read access to all S3 buckets in the same account.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This again demonstrates the rule for same-account access. As long as there isn't an explicit Deny in the bucket policy, the Allow from the identity policy (attached to the Lambda's execution role) is sufficient to grant access. Lambda functions do not have default access to S3 (D).

</details>
    

---

**12. Which policy type is attached to an IAM user?**

- A. Resource-based policy.
- B. Trust policy.
- C. Identity-based policy.
- D. S3 bucket policy.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Policies attached to identities (users, groups, roles) are called identity-based policies. Resource-based policies are attached to resources (like an S3 bucket). Trust policies are a specific type of resource-based policy attached to an IAM role that defines who can assume it.

</details>
    

---

**13. A company's security policy requires all data uploaded to a bucket named `finance-logs` to be stored with the `S3 Intelligent-Tiering` storage class. How can this be enforced?**

- A. Use an IAM policy to deny uploads if the storage class is not correct.
- B. Use an S3 bucket policy to deny the `s3:PutObject` action unless the `x-amz-storage-class` header is set to `INTELLIGENT_TIERING`.
- C. Configure S3 Lifecycle policies to move the objects to Intelligent-Tiering after one day.
- D. This cannot be enforced; it can only be monitored.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is an excellent use case for a bucket policy. By using a Deny statement with a condition that checks the value of the x-amz-storage-class request header, you can enforce the use of a specific storage class at the time of upload. A Lifecycle policy (C) would work, but it would be reactive (move it later) rather than enforcing it on upload.

</details>
    

---

**14. An IAM user has NO IAM policies attached that grant S3 access. However, the user is able to successfully upload a file to an S3 bucket. How is this possible?**

- A. The S3 bucket policy explicitly grants the `s3:PutObject` action to that specific user's ARN.
- B. The user is the account root user.
- C. The S3 bucket has public write access enabled.
- D. All of the above are possible reasons.
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** All three scenarios could explain this.

- A: For same-account access, an `Allow` in the bucket policy is sufficient, even if the user has no IAM permissions.
- B: The root user has implicit access to everything and is not bound by IAM policies.
- C: If the bucket is public, anyone can write to it, and no specific user permissions are needed.

Therefore, all are plausible explanations.

</details>
        

---

**15. Your organization (Account ID `111111111111`) wants to allow a partner organization (Account ID `999999999999`) to upload logs to your `partner-logs` S3 bucket. The partner uses an EC2 instance with an IAM role named `LogUploaderRole`. What is the most secure `Principal` to specify in your S3 bucket policy?**

- A. `"Principal": "*"`
- B. `"Principal": { "AWS": "arn:aws:iam::999999999999:root" }`
- C. `"Principal": { "AWS": "arn:aws:iam::999999999999:role/LogUploaderRole" }`
- D. `"Principal": { "AWS": "arn:aws:iam::111111111111:root" }`
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This follows the principle of least privilege. Specifying the full ARN of the IAM role (LogUploaderRole in the partner's account) is the most specific and secure option. It ensures that only that specific role can perform the action. Granting access to the entire root of the partner's account (B) is too broad, as it would allow any identity in their account to gain access if their own IAM policies permitted it.

</details>
    

---

**16. Which of the following is an example of an identity-based policy?**

- A. An S3 bucket policy.
- B. A policy attached to an IAM group.
- C. A VPC endpoint policy.
- D. A KMS key policy.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Identity-based policies are attached to IAM identities. Since a group is a collection of IAM users (an identity construct), a policy attached to it is identity-based. S3 bucket policies, VPC endpoint policies, and KMS key policies are all examples of resource-based policies because they are attached to the resource itself.

</details>
    

---

**17. You want to deny access to an S3 bucket if the request does not come from your corporate network IP range or from your VPC. Which policy type and condition keys would you use?**

- A. An IAM policy using the `aws:SourceVPC` condition key.
- B. An S3 bucket policy using the `aws:SourceIp` and `aws:SourceVpce` condition keys.
- C. An S3 ACL to specify the allowed IP addresses.
- D. An IAM permissions boundary using the `aws:SourceIp` condition key.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This requirement is best enforced at the resource level using an S3 bucket policy. The policy would contain a statement with "Effect": "Deny" and a Condition block that states the denial should apply if the IP address is NOT in your corporate range (aws:SourceIp) AND the request is NOT coming from your VPC endpoint (aws:SourceVpce).

</details>
    

---

**18. If a user needs access to a large number of S3 buckets and their permissions are the same for all of them, what is the most efficient way to manage these permissions?**

- A. Create a separate S3 bucket policy for each bucket.
- B. Create a single IAM policy that lists all the bucket ARNs in the `Resource` block and attach it to the user's IAM group.
- C. Create a single S3 bucket policy that lists all the buckets.
- D. Manually configure ACLs on each bucket.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** IAM policies are ideal for managing an identity's access across multiple resources. Creating a single IAM policy is much more efficient and less error-prone than managing hundreds of individual bucket policies (A). A single bucket policy cannot be attached to multiple buckets (C is not possible).

</details>
    

---

**19. An S3 bucket policy has a statement that allows a user to perform `s3:GetObject`. The same bucket policy has a second statement that denies the user `s3:GetObject` if their source IP is outside the corporate network. If the user tries to access the object from home, what happens?**

- A. Access is allowed because an `Allow` statement exists.
- B. Access is denied because the condition in the `Deny` statement is met.
- C. The policy is invalid because `Allow` and `Deny` conflict.
- D. The user will be prompted for MFA.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An explicit Deny always wins. When the user makes the request from home, their IP address does not match the corporate network range. This causes the condition in the Deny statement to be met, and the deny rule is applied, blocking the request, regardless of the Allow statement.

</details>
    

---

**20. A resource-based policy is attached to which of the following?**

- A. An IAM user.
- B. An IAM group.
- C. An Amazon S3 bucket.
- D. An IAM role.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Resource-based policies, by definition, are attached to resources. An S3 bucket is a resource. IAM users, groups, and roles are all identities, and they have identity-based policies attached to them.

</details>