# Step 05: Root Account, Billing Alerts, Trusted Advisor

## 🎯 Objective
- [ ] Master: **Root Account, Billing Alerts, Trusted Advisor**

## 📘 Notes
> Knowledge summary, documentation links, videos...

### **Deep Dive: Root Account, Billing Alerts, & Trusted Advisor**

Managing an AWS account effectively goes beyond just architecting applications. It requires strong foundational security, proactive cost management, and continuous optimization. This lecture covers three critical pillars: securing the most privileged identity (the Root User), monitoring your spending with Billing Alerts, and using AWS's automated expert, Trusted Advisor, to guide you.

### **1. The AWS Account Root User** 👑

The Root User is the original identity created when an AWS account is opened. It is the superuser of the account and has **unrestricted access** to all AWS services and resources, including complete control over billing information and the ability to change the account's password.

- **Key Characteristics:**
    - **Full, Unrestricted Privileges:** The Root User cannot be limited by IAM policies. It can do anything, including deleting the entire account.
    - **Unique Identity:** There is only one Root User per AWS account, and it's authenticated by the email address and password used to create the account.
    - **High-Risk Target:** Because of its ultimate power, the Root User is a primary target for attackers.
- **Security Best Practices (Crucial for the Exam):**
    1. **Do Not Use for Daily Tasks:** The Root User should never be used for routine administrative or operational work. For that, create an IAM user with administrative privileges.
    2. **Enable Multi-Factor Authentication (MFA):** This is the single most important security measure. Lock down the Root User with a virtual or hardware MFA device immediately after creating the account.
    3. **Do Not Create Access Keys:** There is almost no valid reason to have programmatic access keys (access key ID and secret access key) for the Root User. If they exist, they should be deleted.
    4. **Securely Store Credentials:** The password and the physical MFA device (if used) for the Root User should be stored in a physically secure location, like a safe. Access should be tightly controlled and logged.

### **2. Billing Alerts and AWS Budgets** 💰🔔

Proactively managing cloud costs is a core responsibility. AWS provides two key tools for this: CloudWatch Billing Alarms and AWS Budgets.

- **CloudWatch Billing Alarms:**
    - **How it works:** This is a specific type of Amazon CloudWatch alarm that monitors your estimated AWS charges. First, you must enable "Billing Alerts" in the Billing and Cost Management console preferences (this must be done by the Root User or an IAM user with billing permissions). Once enabled, the `EstimatedCharges` metric is published to CloudWatch in the `us-east-1` region. You can then create an alarm on this metric.
    - **Functionality:** When your spending exceeds a threshold you define (e.g., $500), the alarm triggers an action, typically sending a notification to an Amazon SNS topic.
    - **Primary Use:** Provides a simple, direct notification when your total monthly bill crosses a specific dollar amount.
- **AWS Budgets:**
    - **How it works:** AWS Budgets is a more advanced and flexible tool. It allows you to set custom budgets and receive alerts when your costs or usage exceed (or are forecasted to exceed) your budgeted amount.
    - **Functionality:**
        - **Forecasting:** Can alert you when it *predicts* you will go over budget, giving you time to react before it happens.
        - **Granularity:** Budgets can be set for total costs, or filtered by specific services (e.g., EC2), linked accounts, tags, and more.
        - **Usage Budgets:** You can budget based on usage metrics (e.g., GB-months of S3 storage) in addition to cost.
        - **Budget Actions:** Can be configured to automatically take action, such as applying a restrictive IAM policy or stopping EC2/RDS instances when a budget threshold is met.
    - **Primary Use:** Comprehensive cost management, forecasting, and automated cost control actions.

**Key Difference:** A CloudWatch Billing Alarm is a simple "you have spent $X" notification. AWS Budgets is a powerful service for "you are on track to spend $Y, which is over your budget of $X." For the SAA-C03 exam, if you see the word **"forecasted,"** think **AWS Budgets**.

### **3. AWS Trusted Advisor** 👨‍⚕️✅

Trusted Advisor is an automated service that inspects your AWS environment and makes recommendations to help you follow AWS best practices. It acts as your personalized cloud expert.

