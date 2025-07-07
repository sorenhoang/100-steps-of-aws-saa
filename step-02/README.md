# Step 02: IAM – Roles vs Users vs Federated Access

## 🎯 Objective

- [x] Master: **IAM – Roles vs Users vs Federated Access**

## 📘 Notes

### **Deep Dive: IAM – Roles vs. Users vs. Federated Access**

Understanding how to grant access to your AWS resources is fundamental to security and architecture. AWS provides three primary mechanisms for this: IAM Users, IAM Roles, and Federated Access. While they can all be used to interact with AWS APIs, their use cases, security implications, and best practices are distinct.

### **1. IAM Users** 👤

An IAM User represents a person or an application that interacts with AWS. It is a permanent identity with a unique name and a set of long-term credentials.

- **Credentials:** An IAM User can have two types of long-term credentials:
  - **Password:** For access to the AWS Management Console.
  - **Access Keys (Access Key ID and Secret Access Key):** For programmatic access via the AWS Command Line Interface (CLI), SDKs, or direct API calls.
- **How it works:** Permissions are granted by attaching one or more IAM policies directly to the user or to a group the user belongs to. The user then uses their permanent credentials to authenticate and perform actions.
- **Key Characteristics:**
  - **Long-Lived Credentials:** Access keys for an IAM user are static and do not expire unless manually rotated. This makes them a significant security risk if compromised.
  - **Tied to One AWS Account:** An IAM User is created within a single AWS account and is meant for accessing resources within that account.
  - **Best Practice:** The AWS root user should not be used for daily tasks. The first step after creating an account is to create an administrative IAM user. For human access, it's now recommended to use IAM Identity Center (formerly AWS SSO) over creating individual IAM users.
- **When to Use (and When Not To):**
  - **Use:** For a small number of individuals who need permanent access and where setting up federation or IAM Identity Center is not feasible. Also used for service accounts for applications running _outside_ of AWS that need long-term credentials (though using roles where possible is still preferred).
  - **Avoid:** For applications running on AWS services (like EC2, Lambda), for granting access to users from other AWS accounts, or for providing access to human users who can be authenticated through an external identity provider.

### **2. IAM Roles** 🎭

An IAM Role is an identity that you can assume to gain a specific set of permissions. Unlike a user, a role does not have its own long-term credentials. When a principal (a user, application, or AWS service) assumes a role, it receives temporary security credentials.

- **Credentials:** Roles provide **temporary credentials** (an Access Key ID, a Secret Access Key, and a Session Token) that are valid for a predefined, limited duration (e.g., 15 minutes to 12 hours).
- **How it works:** A role has two key policies:
  - **Permissions Policy:** An identity-based policy that defines what the role is _allowed_ to do.
  - **Trust Policy (Assume Role Policy):** This is the most important part. It defines _who_ is allowed to assume the role (the "Principal"). This could be an AWS service (like `ec2.amazonaws.com`), another AWS account, an IAM user, or a federated user.
- **Key Characteristics:**
  - **Temporary Credentials:** Automatically rotated and short-lived, which significantly improves the security posture by reducing the risk of compromised keys.
  - **No Permanent Identity:** A role is meant to be assumed, not logged into directly.
  - **Universal and Flexible:** Roles are the preferred mechanism for granting permissions in almost all scenarios.
- **Common Use Cases:**
  - **AWS Service Access:** Allowing an EC2 instance to access an S3 bucket. The EC2 service assumes the role on behalf of the instance.
  - **Cross-Account Access:** Allowing users or resources from Account A (a trusted account) to access resources in Account B (the trusting account).
  - **Federation:** Users from a corporate directory (like Active Directory) or a web identity provider (like Google, Facebook) can assume a role after they authenticate.
  - **Delegating Permissions:** Granting an IAM user temporary elevated privileges to perform a specific administrative task.

### **3. Federated Access (Identity Federation)** 🤝

Federation is a method of linking a user's external identity with an AWS IAM Role. The user authenticates with an external Identity Provider (IdP), and the IdP then grants them a security assertion (like a SAML 2.0 token or OpenID Connect token), which can be exchanged for temporary AWS credentials via the AWS Security Token Service (STS).

