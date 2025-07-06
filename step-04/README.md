# Step 04: MFA & IAM Best Practices

## 🎯 Objective
- [ ] Master: **MFA & IAM Best Practices**

## 📘 Notes

### **Deep Dive: MFA & IAM Best Practices**

Securing an AWS account is not just about assigning permissions; it's about building layers of security and following established best practices to reduce the risk of unauthorized access and human error. Multi-Factor Authentication (MFA) is a cornerstone of this, along with a set of guiding principles for managing identities and access.

### **1. Multi-Factor Authentication (MFA)** 📱🔒

MFA provides an extra layer of security on top of a username and password. With MFA enabled, when a user signs in to the AWS console, they will be prompted for their password (something they **know**) and an authentication code from their AWS MFA device (something they **have**). These two factors together provide much stronger security than a password alone.

- **How it Works:** MFA adds a second authentication factor, making it much harder for an attacker to gain access even if they manage to steal a user's password.
- **Types of MFA Devices:**
    - **Virtual MFA Devices:** These are software applications that run on a smartphone or computer and adhere to the time-based one-time password (TOTP) standard. Examples include Google Authenticator, Authy, and Microsoft Authenticator. This is the most common and flexible option.
    - **U2F Security Keys:** A Universal 2nd Factor (U2F) device is a physical key that plugs into a USB port. Users authenticate by simply tapping the device. These are resistant to phishing because they verify that you are logging into the legitimate AWS site. Examples include YubiKey.
    - **Hardware MFA Tokens:** A physical device that generates a six-digit code. It's like a virtual MFA app but on a dedicated piece of hardware.
- **MFA on the Root User:** **This is the single most important security measure you can take.** The first thing you should do in any new AWS account is enable MFA on the root user.
- **MFA for IAM Users:** It is a strong best practice to require MFA for all human IAM users, especially those with significant permissions. You can enforce this by creating an IAM policy that denies all actions unless the user has authenticated with MFA.
- **MFA-Protected API Access:** You can require MFA authentication to perform specific, sensitive API operations, such as terminating EC2 instances or deleting S3 buckets. This is achieved by using the `aws:MultiFactorAuthPresent` condition key in an IAM policy.

### **2. IAM Best Practices** ⭐

AWS outlines a clear set of best practices that every Solutions Architect should know and apply.

- **Lock Away Your Root User Access Keys:** The root user has unrestricted access to everything in your account. You should **never** use it for daily tasks. Enable MFA on it and then securely store the credentials. Do not create access keys for the root user.
- **Principle of Least Privilege:** This is the most fundamental concept. Grant only the permissions required to perform a task. For example, if an application only needs to read objects from an S3 bucket, its role should only have the `s3:GetObject` permission, not `s3:*`. Start with a minimum set of permissions and grant additional permissions as necessary.
- **Use IAM Roles, Not Long-Term Credentials:** As we've covered, roles are the preferred way to grant permissions. Use roles for applications running on EC2, for cross-account access, and for federated users. Avoid using long-term IAM user access keys whenever possible.
- **Leverage IAM Identity Center for Human Access:** Instead of creating individual IAM users for your employees, use AWS IAM Identity Center (formerly AWS SSO). It allows you to connect to your existing identity provider (like Azure AD or Okta) for Single Sign-On and manage permissions centrally with permission sets.
- **Rotate Credentials Regularly:** For the few cases where you must use long-term credentials (like IAM user access keys), they should be rotated on a regular basis (e.g., every 90 days). You can use services like AWS Secrets Manager to automate the rotation of database credentials.
- **Use Policy Conditions for Extra Security:** Use IAM policy condition keys to further restrict access. We've seen `aws:MultiFactorAuthPresent`. Other useful conditions include `aws:SourceIp` (to limit access to a specific IP address range) and `aws:RequestedRegion` (to enforce data residency).
- **Monitor and Audit Activity:** Use **AWS CloudTrail** to log all API calls made in your account. This provides a complete audit trail of who did what, when, and from where. You can use **Amazon CloudWatch Events** (now EventBridge) and **AWS Config** to create alerts for suspicious activity (e.g., a root user login) or non-compliant configurations.
- **Use a Password Policy:** Enforce a strong password policy for your IAM users (e.g., minimum length, character requirements, password expiration).

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company wants to enforce a rule that all IAM users must use Multi-Factor Authentication (MFA) when signing in. How can a Solutions Architect enforce this requirement?**

