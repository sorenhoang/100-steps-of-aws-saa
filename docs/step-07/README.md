# Step 07: Landing Zone & Multi‑Account Governance

## 🎯 Objective

- [x] Master: **Landing Zone & Multi‑Account Governance**

## 📘 Notes

> Knowledge summary, documentation links, videos...

### **Deep Dive: Landing Zone & Multi-Account Governance**

As an organization's use of AWS expands, simply having multiple accounts isn't enough. You need a consistent, secure, and scalable foundation. This is where the concepts of a Landing Zone and robust multi-account governance become critical. They provide the blueprint for building and managing a secure and efficient multi-account environment at scale.

### **1. What is an AWS Landing Zone?** ✈️

An AWS Landing Zone is a **well-architected, multi-account AWS environment that is secure, scalable, and built on best practices**. It's not a single service, but rather a methodology and a starting point from which an organization can quickly launch and deploy workloads with confidence. It pre-configures your environment with a baseline for security, governance, and networking.

Think of it like building a new housing development. Instead of giving each homeowner an empty plot of land, you first lay down the foundational infrastructure: roads, plumbing, electricity, and zoning rules. A Landing Zone does the same for your cloud environment.

### **2. AWS Control Tower: Automating the Landing Zone** 🗼

Manually setting up a best-practice Landing Zone can be complex, involving the configuration of dozens of services. **AWS Control Tower** is the managed service that automates this process. It provides the easiest way to set up and govern a new, secure, multi-account AWS environment based on AWS best practices.

- **How Control Tower Works:**
  1. **Orchestration:** It uses other powerful AWS services like AWS Organizations, IAM Identity Center (SSO), AWS Config, and CloudTrail under the hood.
  2. **Sets up Core Accounts:** When you launch Control Tower, it automatically creates a foundational multi-account structure within your AWS Organization:
     - **Management Account:** The existing account used to set up Control Tower. Handles billing and central governance.
     - **Log Archive Account:** A dedicated, highly restricted account that acts as a central repository for all immutable CloudTrail (audit) and AWS Config (compliance) logs from all accounts in the organization.
     - **Audit Account (or Security Tooling Account):** A restricted account designed for your security and compliance teams. It gives them cross-account read/write access to all other accounts so they can run security tools, conduct audits, and respond to incidents without needing to log into each account individually.
  3. **Applies Guardrails:** It deploys a set of pre-configured governance rules called **Guardrails**.
  4. **Provides an Account Factory:** A standardized, automated way to provision new AWS accounts that conform to your policies.
- **Key Features of Control Tower:**
  - **Landing Zone:** Deploys the well-architected multi-account environment.
  - **Guardrails:** These are high-level rules that provide ongoing governance. They are expressed in plain English but are implemented using SCPs, AWS Config rules, and Lambda functions.
    - **Preventive Guardrails:** Enforced using **SCPs**. They prevent actions from happening (e.g., "Disallow changes to CloudTrail configuration").
    - **Detective Guardrails:** Enforced using **AWS Config rules**. They detect non-compliant resources after they are created and report them on the Control Tower dashboard (e.g., "Detect whether public read access to S3 buckets is allowed").
  - **Account Factory:** A feature, often built using AWS Service Catalog, that automates the creation of new AWS accounts. When a user "orders" a new account from the factory, it is provisioned with a standard baseline, such as pre-configured VPC networking and necessary Guardrails, ensuring consistency.
  - **Dashboard:** Provides a centralized dashboard to view the compliance status of your OUs, accounts, and enabled Guardrails.

### **3. The Pillars of Multi-Account Governance**

A well-governed environment, as established by a Landing Zone, is built on several key pillars:

- **Centralized Identity and Access Management:** Use **AWS IAM Identity Center (SSO)** as the single entry point for all human users. This allows you to manage users and groups in one place and assign them permissions to assume roles in various member accounts, avoiding the insecure practice of creating IAM users in every account.
- **Centralized Logging:** As implemented by Control Tower, all audit and compliance logs (CloudTrail and Config) from all accounts are shipped to a single, secure S3 bucket in the **Log Archive account**. This ensures logs are immutable and can be analyzed from a central location.
- **Preventive and Detective Controls (Guardrails):** Use a combination of SCPs (to prevent bad things from happening) and AWS Config rules (to detect non-compliance) to enforce your organization's policies across all accounts.
- **Automated Account Provisioning:** Use the **Account Factory** to ensure that new accounts are created consistently and are compliant with your governance rules from the moment of their creation.
- **Network and Security Perimeter:** Establish a baseline network architecture. Often, a dedicated "Network" account is created to manage shared network resources like AWS Transit Gateway, Direct Connect, and centralized traffic inspection points (firewalls).

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A large enterprise is migrating to AWS and needs to establish a secure, scalable, multi-account environment that aligns with AWS best practices from day one. Which AWS service is designed to automate the setup of this foundational environment?**

- A. AWS Organizations
- B. AWS Config
- C. AWS Control Tower
- D. AWS Service Catalog

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the core purpose of AWS Control Tower. It automates the creation of a "Landing Zone," which is a secure, well-architected multi-account environment. While it uses AWS Organizations (A), AWS Config (B), and Service Catalog (D) under the hood, Control Tower is the orchestrator that brings them all together to build the foundation.

</details>

---

**2. In an AWS Control Tower environment, what is the primary purpose of the "Log Archive" account?**

- A. To host the central dashboard for viewing guardrail compliance.
- B. To act as a central, immutable repository for all CloudTrail and AWS Config logs from across the organization.
- C. To provide security teams with cross-account access to perform audits.
- D. To manage the creation and vending of new AWS accounts.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The Log Archive account is specifically designed as a secure, central destination for logs. Its permissions are highly restricted to ensure that the audit and compliance logs sent from all other accounts cannot be tampered with or deleted, providing a single source of truth for all account activity.

</details>

---

**3. AWS Control Tower implements "Preventive" guardrails to enforce policies. Which underlying AWS service is used to implement these preventive controls?**

- A. AWS Config
- B. AWS CloudTrail
- C. Service Control Policies (SCPs)
- D. AWS IAM Identity Center

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Preventive guardrails work by stopping non-compliant actions from happening in the first place. This is the exact function of Service Control Policies (SCPs), which are part of AWS Organizations. Control Tower creates and manages these SCPs on your behalf. AWS Config (A) is used for detective guardrails.

</details>

---

**4. A company has used AWS Control Tower to set up its multi-account environment. A developer now needs a new, sandboxed AWS account for an experimental project. What is the recommended way to provision this account?**

- A. Create a new standalone account with a personal credit card and then invite it to the organization.
- B. Use the "Account Factory" feature within AWS Control Tower.
- C. Log in to the management account and manually create the account via the AWS Organizations console.
- D. Clone the Audit account to create a new sandbox environment.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The Account Factory is the standardized mechanism for vending new accounts in a Control Tower environment. Using it ensures that the new account is created with all the necessary baseline configurations, network settings, and guardrails already applied, making it compliant from the start.

</details>

---

**5. What is the function of the "Audit" account in an AWS Landing Zone set up by Control Tower?**

- A. It pays for all charges incurred by the member accounts.
- B. It is a highly restricted account for the exclusive use of AWS Support.
- C. It provides security and compliance teams with the necessary permissions to audit and manage security services across all accounts in the organization.
- D. It stores all of the organization's S3 data backups.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Audit account (sometimes called the Security Tooling account) is a centralized hub for security teams. It is granted the necessary cross-account roles to allow security personnel to access other accounts, run security tools (like Amazon GuardDuty, Security Hub), and perform investigations without needing credentials for each individual member account.

</details>

---

**6. A "Detective" guardrail in AWS Control Tower reports that an S3 bucket in a member account has been made public. Which underlying service detected this non-compliance?**

- A. Amazon Inspector
- B. AWS Config
- C. AWS Shield
- D. A Service Control Policy (SCP)

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Detective guardrails work by detecting non-compliant resources after they have been created. This is the primary function of AWS Config and its conformance rules. Control Tower creates and manages AWS Config rules across the organization to monitor for policy violations. An SCP (D) would have been a preventive control.

</details>

---

**7. A company wants to provide its developers with access to multiple AWS accounts using their existing corporate credentials from Active Directory. What component of a well-architected landing zone facilitates this?**