- **Core Check Categories:** Trusted Advisor provides checks across five key categories. For the exam, you must know these categories:
    1. **Cost Optimization:** Identifies idle resources (like Load Balancers with no backend instances), underutilized EBS volumes, and makes recommendations for Reserved Instances and Savings Plans.
    2. **Performance:** Checks for things that can impact application performance, like over-utilized EC2 instances or CloudFront configuration issues.
    3. **Security:** Looks for common security misconfigurations, such as unrestricted access to ports in security groups, lack of MFA on the root account, and public S3 buckets.
    4. **Fault Tolerance:** Checks for lack of redundancy, such as EC2 instances in a single Availability Zone (AZ), missing backups for RDS, or misconfigured Auto Scaling Groups.
    5. **Service Limits:** Checks to see if you are approaching any service quotas (formerly called service limits) for your account (e.g., the number of VPCs or EC2 instances you can launch).
- **Support Plan Tiers:** The number of checks you have access to depends on your AWS Support plan.
    - **All Accounts (Basic/Developer Support):** Get access to 7 core security and performance checks (e.g., MFA on Root Account, S3 Bucket Permissions, Service Limits).
    - **Business and Enterprise Support:** Get access to the **full set** of over 100 Trusted Advisor checks across all five categories.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company has just created a new AWS account. What is the most critical security step that should be taken immediately regarding the root user?**

- A. Create an access key and secret key for the root user for automation scripts.
- B. Enable Multi-Factor Authentication (MFA) and then lock away the root user credentials.
- C. Use the root user to create all necessary IAM users and roles for the team.
- D. Change the root user's email address to a distribution list for shared access.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS's #1 best practice is to secure the root user immediately after account creation. This involves enabling MFA and then not using the root user for any daily tasks. Creating access keys (A) is a major security anti-pattern. While the root user might be used to create the very first administrative IAM user, it shouldn't be used for all of them (C). Sharing the root user via a distribution list (D) is extremely insecure.

</details>
    

---

**2. A startup is concerned about its monthly AWS bill and wants to be notified if its total spending is *forecasted* to exceed $1,000 for the month. Which AWS service should they use to configure this specific type of alert?**

- A. AWS CloudTrail
- B. A standard Amazon CloudWatch billing alarm
- C. AWS Budgets
- D. AWS Trusted Advisor
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The key word in the question is "forecasted." Only AWS Budgets can generate alerts based on predicted spending. A standard CloudWatch billing alarm (B) can only alert when the actual spending has already crossed a threshold. CloudTrail (A) is for auditing API calls, and Trusted Advisor (D) provides recommendations but doesn't handle real-time budget forecasting alerts.

</details>
    

---

**3. A solutions architect is reviewing a new client's AWS account and finds that several security groups allow unrestricted inbound access (0.0.0.0/0) on port 22 (SSH) and port 3389 (RDP). Which AWS service would proactively identify this specific security vulnerability?**

- A. Amazon GuardDuty
- B. AWS Shield
- C. AWS Trusted Advisor
- D. AWS Config
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Trusted Advisor "Security Groups - Specific Ports Unrestricted" check is designed to find exactly this type of misconfiguration. It's a core security check available to all AWS accounts. While AWS Config could be used to create a rule for this, Trusted Advisor provides this check out-of-the-box. GuardDuty (A) detects threats and malicious activity, while Shield (B) is for DDoS protection.

</details>
    

---

**4. To create a CloudWatch alarm based on the `EstimatedCharges` metric, what must be enabled first in the AWS account?**

- A. Detailed monitoring for all EC2 instances.
- B. AWS Cost Explorer in the Billing console.
- C. Billing alerts in the Billing and Cost Management console preferences.
- D. A support plan of Business or higher.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** By default, billing metric data is not sent to CloudWatch. You must first go into the Billing and Cost Management console, navigate to "Billing preferences," and explicitly check the box to "Receive billing alerts." This action, which must be done by the root user or an IAM user with proper permissions, enables the EstimatedCharges metric to be published to CloudWatch in us-east-1.

</details>
    

---

**5. A company has an AWS Business Support plan. Which of the following recommendations would they receive from AWS Trusted Advisor that someone with a Basic Support plan would NOT?**

- A. A notification that MFA is not enabled on the root account.
- B. An alert that they are approaching a service limit for the number of VPCs.
- C. A recommendation to purchase Reserved Instances for EC2 instances that have been running continuously.
- D. A warning that an S3 bucket has open access permissions.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Cost optimization checks, such as recommendations for Reserved Instances or identifying idle resources, are only available to customers with a Business or Enterprise support plan. The other options (MFA on root, service limits, S3 bucket permissions) are part of the core set of checks available to all AWS accounts, regardless of support plan.

</details>
    