- A. Manually check each user's security settings every day.
- B. Configure a strong password policy in IAM.
- C. Attach an IAM policy to all user groups that includes a `Deny` statement for all actions if `aws:MultiFactorAuthPresent` is `false`.
- D. Enable MFA only on the root user account, as it protects all other users.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the standard method for enforcing MFA. By creating a policy that explicitly denies actions unless the `aws:MultiFactorAuthPresent` condition key evaluates to true, you effectively block non-MFA authenticated users from doing anything, forcing them to enable it. A password policy (B) is a good practice but does not enforce MFA.

</details>
    

---

**2. A new AWS account has just been created. According to AWS best practices, what is the most critical first step the account owner should take to secure the account?**

- A. Create an administrative IAM user for daily tasks.
- B. Enable Multi-Factor Authentication (MFA) on the root user.
- C. Create an S3 bucket to store access logs.
- D. Enable AWS CloudTrail in all regions.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** While creating an admin user (A) and enabling CloudTrail (D) are crucial next steps, the single most important first action is to secure the root user by enabling MFA. The root user has ultimate power over the account, and protecting it is the top priority.

</details>
    

---

**3. A developer requires permissions to terminate EC2 instances, but the security team wants this sensitive action to require an additional layer of security. How can this be configured?**

- A. Grant the developer's IAM user `AdministratorAccess`.
- B. Create an IAM policy that allows the `ec2:TerminateInstances` action only when the `aws:MultiFactorAuthPresent` condition is true.
- C. Send the developer a one-time password via email each time they need to terminate an instance.
- D. Place the developer's user in a group that has a permissions boundary attached.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a perfect use case for MFA-protected API access. By using the aws:MultiFactorAuthPresent condition in the policy that allows the ec2:TerminateInstances action, you ensure that the user must have an active MFA session to perform that specific sensitive operation. This provides granular, just-in-time security.

</details>
    

---

**4. A company's security policy requires that all IAM user access keys be rotated every 90 days. Which AWS services can help automate the monitoring of this policy? (Select TWO)**

- A. AWS Config
- B. Amazon S3
- C. IAM Access Analyzer
- D. AWS Organizations
- E. Amazon Inspector
<details>
<summary>View Answer</summary>

**Answers: A and C**

**Explanation:** AWS Config has managed rules (like iam-user-no-unused-credentials-check and rules to check for access key age) that can identify non-compliant resources. IAM Access Analyzer also generates findings for unused access keys, helping you identify keys that are candidates for rotation or deletion. AWS Organizations can enforce SCPs but doesn't directly monitor key age.

</details>
    

---

**5. Which IAM best practice helps prevent accidental or malicious deletion of critical resources by even highly privileged users?**

- A. Using IAM roles for EC2 instances.
- B. Enforcing a strong password policy.
- C. Enabling MFA on delete actions.
- D. Rotating access keys every 90 days.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Enabling MFA on specific, destructive actions (like s3:DeleteBucket or ec2:TerminateInstances) is a powerful safeguard. It means that even if an attacker compromises an admin's credentials, they cannot perform the most damaging actions without also having access to the MFA device.

</details>
    

---

**6. A company wants to provide its employees with Single Sign-On (SSO) access to multiple AWS accounts and business applications using their existing corporate credentials. Which AWS service is designed for this purpose?**

- A. AWS CloudTrail
- B. AWS IAM Identity Center (formerly AWS SSO)
- C. AWS Secrets Manager
- D. AWS Certificate Manager
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS IAM Identity Center is the recommended service for centralizing user access management. It integrates with identity providers like Azure AD or Okta to provide a seamless SSO experience for users, allowing them to access their assigned AWS accounts and applications without needing separate IAM users in each account.

</details>
    

