# Step 61: CloudWatch Metrics, Alarms & Dashboards

## 🎯 Objective

- [x] Master: **CloudWatch Metrics, Alarms & Dashboards**

## 📘 Notes

### **Deep Dive: CloudWatch Metrics, Alarms & Dashboards**

Amazon CloudWatch is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), and IT managers. It provides you with data and actionable insights to monitor your applications, respond to system-wide performan1ce changes, optimize resource utilization, and get a unified view of operational health.

---

### **1. CloudWatch Metrics** 📊

A metric is the fundamental concept in CloudWatch. It represents a time-ordered set of data points—a variable to monitor over time.

- **Namespaces:** Metrics are grouped into namespaces. A namespace is a container for metrics. For example, all metrics produced by EC2 are under the `AWS/EC2` namespace, while RDS metrics are under `AWS/RDS`.
- **Dimensions:** A dimension is a name/value pair that is part of the identity of a metric. It allows you to uniquely identify a specific resource. For example, to see the CPU utilization of a *specific* EC2 instance, you would look at the `CPUUtilization` metric within the `AWS/EC2` namespace where the dimension `InstanceId` equals `i-1234567890abcdef0`.
- **Standard vs. Detailed Monitoring (for EC2):**
    - **Standard Monitoring:** Enabled by default. EC2 sends metric data to CloudWatch in **5-minute intervals**. This is free of charge.
    - **Detailed Monitoring:** An optional, paid feature. When enabled, EC2 sends metric data in **1-minute intervals**. This provides more granular data, which is useful for faster-reacting Auto Scaling policies or more detailed troubleshooting.
- **Custom Metrics:** You are not limited to the metrics AWS provides. You can publish your own custom metrics to CloudWatch from your applications using the `PutMetricData` API call. This is useful for monitoring application-level performance indicators, such as the number of user sign-ups or the processing time of a specific business transaction.

**Important Note:** By default, EC2 metrics measure the performance of the **hypervisor**, not the operating system. For example, `CPUUtilization` is the host's view of the vCPU. To get OS-level metrics like **memory utilization, disk space, or swap usage**, you must install the **CloudWatch Agent** on your EC2 instances.

---

### **2. CloudWatch Alarms** 🔔

A CloudWatch Alarm watches a single CloudWatch metric over a specified time period and performs one or more automated actions when the value of the metric breaches a threshold.

- **Alarm States:** An alarm has three possible states:
    - **OK:** The metric is within the defined threshold.
    - **ALARM:** The metric has breached the threshold.
    - **INSUFFICIENT_DATA:** Not enough data points are available to determine the alarm's state (e.g., the monitored resource was just started or stopped).
- **Configuration:** You define:
    - The metric to watch.
    - The threshold value.
    - The evaluation period and number of periods (e.g., "if CPU utilization is greater than 80% for 2 consecutive periods of 5 minutes").
- **Actions:** When an alarm changes state (e.g., from `OK` to `ALARM`), it can trigger an action. Common actions include:
    - **SNS Notification:** Send a message to an SNS topic, which can then fan out to email, SMS, or SQS queues. This is the most common way to alert human operators.
    - **EC2 Auto Scaling:** Trigger an Auto Scaling policy to launch or terminate instances.
    - **EC2 Action:** Stop, terminate, reboot, or recover an EC2 instance. The "recover" action is useful for automatically recovering an instance if it fails a system status check.

---

### **3. CloudWatch Dashboards** 📈

A CloudWatch Dashboard is a customizable home page in the CloudWatch console where you can monitor your resources in a single view, even those spread across different regions.

- **What it does:** You can create dashboards that display graphs of metrics, the current state of alarms, and other visualizations.
- **Use Case:** Creating a single pane of glass to view the operational health of a specific application or your entire AWS environment. For example, you can create a dashboard that shows the CPU utilization of all your web servers, the latency of your Application Load Balancer, the number of healthy hosts, and the status of your database's read replicas, all on one screen. Dashboards can be made public or shared.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A solutions architect needs to monitor the memory utilization of a fleet of EC2 instances. When they go to the CloudWatch console, they can see CPU Utilization metrics but cannot find any metrics for memory. Why is this?**

