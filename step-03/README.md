# Step 03: IAM – SCP & Permissions Boundaries

## 🎯 Objective
- [ ] Master: **IAM – SCP & Permissions Boundaries**

## 📘 Notes
### **Deep Dive: IAM – Service Control Policies (SCPs) & Permissions Boundaries**

In the realm of AWS Identity and Access Management (IAM), both Service Control Policies (SCPs) and Permissions Boundaries are crucial for establishing robust security guardrails. While they both restrict permissions, they operate at different levels and serve distinct purposes. Understanding their interplay is a key competency for a Solutions Architect.

### **1. Service Control Policies (SCPs)** 📜

SCPs are a feature of **AWS Organizations**, designed to provide centralized control over the maximum available permissions for all IAM principals (users and roles) within an organization or an Organizational Unit (OU).

- **How they work:** SCPs act as a filter. They do not grant any permissions themselves. Instead, they define a guardrail that specifies the maximum permissions an account can have. Even if an IAM user has `AdministratorAccess` (a policy granting `Allow *:*`), an SCP can restrict their actions. For an action to be allowed, it must be permitted by **both** the SCP and the IAM identity-based policy.
- **Key Characteristics:**
    - **Applied at the Organization Level:** You attach SCPs to the organization's root, an OU, or directly to an AWS account. They are inherited down the hierarchy.
    - **Never Grant Permissions:** An SCP's sole function is to set boundaries. If an action isn't allowed by an SCP, no IAM policy can grant it.
    - **Affects All Principals in an Account:** This includes the **root user** of the member account. This is a powerful feature for preventing even the highest level of access within a member account from performing certain actions.
    - **Doesn't Affect Service-Linked Roles:** SCPs do not restrict service-linked roles, which are necessary for many AWS services to function correctly.
    - **Default Policy:** By default, every new account and OU has the `FullAWSAccess` SCP attached, which does not restrict any action. To implement restrictions, you must detach this and attach custom SCPs.
- **Common Use Cases:**
    - **Enforcing Data Residency:** Creating an SCP that denies access to all AWS Regions except for specific, approved ones.
    - **Restricting Risky Services:** Preventing member accounts from using services that the organization has deemed too costly or high-risk (e.g., disabling the ability to create new IAM users or leave the organization).
    - **Mandating Security Configurations:** Enforcing that all new EBS volumes must be encrypted by denying the `ec2:CreateVolume` action if the `ec2:Encrypted` condition is false.

### **2. Permissions Boundaries** सीमा

A Permissions Boundary is an advanced IAM feature that sets the maximum permissions an IAM principal (a user or a role) can have. It is an IAM policy that is attached to a principal but, like an SCP, it does not grant permissions on its own.

- **How they work:** A permissions boundary acts as a ceiling on the permissions that can be granted by the principal's identity-based policies. The effective permissions of a principal are the **intersection** of what is allowed by its identity-based policies and its permissions boundary.
- **Key Characteristics:**
    - **Applied at the Principal Level:** You attach a permissions boundary to a specific IAM user or role.
    - **Delegation of Permissions:** A primary use case is to allow developers or other users to create new IAM roles and policies, but only within the confines of the permissions boundary you've set. For example, you can allow a user to create roles for Lambda functions, but prevent them from creating a role with full administrative privileges.
    - **Does Not Grant Permissions:** An action is only allowed if it is permitted by **both** the permissions boundary AND the identity-based policy.
    - **Not a replacement for Identity Policies:** A user or role must still have an identity-based policy attached that grants them the permissions to perform actions. The boundary only limits those permissions.
- **Common Use Cases:**
    - **Safe Delegation:** Allowing a senior developer to manage IAM roles for their project's EC2 instances, but preventing them from creating roles with access to sensitive services like S3 or DynamoDB in other departments.
    - **Enforcing Least Privilege for Applications:** Ensuring that an application's role can never have its permissions escalated beyond what is absolutely necessary for its function.

### **3. The Combined Evaluation Logic** 🚦