- A. Consolidated billing in the management account.
- B. Centralized logging to the Log Archive account.
- C. AWS IAM Identity Center (formerly AWS SSO) integration.
- D. The Account Factory for provisioning new accounts.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS IAM Identity Center is the key service for managing human access in a multi-account environment. It provides the Single Sign-On (SSO) capabilities that allow you to federate with an external Identity Provider (like Active Directory) and provide users with a central portal to access their assigned AWS accounts via temporary role assumption.

</details>

---

**8. Which of the following is NOT a core principle of a well-governed multi-account environment?**

- A. Centralized identity and access management.
- B. Storing long-term IAM user credentials in each member account for easy access.
- C. Enforcing security policies with a combination of preventive and detective guardrails.
- D. Automating the provisioning of new accounts to ensure consistency.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Storing long-term IAM user credentials in member accounts is a major security anti-pattern. A well-governed environment actively avoids this by using a centralized identity solution like IAM Identity Center and having users assume temporary roles in the member accounts.

</details>

---

**9. What is a "Landing Zone" in the context of AWS?**

- A. A specific AWS Region where all company resources must be deployed.
- B. A single, highly secured AWS account used for all production workloads.
- C. A well-architected, multi-account AWS environment configured with a secure and scalable baseline.
- D. An AWS service that automatically migrates on-premises applications to the cloud.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Landing Zone is not a single service or region but a methodology and outcome. It represents the foundational setup of a multi-account environment, pre-configured with best practices for security, networking, logging, and identity, providing a safe "landing" place for an organization's workloads.

</details>

---

**10. Why is it a best practice to centralize AWS CloudTrail logs in a separate Log Archive account?**

- A. To make the logs easier for developers in member accounts to access and delete.
- B. To reduce S3 storage costs by compressing all logs into a single file.
- C. To create a single source of truth for audit data and protect it from being modified or deleted by users in the source accounts.
- D. To automatically analyze logs with Amazon Athena.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The primary driver for centralizing logs is security and immutability. By sending logs to a separate, highly restricted account, you ensure that even an administrator in a member account cannot tamper with or erase their own activity logs. This preserves the integrity of the audit trail, which is critical for security investigations and compliance.

</details>

---

**11. A solutions architect is planning a multi-account strategy. They need to ensure that certain sensitive workloads are completely isolated from general development workloads. What is the strongest boundary of isolation AWS provides?**

- A. An IAM policy.
- B. A VPC.
- C. A separate AWS account.
- D. A resource tag.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** An AWS account is the ultimate boundary for security and resource isolation. Resources in one account are, by default, completely isolated from resources in another. While VPCs and IAM policies provide isolation within an account, separating workloads into different accounts provides the strongest possible separation.

</details>

---

**12. Which statement accurately describes the relationship between AWS Organizations and AWS Control Tower?**

- A. They are competing services that achieve the same outcome.
- B. AWS Control Tower is a feature within the AWS Organizations console.
- C. AWS Control Tower uses AWS Organizations as a foundational service to manage accounts and apply SCPs.
- D. You must set up AWS Control Tower before you can create an AWS Organization.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Control Tower is an orchestrator that builds upon and manages several other core services. It leverages AWS Organizations to create and manage the account structure and to implement preventive guardrails via SCPs. It is not a feature of Organizations but rather a higher-level management service.

</details>

---

**13. A company is concerned about developers accidentally enabling costly services in their sandbox accounts. A preventive guardrail is needed. Which technology would be used to implement this?**

- A. AWS Budgets Action
- B. Service Control Policy (SCP)
- C. Amazon CloudWatch Alarm
- D. AWS Config Rule

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A preventive control stops an action from happening. An SCP is the perfect tool for this, as it can be configured to deny the API actions associated with enabling or using specific services. AWS Budgets Actions (A) and CloudWatch Alarms (C) are reactive to cost, and AWS Config Rules (D) are reactive to non-compliant configurations (detective).

</details>

---

**14. A new account created via the AWS Control Tower Account Factory is automatically enrolled with certain features. Which feature is NOT typically part of this baseline?**