- A. Memory utilization can only be monitored by AWS Trusted Advisor.
- B. The IAM role attached to the instances is missing permissions to send memory metrics.
- C. By default, EC2 does not send OS-level metrics like memory usage to CloudWatch. The CloudWatch Agent must be installed on the instances to collect this data.
- D. Detailed Monitoring must be enabled for memory metrics to appear.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a key concept. Standard EC2 metrics are from the hypervisor level. To get metrics from within the guest operating system, such as memory usage, disk space utilization, or process counts, you must install and configure the CloudWatch Agent on the EC2 instances.
</details>
    

---

**2. An operations team wants to be notified via email whenever the average CPU utilization of a critical production EC2 instance exceeds 90% for a continuous period of 10 minutes. What is the correct sequence of services to set this up?**

- A. Create a CloudWatch Alarm on the CPUUtilization metric, with an action that sends a notification to an SNS topic. Subscribe an email address to that SNS topic.
- B. Configure VPC Flow Logs to send a notification to SNS when CPU is high.
- C. Create a CloudWatch Dashboard that triggers an email when the graph goes above 90%.
- D. Use AWS CloudTrail to monitor the CPU and trigger a Lambda function to send an email.
<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** This describes the standard alerting pattern. A CloudWatch Alarm is created to watch the metric. The action for the alarm is to publish a message to an SNS topic. You then create a subscription for that topic, with the protocol set to "Email," to send the notification to the desired recipients.
</details>
    

---

**3. What is the difference between Standard Monitoring and Detailed Monitoring for an EC2 instance?**

- A. Standard Monitoring is free and sends metrics every 5 minutes, while Detailed Monitoring has a cost and sends metrics every 1 minute.
- B. Standard Monitoring includes memory usage, while Detailed Monitoring does not.
- C. Standard Monitoring sends metrics every 1 minute, while Detailed Monitoring sends metrics every 5 minutes.
- D. Detailed Monitoring is required to create CloudWatch Alarms.
<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** This is the core distinction. Standard Monitoring provides a data point every 5 minutes at no cost. Detailed Monitoring provides more granular, 1-minute data points, but it is a paid feature.
</details>
    

---

**4. A CloudWatch Alarm is configured to monitor an EC2 instance. The alarm's state is currently "INSUFFICIENT_DATA." What does this mean?**

- A. The instance has failed its status check.
- B. The metric being monitored has exceeded its threshold.
- C. There is not enough data available for the metric over the evaluation period to determine the alarm's state.
- D. The IAM role for CloudWatch is configured incorrectly.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The INSUFFICIENT_DATA state means the alarm cannot make a clear decision. This often happens when an instance has just been launched and hasn't had time to report metrics yet, or when an instance has been stopped, or if the metric itself is not being published for some reason.
</details>
    

---

**5. A company has a critical EC2 instance. They want to configure an automated action to reboot the instance if it fails its EC2 System Status Check. Which service can be configured to do this?**

- A. AWS Trusted Advisor
- B. AWS Health
- C. A CloudWatch Alarm
- D. An EC2 Launch Template
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A CloudWatch Alarm can be configured to watch the StatusCheckFailed_System metric. You can then configure an EC2 Action on that alarm to automatically reboot or recover the instance when the alarm enters the ALARM state.
</details>
    

---

**6. What is the primary purpose of a CloudWatch Dashboard?**

- A. To store raw log data for long-term archival.
- B. To provide a customizable, single-pane-of-glass view of metrics and alarms from multiple sources and regions.
- C. To define the scaling policies for an Auto Scaling group.
- D. To trigger Lambda functions based on a schedule.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A CloudWatch Dashboard is a visualization tool. Its purpose is to create a consolidated view of the health and performance of your application by displaying graphs of key metrics and the status of important alarms all in one place.
</details>
    

---

**7. An application developer wants to monitor the number of successful user logins, which is a metric specific to their application logic. How can they get this data into CloudWatch?**

- A. This is not possible, as CloudWatch only monitors AWS infrastructure metrics.
- B. By writing the login count to a log file and having CloudWatch Logs parse it.
- C. By instrumenting the application code to make a `PutMetricData` API call to publish a custom metric to CloudWatch.
- D. By enabling Detailed Monitoring on the EC2 instance running the application.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** To monitor business or application-level metrics, you must create custom metrics. The standard way to do this is to use the AWS SDK within your application to call the PutMetricData API. This allows you to publish any numerical data you want under a custom namespace.
</details>
    

