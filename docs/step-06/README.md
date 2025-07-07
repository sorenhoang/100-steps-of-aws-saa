# Step 06: AWS Organizations & Consolidated Billing

## 🎯 Objective

- [ ] Master: **AWS Organizations & Consolidated Billing**

## 📘 Notes

> Knowledge summary, documentation links, videos...

## 🧪 Practice

### **Deep Dive: AWS Organizations & Consolidated Billing**

As businesses grow on AWS, they quickly move from a single account to a multi-account environment to isolate workloads, manage security boundaries, and track costs. AWS Organizations is the service that makes this multi-account strategy manageable. It provides central governance and management, with consolidated billing as one of its most significant features.

### **1. AWS Organizations: The Structure** 🏛️

AWS Organizations allows you to create a hierarchical structure to group and manage your AWS accounts.

- **Key Terminology:**
  - **Organization:** The top-level entity that contains all of your AWS accounts.
  - **Root:** The parent container for all accounts in your organization. You can apply policies to the root that are inherited by everything beneath it.
  - **Management Account (formerly Master Account):** The account that creates the organization. This account is responsible for paying all charges (consolidated billing) and cannot be removed from the organization. It has the authority to create new accounts, invite existing accounts, and apply policies.
  - **Member Account:** Any AWS account, other than the management account, that is part of an organization.
  - **Organizational Unit (OU):** A container for accounts within the root. OUs can also contain other OUs, allowing you to create a hierarchy that reflects your business needs (e.g., by department, environment, or project). You can apply policies to an OU, and they will be inherited by all accounts and sub-OUs within it.
- **Feature Sets:**
  - **Consolidated Billing only:** The default and most basic feature set. It provides shared billing and cost management features.
  - **All Features:** This is the recommended setting. It includes everything in Consolidated Billing _plus_ advanced policy-based controls, most notably **Service Control Policies (SCPs)**, which allow you to centrally control the maximum available permissions for accounts in your organization.

### **2. Consolidated Billing** 💵

Consolidated billing is a key feature of AWS Organizations that simplifies payment and can significantly reduce costs.

- **How it Works:** All charges incurred by the member accounts are rolled up to the management account. The management account receives a single, consolidated bill covering all accounts in the organization. The individual member accounts can still view their own usage and costs, but they do not receive a bill from AWS.
- **Key Benefits:**
  1. **One Bill:** You receive a single bill for multiple accounts, simplifying accounting and payments.
  2. **Easy Tracking:** The management account can view itemized charges for each member account, making it easy to track costs and perform internal chargebacks. AWS Cost Explorer in the management account has a consolidated view of all member account spending.
  3. **Volume Pricing Discounts (Tiering):** This is a major cost-saving benefit. For services priced in tiers (like S3, where the per-GB cost decreases as you store more data), AWS combines the usage from **all accounts** in the organization to determine which pricing tier you qualify for. For example, if 10 accounts each use 50 TB of S3 storage, AWS treats it as 500 TB of usage, likely placing you in a cheaper per-GB pricing tier than any individual account would achieve alone.
  4. **Shared Reserved Instances (RIs) and Savings Plans:** If the management account enables discount sharing, the pricing benefits of RIs and Savings Plans purchased in any account can be shared across all other accounts in the organization. For example, if Account A has an unused EC2 Reserved Instance, and Account B launches a matching instance, Account B will automatically receive the discounted pricing from Account A's RI. This maximizes the utilization of your commitments and reduces waste.
- **Cost Allocation Tags:** To effectively manage costs, it's a best practice to use cost allocation tags (e.g., `cost-center:123`, `project:athena`). When you activate these tags in the Billing and Cost Management console of the management account, they appear as filterable columns in your cost reports, allowing you to categorize and track costs with high granularity across your organization.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company with multiple AWS accounts wants to simplify its payment process and benefit from volume pricing discounts for services like Amazon S3. What is the most effective solution?**