- **How it works:**
  1. A user attempts to access an application that needs AWS resources.
  2. The application redirects the user to their Identity Provider (e.g., Okta, Azure AD, Google, Facebook).
  3. The user authenticates with the IdP using their corporate or social login credentials.
  4. The IdP sends a signed assertion back to the application.
  5. The application uses the AWS STS `AssumeRoleWithSAML` or `AssumeRoleWithWebIdentity` API call, passing the assertion to AWS.
  6. STS verifies the assertion from the IdP, checks that the IdP is trusted by the requested IAM Role, and then returns temporary credentials.
  7. The user can now use these temporary credentials to access AWS resources as allowed by the role.
- **Key Services:**
  - **AWS IAM Identity Center (formerly AWS SSO):** The preferred, managed AWS service for federating with corporate identity providers like Azure AD or Okta. It simplifies managing user access to multiple AWS accounts and applications.
  - **SAML 2.0 Federation:** Typically used for enterprise identity federation (e.g., connecting Active Directory via ADFS or another SAML-compliant IdP).
  - **OpenID Connect (OIDC) Federation:** Often used for web/mobile applications to federate with social identity providers like Google, Amazon, or Facebook.
- **Benefits:**
  - **No IAM Users Needed:** You don't need to create and manage IAM users for your human users. Their access is managed in your existing identity system.
  - **Single Sign-On (SSO):** Users log in once via their familiar IdP and gain access to multiple applications, including AWS.
  - **Improved Security:** No long-term AWS credentials need to be distributed to users. Access is based on short-lived tokens.

---

### **AWS SAA-C03 Style Questions & Explanations**

1. A company runs a web application on an Amazon EC2 instance that needs to read and write files to an Amazon S3 bucket. What is the most secure and recommended way to grant the application access to the S3 bucket?

   - A. Create an IAM user, generate an access key and secret key, and embed them in the application's configuration files on the EC2 instance.
   - B. Create an IAM role with the necessary S3 permissions and attach it to the EC2 instance profile.
   - C. Place the EC2 instance and the S3 bucket in the same Availability Zone to allow them to communicate.
   - D. Configure the S3 bucket policy to allow public read/write access so the EC2 instance can access it.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Using an IAM role is the standard, most secure method for granting permissions to AWS services like EC2. The role provides temporary, automatically rotated credentials, eliminating the high risk associated with storing long-term access keys (A) on the instance. Public access (D) is extremely insecure, and being in the same AZ (C) does not grant permissions.

</details>

---

2. A large enterprise wants to give its employees access to the AWS Management Console. The company already manages all employee identities in Microsoft Active Directory. They want to avoid creating and managing a separate set of credentials in AWS for each employee. Which solution should a solutions architect recommend?

   - A. Create an IAM user for each employee and teach them how to manage their passwords and access keys.
   - B. Set up federation between the company's Active Directory and AWS using IAM Identity Center (AWS SSO).
   - C. Export all users from Active Directory and run a script to create corresponding IAM users in AWS.
   - D. Create a single powerful IAM user and share the credentials among all employees who need access.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic use case for federation. IAM Identity Center (or setting up a SAML 2.0 trust directly) allows users to authenticate with their existing Active Directory credentials (Single Sign-On) and assume a role in AWS. This avoids managing duplicate identities (A, C) and is far more secure and scalable than sharing credentials (D).

</details>

---

3. Which of the following are characteristics of an IAM Role? (Select TWO)

   - A. It has long-term credentials in the form of an access key and secret key.
   - B. It has an attached Trust Policy that defines which principals can assume it.
   - C. It is primarily used to grant permanent permissions to a person.
   - D. It provides temporary security credentials upon assumption.
   - E. It can only be assumed by AWS services.
<details>
<summary>View Answer</summary>

**Answers: B and D**

**Explanation:** An IAM Role's defining features are its Trust Policy (who can assume it) and the fact that it provides temporary credentials (access key, secret key, and session token) when assumed. Roles do not have long-term credentials (A), and they are not for permanent access (C). They can be assumed by many types of principals, not just AWS services (E).

</details>

---

