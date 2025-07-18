# Step 64: Cost Explorer & AWS Budgets

## 🎯 Objective

- [x] Master: **Cost Explorer & AWS Budgets**

## 📘 Notes

### **Architectural Deep Dive: Cost Explorer & AWS Budgets**

As a Solutions Architect, you are responsible not only for designing functional and performant systems but also for ensuring they are cost-effective. AWS provides two primary tools to help you understand, manage, and control your cloud spending: **AWS Cost Explorer** for analyzing the past and present, and **AWS Budgets** for controlling the future. They are central to the **Cost Optimization** pillar of the AWS Well-Architected Framework.

The core architectural principle here is **Cloud Financial Management**. Instead of getting a surprise bill at the end of the month, you use these tools to create a feedback loop: **Analyze -> Monitor -> Control -> Optimize**.

- **Analyze (Cost Explorer):** You start by understanding your spending patterns. Where is the money going? Which service, account, or project is the most expensive?
- **Monitor & Control (AWS Budgets):** Once you understand your baseline, you set up guardrails. You create budgets to get alerted _before_ you overspend and, in some cases, take automatic action to stop spending.
- **Optimize:** The insights from both tools drive architectural decisions. For example, if Cost Explorer shows high, consistent EC2 spending, you might decide to purchase Savings Plans. If a NAT Gateway is a top cost, you might implement VPC Endpoints.

---

### **1. AWS Cost Explorer: The Analytical Tool** 🔍

Cost Explorer is an interactive tool that lets you visualize, understand, and manage your AWS costs and usage over time.

- **Core Function:** It's a **retrospective and current analysis** tool. It helps you answer questions like:
  - "What was my most expensive service last month?"
  - "Show me a graph of my EC2 spending over the last 6 months, grouped by instance type."
  - "How much have I spent on resources with the `Project:Phoenix` tag?"
- **Key Features:**
  - **Filtering and Grouping:** You can filter your costs by numerous dimensions, including AWS Service, Linked Account, Region, Instance Type, and critically, **Cost Allocation Tags**.
  - **Forecasting:** Cost Explorer uses historical data to provide a forecast of what your spending is likely to be by the end of the month.
  - **Reports:** Provides pre-configured reports, such as monthly costs by service or daily spend. You can also save your custom views as new reports.
  - **Data Granularity:** Can provide data at a daily or monthly level. For more detailed analysis, you can enable hourly and resource-level granularity.

---

### **2. AWS Budgets: The Proactive Control Tool** 💰🔔

AWS Budgets is a tool that allows you to set custom budgets and receive alerts when your costs or usage **exceed (or are forecasted to exceed)** your budgeted amount.

- **Core Function:** It's a **proactive monitoring and alerting** tool. It helps you answer the question:
  - "Notify me when I am _about to_ go over my monthly budget for my development account."
- **Key Features:**
  - **Budget Types:** You can create budgets based on:
    - **Cost:** The total dollar amount.
    - **Usage:** Non-monetary usage, like "Gigabytes of S3 storage."
    - **Reservation/Savings Plans:** Monitor the utilization or coverage of your commitments.
  - **Alerting:** This is the key feature. An alert can be triggered when:
    - **Actual** cost exceeds a threshold (e.g., 80% of the budget).
    - **Forecasted** cost is predicted to exceed a threshold. This is a powerful feature that gives you early warning.
  - **Budget Actions:** You can configure a budget to trigger an automated action when a threshold is met. This moves from simple notification to active control. Actions can include:
    - Applying a restrictive IAM policy to a user or group (e.g., a "deny-new-ec2-launches" policy).
    - Stopping specific EC2 or RDS instances.

---

### **SAA-C03 Style Scenario Questions**

**1. A finance manager wants a monthly report that breaks down the AWS costs for three different projects: Phoenix, Griffin, and Dragon. All project resources are running in the same AWS account. How can a solutions architect provide this data?**

- A. Create three separate AWS accounts, one for each project.
- B. Use AWS Budgets to create a separate budget for each project.
- C. Implement a tagging strategy where all resources are tagged with their project name, then use AWS Cost Explorer to filter and group costs by tag.
- D. Use VPC Flow Logs to determine which project is using which resources.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is a primary use case for Cost Allocation Tags and AWS Cost Explorer. By applying a consistent tag (e.g., Project:Phoenix) to all resources for a given project, you can then use Cost Explorer's filtering and grouping capabilities to generate a detailed cost report specifically for that tag.

</details>

---

**2. A company has given its development team a strict monthly spending limit. The manager wants to be notified via email as soon as AWS predicts that the team's account is on track to exceed its budget. Which service and feature should be used?**

- A. An AWS Cost Explorer daily report.
- B. An AWS Budget with an alert based on forecasted costs.
- C. A CloudWatch alarm on the `EstimatedCharges` metric.
- D. AWS Trusted Advisor's cost optimization checks.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The key word is "predicts" (or forecasted). Only AWS Budgets can create alerts based on forecasted spending. This gives the manager a proactive warning before the budget is actually exceeded. A CloudWatch alarm (C) can only trigger on actual accumulated charges.

</details>

---

**3. After reviewing a report in AWS Cost Explorer, a solutions architect discovers an unexpectedly high cost associated with a NAT Gateway. What is the next logical step to optimize this cost?**