- A. Create a new AWS account and manually transfer the resources from all other accounts into it.
- B. Use AWS Organizations to group the accounts and enable consolidated billing.
- C. Set up a cross-account IAM role from each account to a central "billing" account.
- D. Purchase Reserved Instances in each account individually.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the primary use case for AWS Organizations. By creating an organization and inviting the accounts, you automatically enable consolidated billing. This provides a single bill for all accounts and, crucially, aggregates usage across accounts to help you reach volume pricing tiers faster, which directly addresses the customer's requirements.

</details>


---

**2. A company uses AWS Organizations with consolidated billing. One of its member accounts purchases an EC2 Reserved Instance (RI). A different member account then launches an EC2 instance that matches the specifications of the RI. What will be the result, assuming default settings are used?**

- A. The second account will be charged the standard On-Demand rate for its instance.
- B. The Reserved Instance discount will be applied to the instance in the second account.
- C. An error will occur because an RI purchased in one account cannot be used by another.
- D. Both accounts will be charged 50% of the On-Demand rate.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** By default, Reserved Instance and Savings Plans discounts are shared across all accounts within an AWS Organization. If an RI is unused in the account that purchased it, AWS automatically applies that discount to any matching instance usage in any other member account. This maximizes the value of the RI commitment.

</details>


---

**3. In an AWS Organization, which account is responsible for paying all charges?**

- A. The member account with the highest usage.
- B. The account designated as the "billing delegate."
- C. The management account.
- D. Each account pays for its own charges individually.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The management account (formerly master account) is the single paying account for the entire organization. It receives a consolidated bill for all charges incurred by itself and all member accounts.

</details>


---

**4. A company has structured its AWS Organization with separate Organizational Units (OUs) for its Development, Testing, and Production environments. The security team wants to prevent anyone in the Development OU from using certain high-cost AWS services. What is the most effective way to enforce this?**

- A. Enable "All Features" in the organization and apply a Service Control Policy (SCP) to the Development OU.
- B. Set up a budget for the Development OU that stops all resources when the limit is reached.
- C. Manually apply an IAM policy to every user and role in the accounts within the Development OU.
- D. Use AWS Trusted Advisor to monitor for the use of unapproved services.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** This is a primary use case for Service Control Policies (SCPs). SCPs act as guardrails that can deny access to specific AWS services or actions. By enabling "All Features" and applying a restrictive SCP to the Development OU, you can centrally enforce this policy for all accounts within that OU without having to manage individual IAM policies.

</details>


---

**5. What is the main benefit of aggregated usage with consolidated billing in AWS Organizations?**

- A. It simplifies the process of creating new AWS accounts.
- B. It allows all accounts to share a single set of IAM users.
- C. It combines usage from all accounts to potentially qualify for lower-priced tiers on some AWS services.
- D. It enables cross-region VPC peering by default.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The term "aggregated usage" or "volume pricing" refers to AWS combining the total usage of a service (like S3 or Data Transfer) from all member accounts. This combined total is then used to calculate the price, often resulting in a lower per-unit cost than if each account were billed individually.

</details>


---

**6. Which of the following is a key characteristic of the management account in an AWS Organization?**

- A. It can be changed to a different member account at any time.
- B. It can be restricted by Service Control Policies (SCPs) applied at the root.
- C. It cannot be removed from the organization unless the organization is deleted.
- D. It has read-only access to member accounts by default.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The management account is fundamental to the organization's existence. It cannot leave the organization. The only way to remove it is to first remove all member accounts and then delete the organization itself, which reverts the management account to a standalone account. It's important to note that the management account itself is not restricted by SCPs applied at the root or OUs it manages.

</details>


---

**7. A company needs to track AWS costs by project and by department. All projects run in various member accounts within an AWS Organization. What is the recommended approach?**