4. A mobile application allows users to sign in with their Facebook accounts. Once authenticated, the application needs to upload user-specific files to a private folder within an S3 bucket. Which AWS service and feature should be used to accomplish this securely?

   - A. Create an IAM user with S3 permissions and store the credentials in the mobile app.
   - B. Use Amazon Cognito with Identity Pools to federate with Facebook and grant temporary role-based access to S3.
   - C. Generate pre-signed URLs for the S3 bucket on a backend server, but use a shared IAM user to generate them.
   - D. Configure the S3 bucket for public access and manage security within the mobile application's code.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Amazon Cognito is designed specifically for this scenario (web/mobile identity federation). It integrates with social providers like Facebook (OIDC federation), manages the user pool, and provides temporary credentials by allowing authenticated users to assume an IAM role. This is far more secure than embedding IAM user credentials (A), which is a major security anti-pattern.

</details>

---

5. A solutions architect needs to grant a user from Account A (111122223333) permissions to read objects from an S3 bucket in Account B (999988887777). What is the most appropriate way to configure this access?

   - A. Create an IAM user in Account B and provide the access keys to the user in Account A.
   - B. In Account B, create an IAM Role with S3 read permissions and a trust policy that specifies Account A as the principal. The user in Account A then assumes this role.
   - C. Make the S3 bucket in Account B public and tell the user in Account A the bucket's name.
   - D. In Account A, create a role that trusts Account B to access its resources.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the standard pattern for cross-account access. The trusting account (B) creates a role that defines the permissions. Its trust policy explicitly names the trusted account (A). A user in Account A can then make an STS AssumeRole call to get temporary credentials for accessing the resources in Account B. Creating an IAM user (A) is less secure and harder to manage.

</details>

---

6. When should you prefer using an IAM User over an IAM Role?

   - A. When providing an EC2 instance access to other AWS services.
   - B. When granting access to a human user who authenticates via a corporate directory.
   - C. When an application running on-premises (outside of AWS) requires long-term, static credentials to access AWS APIs.
   - D. When providing temporary, elevated permissions for an administrative task.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While using roles with temporary credentials is still ideal if possible (e.g., via OIDC), one of the few remaining valid use cases for a long-term IAM user is for an application or service running outside the AWS ecosystem that cannot use a role-based workflow. All other options are classic examples where an IAM Role is the superior choice: for AWS services (A), federated users (B), and temporary privilege escalation (D).

</details>

---

7. What is the function of the "Trust Policy" in an IAM Role?

   - A. It specifies the API actions that the role is allowed to perform.
   - B. It defines the IAM users or groups that can use the role.
   - C. It lists the principals (users, services, accounts) that are permitted to assume the role.
   - D. It encrypts the temporary credentials generated by the role.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Trust Policy (or Assume Role Policy) is exclusively for defining "who" can assume the role. The permissions of the role—what it can do after being assumed—are defined in a separate Permissions Policy (an identity-based policy).

</details>

---

8. An organization uses IAM Identity Center (AWS SSO) integrated with their central identity provider. When a user authenticates, how do they gain access to AWS resources?

   - A. IAM Identity Center creates a permanent IAM user for them in the target account.
   - B. The user receives the access key and secret key of the account's root user.
   - C. The user is assigned a permission set, which maps to an IAM Role in the target account that they temporarily assume.
   - D. The user's credentials from the identity provider are used to call AWS APIs directly.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** IAM Identity Center works by mapping users/groups from the IdP to "Permission Sets". When a user is assigned a Permission Set for a specific account, Identity Center automatically creates a corresponding IAM Role in that target account. The user then assumes this role to get temporary credentials. This abstracts away the complexity of managing cross-account roles manually.

</details>

---

9. Which AWS service is used to request temporary security credentials when using IAM Roles or Federation?

   - A. AWS Key Management Service (KMS)
   - B. AWS Security Token Service (STS)
   - C. AWS Organizations
   - D. Amazon Cognito
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS STS is the core service responsible for vending temporary credentials. API calls like AssumeRole, AssumeRoleWithSAML, and GetFederationToken are all part of the STS service. While Cognito (D) uses STS under the hood for federation, STS is the fundamental service that provides the tokens.

</details>

---