---

**7. An IAM policy contains the following condition block: `"Condition": {"IpAddress": {"aws:SourceIp": "203.0.113.0/24"}}`. What is the effect of this condition?**

- A. It allows access only if the user has authenticated with MFA.
- B. It allows access only if the request originates from the specified IP address range.
- C. It denies access if the request comes from the specified IP address range.
- D. It allows access only if the request is targeting resources in the `us-east-1` region.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The aws:SourceIp condition key is used to restrict access based on the requestor's IP address. In this case, the policy will only apply (or allow the action, depending on the Effect) if the user's IP address falls within the 203.0.113.0/24 CIDR block, which is typically a corporate network.

</details>
    

---

**8. An auditor needs to review a log of all API activities performed by IAM users and roles in an AWS account over the past year for compliance purposes. Which service provides this information?**

- A. Amazon CloudWatch
- B. AWS Config
- C. AWS Trusted Advisor
- D. AWS CloudTrail
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** AWS CloudTrail is the definitive service for logging API activity. It captures a detailed record of every action taken in your account, including the identity of the caller, the time of the call, the source IP address, and the request parameters. This is essential for security auditing and compliance.

</details>
    

---

**9. What is the most significant security risk associated with creating and using long-term access keys for an IAM user?**

- A. They are difficult to generate.
- B. They expire automatically after 24 hours.
- C. If compromised, they provide long-term, persistent access to an attacker until manually revoked.
- D. They cannot be used with the AWS Management Console.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The primary danger of long-term credentials is their static nature. Unlike temporary credentials from a role, which expire quickly, a compromised access key remains valid indefinitely. This gives an attacker a persistent foothold in your account, which is a major security breach.

</details>
    

---

**10. A solutions architect is creating a "break-glass" IAM user for emergency access. Which of the following is the LEAST recommended practice for this user?**

- A. Granting the user administrative privileges.
- B. Attaching a strong MFA device to the user.
- C. Creating an access key and secret key and embedding them in a script stored on a bastion host.
- D. Heavily auditing all actions taken by this user with CloudTrail and setting up alerts.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A "break-glass" user should not have long-term access keys stored anywhere. This user should only be accessed via the console with a password and a very securely stored MFA device. Storing its keys on a server (even a bastion host) creates a massive security risk and defeats the purpose of a secure emergency user.

</details>
    

---

**11. Which IAM principle dictates that you should start with a minimal set of permissions and grant more only as necessary?**

- A. The principle of credential rotation.
- B. The principle of least privilege.
- C. The principle of centralized identity.
- D. The principle of MFA enforcement.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The principle of least privilege is a core security concept that involves granting only the permissions that are strictly necessary for a user or service to perform its intended function. This minimizes the "blast radius" if the credentials are ever compromised.

</details>
    

---

**12. To improve the security of an application running on Amazon EC2, a solutions architect has replaced an IAM user's access keys with an IAM role. What is the direct security benefit of this change?**

- A. The application no longer needs to store long-term, static credentials on the instance.
- B. The application will now run faster.
- C. The IAM role provides access to all AWS services by default.
- D. The EC2 instance can now be accessed without a key pair.
<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** The key benefit of using an IAM role for an EC2 instance is the elimination of hard-coded, long-term credentials. The instance automatically receives temporary, rotated credentials from the EC2 metadata service, which is a much more secure and manageable approach.

</details>
    

---

**13. Which of the following is a feature of a U2F security key as an MFA device?**

- A. It generates a time-based six-digit code on a screen.
- B. It is a software application installed on a smartphone.
- C. It helps protect against phishing by verifying the domain it is authenticating against.
- D. It sends a push notification to your phone for approval.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** U2F keys (a type of FIDO security key) provide strong anti-phishing protection. As part of the authentication ceremony, the key verifies that it is communicating with the original site it registered with (e.g., aws.amazon.com). If a user is on a phishing site, the key will not authenticate, protecting the user's credentials.

</details>
    

---

**14. A company policy dictates that all IAM user passwords must be at least 14 characters long and must be changed every 90 days. Where does a solutions architect configure this policy?**