- A. Create a separate AWS Organization for each department.
- B. Use AWS Budgets to create project-specific spending limits.
- C. Implement a consistent tagging strategy and activate the tags as cost allocation tags in the management account.
- D. Analyze the bill on the management account and manually assign costs to departments.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Cost allocation tags are the primary mechanism for detailed cost tracking in AWS. By consistently applying tags (e.g., Project:Phoenix, Department:Marketing) to resources across all member accounts and then activating those tag keys in the Billing console, you can filter and group costs by those tags in Cost Explorer and other reports.

</details>


---

**8. An existing standalone AWS account needs to join an AWS Organization. What is the process?**

- A. The standalone account sends an invitation to the organization's management account.
- B. The management account of the organization sends an invitation to the standalone account's ID, which the standalone account's owner must accept.
- C. A support ticket must be raised with AWS to merge the accounts.
- D. The management account imports the standalone account using its root user credentials.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The process is initiated by the management account. It sends an invitation to the target account's ID. An administrator (typically the root user) of the standalone account then sees this invitation in their console and must explicitly accept it to join the organization.

</details>


---

**9. What happens to the billing of a member account after it joins an AWS Organization?**

- A. The member account continues to receive its own bill from AWS.
- B. The member account's designated payment method is charged, and the cost is then reimbursed by the management account.
- C. The member account stops receiving a bill from AWS, and all its costs are rolled up into the management account's consolidated bill.
- D. The member account's billing is paused until the end of the fiscal year.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Once an account successfully joins an organization, its direct billing relationship with AWS ceases. It no longer pays AWS directly. Instead, all of its usage costs are consolidated and paid for by the management account. The member account can still view its usage in its own console, but it will not receive a payable invoice.

</details>


---

**10. You want to create a new AWS account that is automatically part of your existing organization and has certain security policies applied from the start. What is the best way to do this?**

- A. Create a new standalone account and then invite it to the organization.
- B. Use the "Create account" feature within the AWS Organizations console of the management account.
- C. Contact AWS Support with a request to create a new account.
- D. Clone an existing member account using the AWS CLI.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The AWS Organizations service provides a feature to programmatically create new member accounts directly from the management account. When an account is created this way, it is automatically part of the organization and immediately inherits any SCPs applied to its parent OU or the root.

</details>


---

**11. A large enterprise has multiple business units, each with different compliance and security requirements. How should they structure their accounts in AWS Organizations?**

- A. Place all accounts directly under the root of the organization.
- B. Create a separate Organizational Unit (OU) for each business unit to allow for tailored policy application.
- C. Create a single member account and use IAM roles to separate the business units.
- D. Use a different AWS Region for each business unit.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Organizational Units (OUs) are designed for this exact purpose. By creating an OU for each business unit (e.g., Finance, HR, Engineering), you can apply distinct Service Control Policies (SCPs) to each OU, enforcing the specific security and compliance guardrails required by that unit.

</details>


---

**12. To enable advanced features like Service Control Policies (SCPs) in an organization that was initially created for Consolidated Billing only, what must be done?**

- A. Raise a support request to AWS.
- B. Re-create the entire organization from scratch.
- C. Enable "All Features" from the settings page in the AWS Organizations console. All member accounts must approve the change.
- D. Manually create an SCP, which will automatically enable the feature.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** You can migrate an organization from the "Consolidated Billing only" feature set to the "All Features" set. This is done from the settings of the AWS Organizations console. However, this is a significant change, and for it to be enabled, every invited member account in the organization must accept the request to switch to all features.

</details>


---

**13. A company has an active AWS Enterprise Support plan on its management account. What level of support is available to its member accounts?**

- A. All member accounts are automatically upgraded to the Enterprise Support plan.
- B. All member accounts must purchase their own Enterprise Support plan.
- C. Member accounts can choose between Basic, Developer, and Business support plans.
- D. Member accounts will remain on the Basic Support plan.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** When the management account has a Business or Enterprise support plan, that support level extends to all member accounts within the organization. This is a significant benefit, allowing all accounts access to premium support features while paying for it once at the management account level.