10. What is a key security advantage of using IAM Roles instead of IAM Users with long-term keys?

   - A. IAM Roles are less expensive to manage.
   - B. IAM Roles automatically rotate credentials, minimizing the risk of a compromised key.
   - C. IAM Roles have more permissions by default.
   - D. IAM Roles do not require multi-factor authentication (MFA).
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The primary security benefit is the elimination of static, long-lived credentials. By providing short-term, automatically rotated credentials, IAM Roles drastically reduce the security risk. If temporary credentials leak, they expire quickly, limiting the window of exposure for an attacker.

</details>

---

11. A company has a fleet of EC2 instances that process data from a Kinesis Data Stream. What is the AWS best practice for granting these instances the necessary permissions to read from the stream?

   - A. Create one IAM user with Kinesis read permissions and securely distribute its access keys to all instances.
   - B. Create an IAM role with a permissions policy allowing Kinesis read access and a trust policy allowing the EC2 service to assume it. Attach this role to the instances.
   - C. Create an IAM user for each EC2 instance and attach a policy with Kinesis read permissions to each user.
   - D. Configure the Kinesis Data Stream to allow anonymous read access.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the canonical use case for an IAM Role for an AWS service. The role provides temporary, automatically-rotated credentials to the EC2 instances without the need to manage or distribute static access keys. This is significantly more secure and scalable than managing IAM users for services (A, C). Anonymous access (D) is insecure and not a recommended practice.

</details>

---

12. A user authenticates to AWS via their company's SAML 2.0-compliant identity provider (IdP). Which AWS API call is used to exchange the SAML assertion from the IdP for temporary AWS security credentials?

   - A. `GetFederationToken`
   - B. `AssumeRole`
   - C. `AssumeRoleWithSAML`
   - D. `GetSessionToken`
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The AssumeRoleWithSAML API call is specifically designed for this purpose. It takes the SAML assertion, the ARN of the role to be assumed, and other parameters, and returns temporary credentials from STS. AssumeRole is used for role assumption within AWS (e.g., cross-account or by an IAM user). GetFederationToken and GetSessionToken are older and less common federation methods.

</details>

---

13. Which of the following is NOT a valid principal in an IAM Role's Trust Policy?

   - A. An AWS Service (e.g., `ec2.amazonaws.com`).
   - B. Another AWS Account ARN (e.g., `arn:aws:iam::123456789012:root`).
   - C. An IAM Group.
   - D. A Federated User (via an OIDC provider ARN).
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** An IAM Group is simply a collection of users; it is not an identity that can be authenticated or assume a role. Therefore, you cannot specify an IAM Group as a principal in a trust policy. You can, however, specify individual users within that group, an entire AWS account, an AWS service, or a federated identity provider.

</details>

---

14. A company wants to grant its data science team, who are all in the IAM Group `DataScientists`, access to a specific S3 bucket in another AWS account (the "Data" account). What is the most secure and efficient way to grant this access?

   - A. In the Data account, create an IAM Role with S3 access. In the trust policy, list every individual IAM user ARN from the `DataScientists` group.
   - B. In the Data account, create an IAM user, generate access keys, and share them with the entire data science team.
   - C. In the Data account, create an IAM Role with S3 access and a trust policy that specifies the company's main account ARN as the principal. In the main account, attach a policy to the `DataScientists` group that allows them to assume that specific role.
   - D. In the Data account, modify the S3 bucket policy to allow public access, and then use IAM policies in the main account to restrict access to the `DataScientists` group.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This solution correctly uses the cross-account role pattern. The Data account trusts the main account. The main account then delegates the permission to assume that role to the DataScientists group. This is efficient because you don't need to update the trust policy every time a user is added to or removed from the group (unlike A). It is also far more secure than sharing keys (B) or using public access (D).

</details>

---

15. What is the primary difference between an IAM User and a Federated User?

   - A. An IAM User has permanent credentials managed in AWS, while a Federated User's identity is managed by an external IdP and they use temporary credentials in AWS.
   - B. An IAM User can only access the AWS Console, while a Federated User can only access the AWS CLI.
   - C. An IAM User is for humans, and a Federated User is for applications.
   - D. An IAM User's permissions are defined by SCPs, while a Federated User's permissions are defined by permissions boundaries.