- A. In AWS Organizations using an SCP.
- B. In the IAM account password policy settings.
- C. In a permissions boundary applied to each user.
- D. In AWS Secrets Manager.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** IAM provides a dedicated section for setting an account-wide password policy for all IAM users. This is where you can configure minimum length, character requirements, password expiration periods, and preventing password reuse.

</details>
    

---

**15. A company is using AWS Organizations. The security team wants to prevent any user in any member account, including the root user, from disabling AWS GuardDuty. What is the most effective tool for this?**

- A. An IAM permissions boundary.
- B. An IAM identity-based policy.
- C. An AWS WAF rule.
- D. A Service Control Policy (SCP).
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** A Service Control Policy (SCP) is the only tool listed that can apply a permission guardrail across an entire Organization or OU that affects all principals, including the root user of member accounts. Denying the guardduty:DisableOrganizationAdminAccount and related actions in an SCP is the correct way to enforce this centrally.

</details>
    

---

**16. What is the purpose of the `iam:PassRole` permission?**

- A. It allows a user to assume a role.
- B. It allows a user to create a new role.
- C. It allows a user to attach a role to an AWS service or resource, like an EC2 instance.
- D. It allows a user to edit the permissions of a role.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The iam:PassRole permission is a critical safeguard. It allows a principal (like a user) to pass a role to an AWS service. For example, to launch an EC2 instance with an instance profile (role), the user launching it must have iam:PassRole permission for that specific role. This prevents a user from attaching a role with higher privileges than they themselves possess.

</details>
    

---

**17. A security audit has flagged an IAM user who has not logged in or used their access keys in over 180 days. What is the recommended action?**

- A. Force the user to change their password.
- B. Attach an MFA device to the user.
- C. Delete the user's access keys and password to disable the user.
- D. Add the user to the Administrators group.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Inactive credentials pose a security risk. The best practice is to remove unused credentials to reduce the attack surface. Disabling the user by deleting their password and access keys is the most secure action. You can always re-enable them if needed.

</details>
    

---

**18. Which service provides a formal report on the extent to which your use of AWS aligns with security best practices and can check the MFA status of the root account?**

- A. AWS CloudTrail
- B. Amazon Macie
- C. AWS Trusted Advisor
- D. Amazon GuardDuty
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Trusted Advisor acts like an automated consultant, inspecting your AWS environment and making recommendations to align with best practices across cost optimization, performance, fault tolerance, and security. One of its primary security checks is verifying that MFA is enabled on the root account.

</details>
    

---

**19. You are creating a policy to enforce MFA for a group of users. The policy denies all actions when `aws:MultiFactorAuthPresent` is `false`. A user in this group complains they can't even enable MFA on their own account. What is the problem?**

- A. The user must contact AWS Support to enable MFA.
- B. The `Deny` policy is blocking the `iam:EnableMFADevice` action before they can set it up.
- C. The user needs to use a hardware MFA token, not a virtual one.
- D. The policy is working as intended; this is expected behavior.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic "catch-22" scenario. The broad Deny policy blocks all actions, including the very IAM actions needed to set up an MFA device. The policy must be modified to allow specific actions like iam:EnableMFADevice, iam:ListMFADevices, etc., even when MFA is not yet present.

</details>
    

---

**20. A solutions architect wants to grant an application running on-premises temporary access to upload files to an S3 bucket. Which is the MOST secure method?**

- A. Create an IAM user and embed its long-term access keys in the application's configuration.
- B. Create an IAM role with a trust policy for the on-premises application, and have the application use STS to assume the role.
- C. Make the S3 bucket public and have the application upload files without credentials.
- D. Create a pre-signed URL for the S3 bucket with an expiration of 10 years.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The most secure method for granting access to entities outside of AWS is to use identity federation. The on-premises application would first authenticate to an Identity Provider (IdP) and then use the token from the IdP to call the AWS STS AssumeRole (or AssumeRoleWithSAML/WebIdentity) API. This provides the application with short-lived, temporary credentials, avoiding the risk of storing long-term keys (A).

</details>