---

**6. An administrator needs to perform a task that can only be done by the root user, such as changing the AWS Support plan. What is the correct procedure?**

- A. Use an IAM user with `AdministratorAccess` and `sudo` to the root user.
- B. Log in to the AWS console using the root user's email and password, perform the task, and then sign out immediately.
- C. Contact AWS Support and ask them to perform the task on your behalf.
- D. Use the root user's access keys with the AWS CLI.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For the very few tasks that require the root user, the correct procedure is to use the root credentials to sign in to the console, complete the single necessary action, and sign out. There is no sudo concept in AWS (A). AWS Support cannot perform these actions for you (C). Using root access keys is highly discouraged and should be avoided (D).

</details>
    

---

**7. A company wants to automatically stop all EC2 instances in a development account if the monthly cost exceeds $500 to prevent budget overruns. Which service allows for this type of automated remedial action?**

- A. AWS Trusted Advisor
- B. Amazon CloudWatch Events
- C. AWS Budgets
- D. Service Control Policies (SCPs)
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Budgets has a feature called "Budget Actions." This allows you to configure an action, such as applying a restrictive IAM policy or stopping specific EC2 or RDS instances, when a budget threshold is met. This is a powerful tool for automated cost control.

</details>
    

---

**8. Which Trusted Advisor category would provide a recommendation to create snapshots for your Amazon EBS volumes?**

- A. Performance
- B. Security
- C. Cost Optimization
- D. Fault Tolerance
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The "EBS Snapshots" check falls under the Fault Tolerance category. Trusted Advisor identifies volumes that have not been backed up recently, as a lack of backups is a risk to data durability and the ability to recover from failure.

</details>
    

---

**9. Which of the following is NOT a best practice for the AWS account root user?**

- A. Enabling MFA on the root user.
- B. Creating a strong, complex password for the root user.
- C. Deleting any existing access keys for the root user.
- D. Attaching an IAM policy to the root user to limit its permissions.
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** IAM policies cannot restrict the root user. The root user has implicit full access to all resources in the account, and this cannot be changed or limited by attaching an IAM policy. The other three options are all critical best practices.

</details>
    

---

**10. A user has set up a CloudWatch billing alarm for $100. The company's spending for the month is currently $90. The user also has an AWS Budget set for $100, which is configured to send an alert at 80% of the forecasted amount. The forecast predicts the total monthly spend will be $120. Which alert will be sent first?**

- A. The CloudWatch billing alarm.
- B. The AWS Budgets alert.
- C. Both will be sent at the same time.
- D. Neither will be sent.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The AWS Budgets alert is based on the forecast. Since the forecast ($120) is over the budget ($100), the 80% threshold has already been met from a forecasting perspective, and an alert will be sent. The CloudWatch billing alarm is based on actual spend. It will not trigger until the bill actually crosses the $100 mark.

</details>
    

---

**11. Which two checks are available from AWS Trusted Advisor for ALL customers, regardless of their support plan? (Select TWO)**

- A. Idle Load Balancers
- B. Service Limits
- C. EC2 Reserved Instance Optimization
- D. MFA on Root Account
- E. Underutilized EBS Volumes
<details>
<summary>View Answer</summary>

**Answers: B and D**

**Explanation:** AWS provides a core set of seven checks for free to all customers. Among the options listed, "Service Limits" and "MFA on Root Account" are part of this core set. The other options (A, C, E) are cost optimization checks, which require a Business or Enterprise support plan.

</details>
    

---

**12. A financial controller needs to view billing information but should not have access to any other AWS resources. What is the most secure way to grant this access?**

- A. Share the root user credentials with the financial controller.
- B. Create an IAM user for the controller and grant them the `AdministratorAccess` managed policy.
- C. Activate IAM user and role access to the Billing and Cost Management console from the Account Settings page, then create an IAM policy granting only billing permissions.
- D. Create an IAM user and add them to a "Billing" group, then email them the group's access keys.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** By default, IAM users do not have access to the billing console. The root user must first delegate this access by enabling it in the account settings. Once enabled, you can create a specific IAM user or role with a policy that only grants permissions to view billing data, adhering to the principle of least privilege. Sharing root credentials (A) or granting admin access (B) is dangerously excessive.

</details>
    

---

**13. A company is running a production workload on an EC2 instance but has not deployed a standby instance in a different Availability Zone. Which Trusted Advisor category would flag this as a potential issue?**