When a principal makes a request, AWS evaluates policies in a specific order. The final decision is a "Deny" if any of the policies explicitly deny the action. For an "Allow" decision, the request must be allowed at multiple levels:

1. **Organizational SCPs:** The action must be allowed by the SCPs. If the SCP denies the action, the evaluation stops, and the request is denied.
2. **Permissions Boundary:** (If present) The boundary must allow the action.
3. **Identity-Based Policies:** The user or role's own IAM policies must allow the action.
4. **Resource-Based Policies:** (If applicable) Must also allow the action.

**The Golden Rule:** For an action to be permitted, it must be allowed by the SCP, the Permissions Boundary (if present), and the Identity-Based Policy. An explicit `Deny` in any of these policies will always override an `Allow`.

---

### **AWS SAA-C03 Style Questions & Explanations**

1. A company uses AWS Organizations to manage multiple AWS accounts. The security team wants to prevent any user in a specific Organizational Unit (OU) from disabling AWS CloudTrail. What is the most effective way to achieve this?
    - A. Create an IAM permissions boundary that denies the `cloudtrail:StopLogging` action and attach it to all IAM users in the OU.
    - B. Implement an identity-based policy with a `Deny` statement for `cloudtrail:StopLogging` and attach it to all roles in the accounts within the OU.
    - C. Create a Service Control Policy (SCP) that denies the `cloudtrail:StopLogging` action and attach it to the OU.
    - D. Configure AWS Config to trigger an alert if CloudTrail is stopped and use a Lambda function to restart it.
    
    Answer: C
    
    Explanation: An SCP is the perfect tool for this. It acts as a centralized guardrail for an entire OU. By denying cloudtrail:StopLogging at the OU level, no principal (including the root user) in any account within that OU can perform the action, regardless of their individual IAM permissions. This is more scalable and foolproof than managing permissions boundaries (A) or identity policies (B) for every principal. Option D is a reactive measure, not a preventative one.
    

---

1. A development team needs to create IAM roles for their Amazon EC2 instances. The solutions architect wants to ensure that these developers can only create roles with permissions related to EC2 and S3, and nothing else. How can this be achieved while following the principle of least privilege?
    - A. Grant the developers `AdministratorAccess` and trust them to create appropriate roles.
    - B. Attach an SCP to the developers' account that only allows EC2 and S3 actions.
    - C. Create an IAM role for the developers that allows the `iam:CreateRole` action and has an attached permissions boundary that only permits `ec2:*` and `s3:*` actions.
    - D. Give the developers an IAM policy that allows them to pass any existing role to an EC2 instance.
    
    Answer: C
    
    Explanation: This is the primary use case for permissions boundaries: safe delegation. By attaching a permissions boundary, you allow the developers to create roles (iam:CreateRole), but you limit the maximum permissions those new roles can have. Any role they create cannot have permissions outside of what is defined in the boundary (ec2:* and s3:*). This prevents privilege escalation. Option B is incorrect because an SCP would restrict the developers themselves, not the roles they create.
    

---

1. An IAM user has an identity-based policy attached that grants full S3 access (`s3:*`). The user also has a permissions boundary attached that explicitly allows `s3:GetObject` but denies all other S3 actions. What is the effective permission for this user?
    - A. The user has full S3 access because the identity-based policy is less restrictive.
    - B. The user can only perform the `s3:GetObject` action.
    - C. The user has no S3 permissions because the policies conflict.
    - D. The permissions boundary is ignored because an identity-based policy is also present.
    
    Answer: B
    
    Explanation: The effective permissions are the intersection of the identity-based policy and the permissions boundary. The identity policy allows everything (s3:*), but the boundary only allows s3:GetObject. The intersection of "everything" and "s3:GetObject" is just "s3:GetObject".
    

---