- A. Delete the NAT Gateway.
- B. Analyze the traffic to see if resources are communicating with AWS services (like S3 or DynamoDB) that could be accessed via a VPC Endpoint instead.
- C. Purchase a Savings Plan for the NAT Gateway.
- D. Replace the NAT Gateway with a cheaper NAT Instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The report from Cost Explorer provides the "what" (high NAT Gateway cost). The next step is to find the "why". A common reason for high NAT Gateway costs is traffic going to other AWS services within the same region. The best architectural optimization is to implement VPC Endpoints for those services, which routes the traffic over the private AWS network, bypassing the NAT Gateway and eliminating its data processing fees.

</details>

---

**4. A company wants to automatically prevent developers from launching new EC2 instances in a sandbox account once the account's monthly budget of $500 is exceeded. What is the most direct way to achieve this?**

- A. Create an AWS Budget with a Budget Action that attaches a restrictive IAM policy.
- B. Create a CloudWatch alarm that triggers a Lambda function to delete new instances.
- C. Use an SCP to deny `ec2:RunInstances` for the entire month.
- D. Manually revoke developer permissions when the budget is exceeded.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** AWS Budget Actions allow you to automatically apply restrictive IAM policies when budget thresholds are exceeded. This is the most direct and automated way to prevent new EC2 instance launches when the budget limit is reached, providing proactive cost control without manual intervention.

</details>

---

**5. A manager asks for a visualization of the company's AWS spending over the past 12 months, with a breakdown by service. Which tool is designed for this type of historical analysis and visualization?**

- A. The AWS Billing Dashboard
- B. AWS Budgets
- C. AWS Cost Explorer
- D. AWS Trusted Advisor

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Cost Explorer is specifically designed for visualizing, analyzing, and understanding historical AWS costs and usage over time. It provides interactive charts and graphs that can break down spending by service, account, region, and other dimensions, making it the ideal tool for this type of historical cost analysis and visualization.

</details>

---

### **Key Exam Tips & Tricks**

- **Keyword Association:** This is one of the most important areas for keyword association on the exam.
  - If the question asks about **visualizing, analyzing, discovering, or reporting on past costs**, the answer is almost always **Cost Explorer**.
  - If the question asks about **alerting, notifying, or taking action** when costs are **forecasted to exceed** or **actually exceed** a limit, the answer is always **AWS Budgets**.
- **Forecasted vs. Actual:** Pay close attention to this wording. A CloudWatch billing alarm can only trigger on _actual_ accumulated costs. If the question uses words like "predict," "projected," or "forecasted," it's a clear signal that the answer is **AWS Budgets**.
- **Cost Optimization Loop:** Remember the pattern: Cost Explorer gives you the insight (the "what"), and this insight drives the architectural change (the "how"). For example, Cost Explorer shows high data transfer cost -> you realize it's inter-AZ -> you co-locate the instances. Cost Explorer is the _first step_ in analysis, not the optimization itself.
- **Tags are Key:** Any time a question asks about breaking down costs by project, department, team, or any other business logic within a single account, the answer will always involve **Cost Allocation Tags**. You must first tag your resources, then activate the tags in the Billing console, and then you can filter by them in Cost Explorer or create budgets based on them.
- **Distractor: Trusted Advisor:** Trusted Advisor has a "Cost Optimization" category and can give you valuable recommendations (like identifying idle RDS instances). However, it is not an _analytical_ tool like Cost Explorer, nor is it a _budgeting_ tool like AWS Budgets. It provides a list of recommendations, not a way to explore your data or set alerts. Don't choose it if the question asks to analyze or create a budget.

---

### **Key Exam Tips & Tricks**

- **Keyword Association:** This is one of the most important areas for keyword association on the exam.
  - If the question asks about **visualizing, analyzing, discovering, or reporting on past costs**, the answer is almost always **Cost Explorer**.
  - If the question asks about **alerting, notifying, or taking action** when costs are **forecasted to exceed** or **actually exceed** a limit, the answer is always **AWS Budgets**.
- **Forecasted vs. Actual:** Pay close attention to this wording. A CloudWatch billing alarm can only trigger on _actual_ accumulated costs. If the question uses words like "predict," "projected," or "forecasted," it's a clear signal that the answer is **AWS Budgets**.
- **Cost Optimization Loop:** Remember the pattern: Cost Explorer gives you the insight (the "what"), and this insight drives the architectural change (the "how"). For example, Cost Explorer shows high data transfer cost -> you realize it's inter-AZ -> you co-locate the instances. Cost Explorer is the _first step_ in analysis, not the optimization itself.
- **Tags are Key:** Any time a question asks about breaking down costs by project, department, team, or any other business logic within a single account, the answer will always involve **Cost Allocation Tags**. You must first tag your resources, then activate the tags in the Billing console, and then you can filter by them in Cost Explorer or create budgets based on them.
- **Distractor: Trusted Advisor:** Trusted Advisor has a "Cost Optimization" category and can give you valuable recommendations (like identifying idle RDS instances). However, it is not an _analytical_ tool like Cost Explorer, nor is it a _budgeting_ tool like AWS Budgets. It provides a list of recommendations, not a way to explore your data or set alerts. Don't choose it if the question asks to analyze or create a budget.