- A. Performance
- B. Security
- C. Fault Tolerance
- D. Cost Optimization
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Fault Tolerance category is concerned with the availability and resilience of your architecture. Running a critical instance in only one AZ is a single point of failure. Trusted Advisor's "EC2 Availability Zone Balance" check would identify this and recommend distributing instances across multiple AZs.

</details>
    

---

**14. An organization wants to create different spending budgets for its Development, Testing, and Production environments, which are all running in the same AWS account. How can they accomplish this?**

- A. Create three separate AWS accounts for each environment.
- B. Use AWS Budgets and filter the cost by applying specific resource tags to the resources in each environment.
- C. Create three separate CloudWatch billing alarms.
- D. This is not possible; budgets can only be set for the entire account.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS Budgets is a powerful tool that allows for fine-grained budget creation. One of its key features is the ability to filter costs by resource tags. By tagging all resources with their respective environment (e.g., environment:dev, environment:prod), the company can create separate budgets to track the costs for each one independently.

</details>
    

---

**15. What AWS credential is used to log in to the AWS account root user?**

- A. The access key ID and secret access key.
- B. The account number and an IAM user password.
- C. The email address used to create the account and its associated password.
- D. An X.509 certificate.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The unique identifier for the root user is the email address that was used when the AWS account was first created. This email, combined with the password set at that time, constitutes the root user's login credentials for the AWS Management Console.

</details>
    

---

**16. The AWS Billing and Cost Management console shows an unexpected spike in data transfer costs. Which service should be used to investigate the source of this activity by reviewing API call logs?**

- A. AWS Trusted Advisor
- B. AWS Cost Explorer
- C. AWS CloudTrail
- D. AWS Budgets
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While Cost Explorer (B) can help visualize the cost spike, AWS CloudTrail is the service used for auditing and logging API activity. By analyzing CloudTrail logs, a solutions architect can determine which users or services made the API calls that resulted in the high data transfer, helping to pinpoint the cause.

</details>
    

---

**17. A team is approaching its service limit for the number of security groups in a region. Which service will proactively warn them about this?**

- A. Amazon CloudWatch
- B. AWS Health
- C. AWS Service Catalog
- D. AWS Trusted Advisor
<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The "Service Limits" check is one of the core checks provided by AWS Trusted Advisor and is available to all accounts. It monitors your usage of various AWS resources against the account's quotas and provides a warning when you are approaching a limit, giving you time to request an increase if needed.

</details>
    

---

**18. What is the primary difference between how a CloudWatch billing alarm is configured versus how an AWS Budget is configured?**

- A. Billing alarms are configured in the CloudWatch console, while Budgets are configured in the Billing console.
- B. Billing alarms can trigger automated actions, while Budgets can only send notifications.
- C. Billing alarms are only available for Enterprise support plans.
- D. Budgets can only track total account spending, while alarms can track specific services.
<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** This is a key operational difference. To set up an AWS Budget, you go directly to the AWS Budgets page within the Billing and Cost Management console. To set up a billing alarm, you must first enable the metric in the Billing console and then go to the Amazon CloudWatch console to create the alarm itself.

</details>
    

---

**19. You receive a Trusted Advisor notification for "Amazon S3 Bucket Permissions" with a red status. What does this most likely indicate?**

- A. One of your S3 buckets is nearing its storage capacity.
- B. You have an S3 bucket with permissions that allow public list or write access.
- C. Versioning has been disabled on a critical S3 bucket.
- D. An S3 bucket does not have a lifecycle policy configured.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The S3 Bucket Permissions check is a core security check that specifically looks for buckets that have "open" or public access. Allowing List access to everyone can lead to unexpected charges, and allowing public Upload/Delete access is a major security vulnerability.

</details>
    

---

**20. A manager wants a daily report of the company's progress against their quarterly AWS budget. What is the easiest way to achieve this?**

- A. Manually log into the console each day and take a screenshot of the billing dashboard.
- B. Configure a CloudWatch billing alarm to send a notification to the manager's email every 24 hours.
- C. Set up an AWS Budget and configure a "Budget Report" to be emailed daily.
- D. Write a Lambda function to query the Cost Explorer API and email the results.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Budgets has a specific feature called "Budget Reports" designed for this exact use case. You can configure a report to be sent on a daily, weekly, or monthly cadence to specified email addresses, providing a summary of the budget's status without requiring manual intervention. While option D is possible, it is far more complex than using the built-in functionality.

</details>