---

**8. In CloudWatch, what is a "namespace"?**

- A. A container or grouping for related metrics.
- B. A specific value of a metric at a point in time.
- C. A name/value pair that identifies a specific resource.
- D. The unit of a metric (e.g., Percent, Bytes).
<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** A namespace is a top-level container that groups metrics from a single service or application. For example, all EC2 metrics are in the AWS/EC2 namespace, and all RDS metrics are in the AWS/RDS namespace.
</details>
    

---

**9. A CloudWatch alarm is configured with a period of 5 minutes and an evaluation period of 3. What does this mean?**

- A. The alarm will check the metric every 3 minutes for 5 consecutive periods.
- B. The alarm will go into an ALARM state if the threshold is breached for 3 consecutive 5-minute periods (a total of 15 minutes).
- C. The alarm will check the metric every 5 minutes for 3 seconds.
- D. The alarm will go into an ALARM state if the threshold is breached for 5 consecutive 3-minute periods (a total of 15 minutes).
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Period is the length of time over which the metric is aggregated (5 minutes). The Evaluation Period is the number of most recent periods to check. So this configuration means "look at the last three 5-minute data points, and if all three are above the threshold, trigger the alarm."
</details>
    

---

**10. Which action can a CloudWatch Alarm directly trigger?**

- A. Create an S3 bucket.
- B. Stop, terminate, or reboot an EC2 instance.
- C. Create a new IAM user.
- D. Run a command on an EC2 instance via SSH.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** CloudWatch Alarms have a specific set of built-in actions. These include sending a notification to SNS, triggering an Auto Scaling action, and performing an EC2 action (stop, terminate, reboot, or recover). To perform more complex actions like running a command, you would typically have the alarm trigger a Lambda function which then executes the command.
</details>
    

---

**11. What is a "dimension" in a CloudWatch metric?**

- A. The time at which the data point was recorded.
- B. A key-value pair that helps you uniquely identify a metric.
- C. The unit of measurement for the metric.
- D. The namespace the metric belongs to.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A dimension adds identifying information to a metric. For example, the CPUUtilization metric is generic. The dimension InstanceId=i-12345 makes it specific to a single EC2 instance. You can have multiple dimensions to further refine the metric's identity.
</details>
    

---

**12. When Detailed Monitoring is enabled for an EC2 instance, what is the frequency of metric data points sent to CloudWatch?**

- A. 10 seconds
- B. 30 seconds
- C. 1 minute
- D. 5 minutes
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Detailed Monitoring provides metric data at a 1-minute frequency, offering a more granular view of performance compared to the 5-minute interval of Standard Monitoring.
</details>
    

---

**13. By default, what kind of data does the `CPUUtilization` metric for an EC2 instance represent?**

- A. The CPU utilization of the physical host server.
- B. The average CPU utilization across all instances in the same security group.
- C. The percentage of allocated EC2 compute units that are currently in use on the hypervisor level.
- D. The CPU utilization as reported by the guest operating system's kernel.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Standard EC2 metrics are collected from the hypervisor, the software layer that manages the virtual machines. CPUUtilization represents the percentage of the vCPU's allocated time that is being used by the guest OS. It is not a view from inside the OS.
</details>
    

---

**14. An application's performance depends on the number of items in an SQS queue. The team wants to create an alarm that triggers if the `ApproximateNumberOfMessagesVisible` metric exceeds 1000. What namespace would this metric be found in?**

- A. `AWS/EC2`
- B. `AWS/Lambda`
- C. `AWS/SQS`
- D. `Custom/SQS`
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Each AWS service that publishes metrics to CloudWatch does so under its own unique namespace. For metrics related to SQS queues, the namespace is AWS/SQS.
</details>
    

---

**15. Can a single CloudWatch Dashboard display graphs for metrics from different AWS Regions?**

- A. No, a dashboard is limited to a single region.
- B. Yes, a dashboard can display data from multiple regions in a single view.
- C. Only if the regions are in the same country.
- D. Only if you use the CloudWatch API, not the console.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Yes, one of the key benefits of CloudWatch Dashboards is their ability to provide a cross-region, consolidated view of your application's health. You can add widgets for metrics from us-east-1 and eu-west-1 onto the same dashboard.
</details>
    