1. A company is expanding into Europe and needs to ensure that no resources are provisioned outside of the `eu-west-1` and `eu-central-1` regions for data sovereignty reasons. Which of the following is the most appropriate and scalable solution?
    - A. Attach a permissions boundary to every IAM user and role in all accounts, denying access to other regions.
    - B. Create an SCP that denies all actions if the requested region is not `eu-west-1` or `eu-central-1`, and apply it to the root of the organization.
    - C. Use AWS Config rules to detect and remediate resources created in non-approved regions.
    - D. Manually audit the accounts on a weekly basis to check for non-compliant resources.
    
    Answer: B
    
    Explanation: Using an SCP applied to the organization's root is the most effective and encompassing solution. It creates a hard guardrail that is automatically inherited by all accounts. This prevents resources from being created in unapproved regions in the first place. Managing permissions boundaries (A) is not scalable. AWS Config (C) is reactive; it detects non-compliance after it happens, whereas an SCP prevents it.
    

---

1. Which of the following statements is TRUE regarding Service Control Policies (SCPs)?
    - A. SCPs can grant permissions to IAM users.
    - B. SCPs do not affect the root user of a member account.
    - C. By default, a new AWS account in an organization has an SCP attached that denies all actions.
    - D. An explicit `Deny` in an SCP will override any `Allow` in an IAM policy.
    
    Answer: D
    
    Explanation: An explicit Deny in any policy, especially a high-level one like an SCP, always takes precedence. SCPs never grant permissions (A is false), they do affect the root user of member accounts (B is false), and the default SCP is FullAWSAccess, which allows all actions (C is false).
    

---

1. A solutions architect has configured a permissions boundary for an IAM role. The permissions boundary allows `dynamodb:PutItem`. The identity-based policy for the same role allows `dynamodb:GetItem`. What actions can a user assuming this role perform?
    - A. `dynamodb:PutItem` only.
    - B. `dynamodb:GetItem` only.
    - C. Both `dynamodb:PutItem` and `dynamodb:GetItem`.
    - D. No actions, as the effective permission is the empty set where the policies intersect.
    
    Answer: D
    
    Explanation: The effective permission is the intersection. The set of actions in the identity policy is {GetItem} and the set of actions in the boundary is {PutItem}. Since there are no common actions between the two policies, the user can perform neither. For an action to be allowed, it must be permitted by BOTH policies.
    

---

1. An organization wants to delegate administrative tasks for a specific application to a team lead. The team lead should be able to create and manage IAM policies and roles for their application's EC2 instances, but they must be prevented from escalating their own privileges or creating roles with access to billing information. How should a solutions architect configure this?
    - A. Grant the team lead `PowerUserAccess`.
    - B. Create a custom IAM role for the team lead with an attached permissions boundary that limits the permissions they can grant to new roles.
    - C. Create an SCP that allows only EC2-related actions and attach it to the team lead's user.
    - D. Place the team lead's account in a separate OU with a restrictive SCP.
    
    Answer: B
    
    Explanation: This is another classic delegation scenario perfectly suited for a permissions boundary. The boundary acts as a "guardrail" on the iam:CreateRole and iam:AttachPolicy permissions, ensuring the team lead can only create roles/policies within the limits set by the architect, thus preventing privilege escalation. SCPs (C and D) apply to the entire account/OU, which is too broad for delegating permissions to a single user.
    

---

1. What is the primary purpose of an IAM permissions boundary?
    - A. To provide centralized, high-level permission guardrails across multiple AWS accounts.
    - B. To grant temporary credentials to IAM users.
    - C. To set the maximum permissions that an identity-based policy can grant to an IAM user or role.
    - D. To replace the need for identity-based policies.
    
    Answer: C
    
    Explanation: A permissions boundary acts as a ceiling. It defines the "maximum" scope of a principal's permissions. The actual permissions are then granted by identity-based policies, but they can never exceed what the boundary allows. Option A describes SCPs.
    

---

1. An AWS account is part of an organization. The OU it belongs to has an SCP attached that explicitly denies the `iam:CreateUser` action. The account's root user logs in and attempts to create a new IAM user. What will be the outcome?
    - A. The action will be successful because the root user's permissions cannot be restricted.
    - B. The action will fail because SCPs apply to all principals in a member account, including the root user.
    - C. The action will be successful only if the root user has MFA enabled.
    - D. The action will be successful because SCPs do not apply to the `iam:CreateUser` action.
    
    Answer: B
    
    Explanation: A key feature and a major reason to use SCPs is that they do apply to the root user of member accounts. This allows an organization to enforce critical guardrails that even the highest-privileged user in a member account cannot bypass.
    

