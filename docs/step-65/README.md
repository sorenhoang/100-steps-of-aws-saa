# Step 65: Trusted Advisor Core Checks

## 🎯 Objective

- [x] Master: **Trusted Advisor Core Checks**

## 📘 Notes

### **Architectural Deep Dive: Trusted Advisor Core Checks**

AWS Trusted Advisor is an online tool that provides you with real-time guidance to help you provision your resources following AWS best practices. Think of it as an automated consultant that inspects your AWS environment and makes recommendations. These recommendations are aligned with the pillars of the AWS Well-Architected Framework.

While the full version of Trusted Advisor offers over 100 checks across five categories, AWS provides a fundamental set of **core checks** to **all customers** at no additional cost, regardless of their support plan. These core checks are focused on the most critical aspects of security and performance.

---

### **The Five Categories of Trusted Advisor**

Before diving into the core checks, it's important to remember the five pillars that Trusted Advisor analyzes.

1. **Cost Optimization:** Recommendations to reduce your spending.
2. **Performance:** Recommendations to improve the speed and responsiveness of your applications.
3. **Security:** Recommendations to close security gaps.
4. **Fault Tolerance:** Recommendations to improve the availability and resilience of your applications.
5. **Service Quotas (formerly Service Limits):** Checks for usage that is approaching service quotas.

---

### **The Core Checks (Available to All Accounts)**

For the SAA-C03 exam, you should be familiar with the core checks that everyone gets for free. They fall under the **Security** and **Service Quotas** categories.

**Security Checks:**

1. **MFA on Root Account:** Checks if Multi-Factor Authentication is enabled for the root user. This is one of the most critical security best practices for any AWS account.
2. **IAM Access Key Rotation:** Checks for active IAM user access keys that have not been rotated in the last 90 days.
3. **Security Groups - Specific Ports Unrestricted:** Scans your security groups for rules that allow unrestricted inbound access (`0.0.0.0/0`) to high-risk ports like SSH (22), RDP (3389), FTP (20, 21), etc.
4. **Amazon S3 Bucket Permissions:** Checks for S3 buckets that have open access permissions (public read or write access).
5. **IAM Use:** Checks if you are using an IAM user in your account. The recommendation is to use IAM users for daily access instead of the root user.

Service Quotas Check:

6.  Service Quotas: Checks your usage of key AWS services and warns you when you are approaching more than 80% of your service quota for a resource (e.g., number of VPCs, EC2 instances, or EBS volumes).

_(Note: The list of free checks can change slightly, but these are the historically consistent and most important ones to know for the exam)._ The key takeaway is that the free checks focus on foundational security and preventing operational issues from hitting limits. The other categories (Cost Optimization, Performance, Fault Tolerance) are almost entirely for customers with a Business or Enterprise support plan.

---

### **SAA-C03 Style Scenario Questions**

**1. A new intern in a company has accidentally configured a security group to allow unrestricted SSH access (`0.0.0.0/0` on port 22) to a production web server. Which AWS service would proactively detect and flag this specific security misconfiguration?**

- A. Amazon GuardDuty
- B. AWS Shield
- C. AWS Config
- D. AWS Trusted Advisor

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** This is a direct check performed by AWS Trusted Advisor. The "Security Groups - Specific Ports Unrestricted" check is one of the core security checks available to all AWS accounts and is designed to find exactly this type of common security vulnerability. While AWS Config (C) could be configured with a custom rule to find this, Trusted Advisor does it automatically out-of-the-box.

</details>

---

**2. A company is using an AWS account with a Basic support plan. They want to find out if they are approaching the default limit for the number of VPCs they can create in a region. Which service can provide this information?**

- A. VPC Dashboard
- B. Amazon CloudWatch
- C. AWS Trusted Advisor
- D. This check is only available with an Enterprise support plan.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Service Quotas check (formerly Service Limits) is a core check available to all customers, including those on the Basic plan. It monitors your usage of numerous services and provides a warning when you approach 80% of the quota, giving you time to request an increase before your operations are impacted.

</details>

---

**3. An auditor is reviewing a company's AWS account and needs to verify that Multi-Factor Authentication (MFA) is enabled on the root user. What is the quickest way for an administrator to check this status using an automated AWS tool?**

- A. Check the IAM console.
- B. Run a script using the AWS CLI.
- C. View the results of the Security checks in AWS Trusted Advisor.
- D. Review the latest CloudTrail logs.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Trusted Advisor provides an automated tool that specifically checks for MFA on the root account as one of its core security checks. This is the quickest automated way to verify this critical security configuration without manually checking the IAM console or writing custom scripts.

</details>

---

**4. A company has a Business support plan. They are looking for ways to reduce their monthly bill. Which Trusted Advisor category would provide recommendations for identifying idle RDS instances or underutilized EBS volumes?**

- A. Security
- B. Performance
- C. Cost Optimization
- D. Fault Tolerance

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Cost Optimization is the Trusted Advisor category that provides recommendations to reduce spending. This includes identifying idle RDS instances, underutilized EBS volumes, and other resources that could be right-sized or terminated to save costs. This category is available to customers with Business or Enterprise support plans.

</details>

---

**5. Which of the following is a core security check provided by Trusted Advisor to all AWS accounts?**

- A. A check for EC2 instances with high CPU utilization.
- B. A check for S3 buckets with open access permissions.
- C. A check for RDS instances that are not in a Multi-AZ configuration.
- D. A recommendation to purchase EC2 Reserved Instances.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The "Amazon S3 Bucket Permissions" check is one of the core security checks available to all AWS accounts at no additional cost. It identifies S3 buckets that have open access permissions (public read or write access), which is a critical security vulnerability. The other options are either performance checks (A), fault tolerance checks (C), or cost optimization recommendations (D) that require a paid support plan.

</details>

---

### **Key Exam Tips & Tricks**

- **Free vs. Paid Tiers:** This is the most common way Trusted Advisor is tested. You MUST know which checks are part of the free core offering. Remember: **foundational Security checks and the Service Quotas check are FREE for everyone**. Cost Optimization, Performance, and Fault Tolerance checks require a paid (Business or Enterprise) support plan. If a question mentions a Basic support plan and asks about cost optimization, Trusted Advisor is likely a distractor.
- **Trusted Advisor vs. Other Services:** Be ready to differentiate Trusted Advisor from other monitoring and auditing services.
  - **Trusted Advisor:** Provides **high-level recommendations** based on AWS best practices. It tells you _what_ is wrong (e.g., "Port 22 is open to the world").
  - **AWS Config:** Provides **detailed configuration history** and compliance checking. It answers _"What is the exact configuration of this resource and has it changed?"_
  - **Amazon Inspector:** Finds **vulnerabilities _inside_** your EC2 instances/containers (e.g., "You are running an old version of Nginx with a known CVE").
  - **Amazon GuardDuty:** Detects **active threats and malicious behavior** (e.g., "Your instance is communicating with a known crypto-mining server").
- **Keyword Association:**
  - If you see "best practices," "recommendations," or a question about checking against the Well-Architected Framework, think **Trusted Advisor**.
  - If you see "approaching service limits" or "approaching quotas," think **Trusted Advisor**.
  - If you see "unrestricted access," "MFA on root," or "open S3 buckets," think **Trusted Advisor** (but also consider AWS Config, depending on the specific wording).
- **Proactive, Not Real-Time:** Trusted Advisor is a proactive guidance tool, but it's not a real-time alerting service like CloudWatch Alms. It refreshes its checks periodically. If a question requires an immediate, automated action based on a metric threshold, the answer is almost always a CloudWatch Alarm, not Trusted Advisor.