<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** This is the core distinction. An IAM User is an identity that exists within AWS and has long-term credentials. A Federated User's identity exists outside of AWS (e.g., in Active Directory or Google). They authenticate externally and then assume a role in AWS to get temporary credentials.

</details>

---

16. An application running on an on-premises server needs to upload daily backups to Amazon S3. The security team has mandated that no long-term AWS credentials should be stored on the server. How can the on-premises application get temporary AWS credentials?

   - A. This is not possible; on-premises applications must use long-term IAM user credentials.
   - B. Install AWS CLI on the server and run `aws configure` with user credentials. The CLI will manage temporary credentials.
   - C. Use AWS STS `GetFederationToken` API to get temporary credentials for a federated user.
   - D. Establish a federation trust using an identity provider (like ADFS with SAML or an OIDC provider) that the on-premises application can authenticate against to assume a role.
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** To avoid long-term credentials on-premises, you must use federation. The application can be configured as a client in an IdP. After authenticating, it receives a token that it can use with the AssumeRoleWithSAML or AssumeRoleWithWebIdentity STS calls to get temporary AWS credentials. This is the modern, secure way to handle this scenario. GetFederationToken (C) is an older method and less common today.

</details>

---

17. A solutions architect is designing a system where an AWS Lambda function needs to write logs to Amazon CloudWatch Logs. What should the architect do?

   - A. Create an IAM user with CloudWatch permissions and store the access keys as environment variables in the Lambda function.
   - B. Assign an IAM Execution Role to the Lambda function that grants the necessary permissions to write to CloudWatch Logs.
   - C. All Lambda functions have default access to CloudWatch Logs, so no configuration is needed.
   - D. Configure the CloudWatch Logs log group policy to trust the Lambda service principal.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Every Lambda function has an Execution Role. This is an IAM Role that the Lambda service assumes when it runs your function. To grant permissions, you must attach the appropriate policies to this role. This is the standard, secure, and built-in mechanism. Storing keys as environment variables (A) is a security anti-pattern. Lambda functions do not have default permissions (C).

</details>

---

18. What information is included in the temporary credentials provided by AWS STS? (Select THREE)

   - A. An Access Key ID
   - B. A Secret Access Key
   - C. A Password
   - D. A Session Token
   - E. A User Name
<details>
<summary>View Answer</summary>

**Answers: A, B, and D**

**Explanation:** Temporary credentials consist of three parts: an access key, a secret key, and a session token. The session token is the key differentiator from long-term credentials. Any API call made with temporary credentials must include all three pieces to be validated by AWS.

</details>

---

19. A company has two AWS accounts: `Dev` and `Prod`. A developer with an IAM user in the `Dev` account needs to assume a role named `DeployRole` in the `Prod` account. What is the correct ARN to use as the Principal in the `DeployRole`'s trust policy?

   - A. The ARN of the developer's IAM user in the `Dev` account.
   - B. The ARN of the `Dev` account itself.
   - C. The ARN of the `DeployRole` in the `Prod` account.
   - D. The ARN of the `Prod` account.
<details>
<summary>View Answer</summary>

**Answer: A or B**

**Explanation:** Both A and B are valid and common patterns. You can specify the full ARN of the IAM user (e.g., arn:aws:iam::Dev-Account-ID:user/DeveloperName) for maximum specificity. This means only that user can assume the role. Alternatively, you can specify the root of the Dev account (e.g., arn:aws:iam::Dev-Account-ID:root). This trusts the entire Dev account, which then uses its own IAM policies to control which of its users can assume the role. For the SAA-C03 exam, trusting the account (B) is a very common pattern.

</details>

---

20. A mobile game uses a "Login with Amazon" button for authentication. Which federation technology is being used?

   - A. SAML 2.0
   - B. OpenID Connect (OIDC)
   - C. Kerberos
   - D. LDAP
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Social identity providers like Amazon, Google, and Facebook use the OpenID Connect (OIDC) standard for federation. SAML 2.0 is typically used for enterprise/corporate identity federation (e.g., Active Directory). Kerberos and LDAP are authentication protocols but are not used for this type of web identity federation with AWS.

</details>