---

1. A user is attempting to perform an action that is allowed by their IAM policy, their permissions boundary, and a resource-based policy. However, the action is failing with an access denied error. What is the most likely cause?
    - A. The user's IAM policy has a syntax error.
    - B. A Service Control Policy at the organizational level is denying the action.
    - C. The resource-based policy needs to be attached to the user as well.
    - D. The permissions boundary is overriding the IAM policy.
    
    Answer: B
    
    Explanation: The evaluation logic checks SCPs first. If the action is explicitly allowed at the user/role level (by the IAM policy and permissions boundary) and the resource level, the only remaining place for a denial in this context is a higher-level SCP.
    

---

1. Which two of the following are valid use cases for a Service Control Policy (SCP)? (Select TWO)
    - A. To grant read-only access to a specific S3 bucket.
    - B. To restrict access to AWS services for a single IAM user.
    - C. To enforce that all new S3 buckets must have encryption enabled.
    - D. To prevent member accounts from leaving the AWS Organization.
    - E. To define the maximum permissions for a single IAM role.
    
    Answers: C and D
    
    Explanation: SCPs are for broad, preventative guardrails. You can deny the s3:PutBucket action if the encryption header is not present (C). You can also deny the organizations:LeaveOrganization action to prevent member accounts from detaching themselves (D). SCPs cannot grant permissions (A), and while they restrict users, they apply to whole accounts/OUs, not individuals (B). Defining max permissions for a single role is the job of a permissions boundary (E).
    

---

1. The policy evaluation logic for an IAM principal with a permissions boundary is as follows:
    - A. The permissions are the union of the identity policy and the permissions boundary.
    - B. The permissions are determined only by the permissions boundary.
    - C. The effective permissions are the intersection of the identity policy and the permissions boundary.
    - D. The identity policy is ignored if a permissions boundary is present.
    
    Answer: C
    
    Explanation: This is the core concept of permissions boundaries. An action must be allowed by BOTH the identity policy and the boundary to be permitted. The final set of allowed actions is the intersection (the common ground) between the two.
    

---

1. A company wants to ensure that developers can only launch T2 and T3 family EC2 instances to control costs. How can this be enforced across an entire development OU in AWS Organizations?
    - A. Create an IAM policy allowing only `t2.*` and `t3.*` instance types and attach it to a developers group.
    - B. Create an SCP with a `Deny` statement for the `ec2:RunInstances` action if the instance type is not `t2.*` or `t3.*`, and attach it to the development OU.
    - C. Use a permissions boundary to limit the instance types developers can launch.
    - D. Implement a Lambda function that terminates any non-compliant instance types after they are launched.
    
    Answer: B
    
    Explanation: To enforce a rule across an entire OU, an SCP is the correct tool. By using a Deny rule with a condition key (ec2:InstanceType), you can prevent the launching of any non-approved instance types before they are even created. Option A can be bypassed if users can create new policies. Option D is reactive.
    

---

1. A user reports they cannot perform an action that is explicitly allowed in their attached IAM policy. A permissions boundary is NOT attached to the user. What is the first thing a solutions architect should check?
    - A. The user's group policies.
    - B. The Service Control Policies (SCPs) applied to the user's account.
    - C. The trust policy of the user's role.
    - D. Network ACLs and Security Groups.
    
    Answer: B
    
    Explanation: When troubleshooting permissions, you must think about the entire evaluation chain. If the user's identity-based policies (user and group policies) and permissions boundary (which is absent here) allow an action, the next logical place to look for a restriction is the account-level guardrail: the SCP.
    

---