- A. A default VPC configured according to the network baseline.
- B. Centralized CloudTrail logging configured to the Log Archive account.
- C. A deployed production-ready web application.
- D. Necessary SCPs inherited from the parent OU.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Account Factory provisions the foundational infrastructure and governance for an account. It does not deploy business applications. The purpose of the landing zone is to create a safe and compliant environment into which the company can then deploy its own applications.

</details>

---

**15. What are the two core AWS accounts that AWS Control Tower creates in addition to the Management account? (Select TWO)**

- A. Network Account
- B. Log Archive Account
- C. Billing Account
- D. Security Tooling (Audit) Account
- E. DevOps CI/CD Account

<details>
<summary>View Answer</summary>

**Answers: B and D**

**Explanation:** The standard three-account setup for a Control Tower landing zone consists of the Management account (which you use to enable Control Tower), a new Log Archive account for immutable logs, and a new Security Tooling/Audit account for centralized security operations.

</details>

---

**16. A financial services company must prove to auditors that no data is stored outside of a specific AWS Region. What is the most effective way to enforce this across their entire AWS Organization?**

- A. Trust developers to only select the correct region when launching resources.
- B. Use AWS Control Tower to deploy a preventive guardrail (SCP) that denies access to all other Regions.
- C. Set up an AWS Budget for each unapproved region with a threshold of $1.
- D. Manually inspect all accounts for non-compliant resources every month.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A preventive guardrail, implemented as an SCP, is the strongest and most reliable way to enforce data residency requirements. By denying actions that are requested outside the approved region(s), you make it impossible for users to create non-compliant resources, providing a strong, auditable control.

</details>

---

**17. A company does not have the resources to manage a complex multi-account setup but wants to follow AWS best practices for security and governance. They prefer an automated, prescriptive solution. What should a solutions architect recommend?**

- A. Continue using a single AWS account but with very strict IAM policies.
- B. Manually build a landing zone using custom CloudFormation scripts.
- C. Use AWS Control Tower to get a managed, automated landing zone with pre-configured guardrails.
- D. Outsource all infrastructure management to a third party.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Control Tower is specifically designed for customers who want the benefits of a well-architected multi-account environment without the complexity of building and managing it themselves. It is the most direct and prescriptive path to achieving good governance with minimal operational overhead.

</details>

---

**18. What core problem does a multi-account strategy using AWS Organizations help solve?**

- A. It eliminates the need for network firewalls.
- B. It provides a way to isolate workloads and resources to reduce the "blast radius" of security incidents.
- C. It automatically makes all applications highly available across multiple regions.
- D. It reduces the cost of all AWS services by 50%.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** One of the primary drivers for a multi-account strategy is isolation. By separating different environments (e.g., prod vs. dev) or sensitive workloads (e.g., PCI data) into their own accounts, you contain the scope of impact. If a development account is compromised, the production account remains unaffected.

</details>

---

**19. A member account in a Control Tower environment has been decommissioned. What is the best practice for handling the CloudTrail logs from this account that are stored in the Log Archive account?**

- A. Delete the logs to save on S3 storage costs.
- B. Keep the logs for the period defined by the company's data retention policy for audit purposes.
- C. Move the logs to a cheaper storage class like S3 Glacier Deep Archive.
- D. Encrypt the logs with a new KMS key.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Audit logs are critical for security and compliance and must be retained according to the organization's retention policies, even after an account is closed. Deleting them (A) would violate this. While moving them to a cheaper storage class (C) might be part of the retention strategy, the primary driver is to keep them for the required period.

</details>

---

**20. You have an existing AWS Organization with several OUs and accounts. Can you deploy AWS Control Tower into this existing organization?**

- A. No, Control Tower can only be deployed into a brand new, empty management account.
- B. No, you must first delete the existing organization and start over with Control Tower.
- C. Yes, you can extend governance with Control Tower to an existing organization, but it requires prerequisites and a process to "enroll" existing OUs and accounts.
- D. Yes, it can be deployed with one click and will automatically manage all existing accounts.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Control Tower can be set up in an account that is already a management account for an existing organization. However, it's not a simple one-click process. You must first ensure the organization meets certain prerequisites, and then you can "register" your existing OUs and "enroll" your existing member accounts under Control Tower governance to apply the guardrails and other features.

</details>