</details>


---

**14. What happens when a member account leaves an organization?**

- A. The account is automatically deleted.
- B. The account becomes a standalone account and is responsible for its own billing from that point forward.
- C. The account can no longer be used and must be closed.
- D. The account remains part of the organization but stops sharing its billing information.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When an account is successfully removed from an organization, it reverts to being a standalone AWS account. It immediately becomes responsible for its own costs and must have a valid payment method on file. It does not inherit any SCPs from its former organization.

</details>


---

**15. Where would a solutions architect go to see a combined view of the costs incurred by all member accounts in an organization?**

- A. The AWS CloudTrail console of the management account.
- B. The AWS Budgets page of each member account.
- C. The AWS Cost Explorer service in the management account's console.
- D. The AWS Trusted Advisor dashboard.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Cost Explorer, when accessed from the management account, provides a consolidated view of costs and usage for the entire organization. It allows you to filter and group costs by account, service, tags, and other dimensions to analyze spending patterns across all member accounts.

</details>


---

**16. Which of the following is NOT a direct benefit of using AWS Organizations?**

- A. Centralized application of security guardrails using SCPs.
- B. Simplified billing with a single invoice.
- C. Automatic high-availability for all resources in member accounts.
- D. Sharing of Reserved Instance and Savings Plans discounts.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Organizations is a management and governance service. It does not automatically configure or provide high-availability for the resources (like EC2 instances or RDS databases) running within the member accounts. High-availability is an architectural responsibility that must be designed and implemented by the account owners, for example, by deploying resources across multiple Availability Zones.

</details>


---

**17. Can the management account be moved into an Organizational Unit (OU)?**

- A. Yes, it can be moved into any OU just like a member account.
- B. No, the management account must always reside directly under the root.
- C. Yes, but only if it's the only account in that OU.
- D. No, only member accounts can be placed in OUs.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The management account has a special status and cannot be placed inside an OU. It always resides at the top of the hierarchy, directly under the root container. Only member accounts can be organized into OUs.

</details>


---

**18. A company is using AWS Organizations. The management account has `iam:CreateUser` denied by an SCP attached to the root. An administrator in the management account tries to create a new IAM user. What will happen?**

- A. The action will be denied because the SCP applies to the management account.
- B. The action will be successful because SCPs do not affect the management account.
- C. The action will be successful only if the administrator has `AdministratorAccess`.
- D. The action will be denied because you cannot create IAM users in a management account.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A critical rule to remember is that Service Control Policies (SCPs) do not apply to the management account. They are designed to enforce guardrails on the member accounts only. Therefore, any policy attached at the root or an OU does not restrict principals (users or roles) within the management account.

</details>


---

**19. What is a primary purpose of using Organizational Units (OUs)?**

- A. To group accounts for easier billing and apply policies collectively.
- B. To create regional boundaries for resources.
- C. To define the payment method for a group of accounts.
- D. To share AMIs and Snapshots between accounts.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** OUs serve two main purposes: grouping accounts logically (which helps in reporting and management) and applying policies (like SCPs or Backup policies) to that group of accounts in a single action. This hierarchical policy inheritance is a core feature of AWS Organizations.

</details>


---

**20. A company has a mix of production and development accounts. They want to share the benefit of their Reserved Instances with production accounts first before any development accounts can use them. Can this be configured?**

- A. No, RI sharing is random and cannot be controlled.
- B. Yes, this can be configured by applying a specific SCP to the development OU.
- C. Yes, in the management account's Billing preferences, RI and Savings Plans discount sharing can be turned off for specific OUs or accounts.
- D. No, all accounts in the organization share RI benefits equally.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While sharing is on by default, the management account has granular control. From the Billing and Cost Management console, an administrator can disable discount sharing for specific OUs or individual member accounts. This would cause instances in those disabled accounts to be billed at the On-Demand rate, preserving the RI discounts for the remaining accounts.

</details>