1. What is a key difference between an SCP and a permissions boundary?
    - A. SCPs can grant permissions, while permissions boundaries cannot.
    - B. Permissions boundaries apply to OUs, while SCPs apply to IAM users.
    - C. SCPs are managed within AWS Organizations, while permissions boundaries are a feature of IAM.
    - D. Permissions boundaries affect the root user of an account, while SCPs do not.
    
    Answer: C
    
    Explanation: This is a fundamental distinction. SCPs are a component of AWS Organizations and are used for multi-account governance. Permissions boundaries are a feature within the IAM service itself and are applied to individual IAM principals (users and roles). Both cannot grant permissions (A is false). Their application targets are swapped in option B. SCPs affect the root user, while boundaries do not (D is false).
    

---

1. An IAM Role has an identity policy granting `s3:*` and `ec2:*`. A permissions boundary is attached that allows `s3:GetObject` and `ec2:DescribeInstances`. What are the effective permissions of this role?
    - A. `s3:*` and `ec2:*`
    - B. `s3:GetObject` and `ec2:DescribeInstances`
    - C. All S3 and EC2 actions will be denied due to a conflict.
    - D. `s3:GetObject`, `ec2:DescribeInstances`, and `ec2:*`
    
    Answer: B
    
    Explanation: The effective permissions are the intersection of the two policies. The identity policy allows all S3 and EC2 actions. The boundary allows only s3:GetObject and ec2:DescribeInstances. The common ground (intersection) between these two sets is just s3:GetObject and ec2:DescribeInstances.
    

---

1. Your company has a policy that IAM users should not be created with long-term access keys. How can you enforce this across your entire AWS Organization?
    - A. Create a permissions boundary for every user that denies `iam:CreateAccessKey`.
    - B. Use an SCP to deny the `iam:CreateAccessKey` action and apply it to the root of the organization.
    - C. Educate all developers on best practices and periodically run a script to find and delete access keys.
    - D. Use an IAM policy with an explicit deny for `iam:CreateAccessKey`.
    
    Answer: B
    
    Explanation: To enforce a policy across an entire organization, an SCP is the most powerful and appropriate tool. Applying it at the root ensures all OUs and accounts inherit the policy, effectively preventing the iam:CreateAccessKey action everywhere. Options A and D are not scalable and can be bypassed. Option C is reactive and not preventative.
    

---

1. To which of the following can a permissions boundary be attached? (Select TWO)
    - A. An IAM Group
    - B. An IAM User
    - C. An Organizational Unit (OU)
    - D. An IAM Role
    - E. A Service Control Policy (SCP)
    
    Answers: B and D
    
    Explanation: A permissions boundary is a feature designed to set limits on a specific IAM principal. The two types of IAM principals are Users and Roles. You cannot attach a boundary to a group, OU, or another policy.
    

---

1. A solutions architect needs to grant a junior administrator the ability to manage IAM users, but not the ability to assign them to the "Administrators" group. Which feature would be most suitable for this?
    - A. Service Control Policy (SCP)
    - B. Permissions Boundary
    - C. Resource-based policy
    - D. Access Control List (ACL)
    
    Answer: B
    
    Explanation: This is a perfect use case for a permissions boundary. You can grant the junior admin permissions like iam:CreateUser, iam:PutUserPolicy, and iam:AddUserToGroup. Then, you attach a permissions boundary that denies the iam:AddUserToGroup action when the resource (the group) is the "Administrators" group. This allows them to perform their duties while preventing them from escalating privileges.
    

---

1. An organization's SCP allows `ec2:*` and `s3:*`. A user in a member account has an IAM policy allowing `ec2:DescribeInstances` and a permissions boundary allowing `s3:ListBucket`. What can the user do?
    - A. Only `ec2:DescribeInstances`
    - B. Only `s3:ListBucket`
    - C. Both actions.
    - D. Neither action.
    
    Answer: D
    
    Explanation: For any action to be allowed, it must pass through all checks.
    
    - For `ec2:DescribeInstances`: The SCP allows it, the IAM policy allows it, but the permissions boundary does *not* allow it. So, it's denied.
    - For s3:ListBucket: The SCP allows it, the permissions boundary allows it, but the IAM policy does not allow it. So, it's denied.
        
        The user can perform neither action because in each case, one of the required policies does not grant permission.