---

**16. You want to create an alarm that triggers when your estimated AWS bill exceeds $500. Where must you first enable a setting for this metric to be available in CloudWatch?**

- A. In the IAM console.
- B. In the AWS Budgets console.
- C. In the Billing and Cost Management console preferences.
- D. In the CloudTrail console.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Billing metrics are not sent to CloudWatch by default. You must first go to the Billing and Cost Management console, navigate to "Billing preferences," and enable the "Receive Billing Alerts" option. This action will start publishing the EstimatedCharges metric to CloudWatch (in the us-east-1 region), which you can then create an alarm on.
</details>
    

---

**17. The CloudWatch Agent can be used to collect which of the following? (Select TWO)**

- A. VPC Flow Logs
- B. OS-level metrics like memory and disk usage from EC2 instances.
- C. Custom application log files from EC2 instances.
- D. The results of AWS Trusted Advisor checks.
- E. The number of objects in an S3 bucket.
<details>
<summary>View Answer</summary>
<br>

**Answer: B and C**

**Explanation:** The CloudWatch Agent has two main functions: 1) Collect system-level metrics from inside the OS of your EC2 instances (and on-premises servers) and send them to CloudWatch as custom metrics (B). 2) Collect and stream log files from the instances to CloudWatch Logs (C).
</details>
    

---

**18. What is the most common and flexible action to take from a CloudWatch Alarm if you need to perform a complex, custom response?**

- A. Trigger an EC2 recover action.
- B. Send a notification to an SNS topic, which then triggers a Lambda function.
- C. Send an email directly from the alarm.
- D. Reboot the instance.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** For any complex or custom action, the standard pattern is to have the alarm publish a message to an SNS topic. You then subscribe an AWS Lambda function to that topic. This gives you the full power of a programming language in the Lambda function to perform any custom logic you need, such as calling other APIs, creating a support ticket, or updating a database.
</details>
    

---

**19. What is the default retention period for metric data in CloudWatch?**

- A. 14 days
- B. 30 days
- C. 1 year
- D. 15 months
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** CloudWatch stores metric data for 15 months. It automatically rolls up the data's granularity over time: high-resolution data is kept for a short period, then it's aggregated to 1-minute data, then to 5-minute data, and so on, to optimize storage.
</details>
    

---

**20. A metric has dimensions `InstanceId=i-123` and `Environment=Prod`. How would you view the metric for all environments, not just `Prod`?**

- A. You must create a new metric without the `Environment` dimension.
- B. In the CloudWatch console, you can search for the metric and view it aggregated across all dimensions.
- C. This is not possible; you must view each dimension combination separately.
- D. By removing the `Prod` dimension from the metric definition.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** CloudWatch allows you to aggregate metrics. You can choose to view a metric with all of its dimensions specified, or you can remove a dimension from your view to see an aggregated graph (e.g., the average CPUUtilization across all InstanceIds).
</details>
    

---

**21. What is the primary difference between a CloudWatch metric and a CloudWatch log event?**

- A. A metric is a numerical data point, while a log event is a text record of a specific event.
- B. Metrics are free, while logs have a cost.
- C. Metrics have a longer retention period than logs.
- D. You can create alarms on logs, but not on metrics.
<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** This is the fundamental difference. A metric is a time-series of a numerical value (e.g., CPU is 45%). A log event is a text-based record of something that happened at a specific time (e.g., "ERROR: User failed to log in"). While you can extract metrics from logs, they are different types of data.
</details>
    

---

**22. Which CloudWatch feature would you use to create a graph showing the correlation between the number of users on your website and the average CPU load of your server fleet?**

- A. CloudWatch Alarms
- B. CloudWatch Logs
- C. CloudWatch Dashboards
- D. CloudWatch ServiceLens
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A CloudWatch Dashboard is the perfect tool for this. You can create a dashboard and add two separate graph widgets. One widget would graph your custom metric for "user count," and the second widget would graph the average CPUUtilization of your EC2 fleet. Displaying them together makes it easy to visually correlate the two trends.
</details>