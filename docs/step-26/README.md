# Step 26: Automating EBS Snapshots

## 🎯 Objective
- [x] Master: **Automating EBS Snapshots**

### **Deep Dive: Automating EBS Snapshots**

Manually creating EBS snapshots is feasible for a few volumes, but it's not a scalable or reliable strategy for a production environment. To ensure regular, consistent backups and manage their retention, you need automation. AWS provides several tools for this, with Amazon Data Lifecycle Manager (DLM) being the primary native service.

---

### **1. Amazon Data Lifecycle Manager (DLM)** 🗓️

Amazon DLM is a managed service that provides a simple, automated way to back up the data stored on your EBS volumes. It allows you to create and manage the lifecycle of your EBS snapshots through policies.

- **How it works:** You create a **Lifecycle Policy** that defines the "what, when, and how" of your snapshot management.
    - **Targeting:** You specify which EBS volumes to back up by targeting them with **resource tags**. For example, you can create a policy that targets all volumes with the tag `Backup:Daily`. This is powerful because any new instance launched with a volume that has this tag will be automatically included in the backup plan.
    - **Schedules:** You define one or more schedules within the policy. Each schedule specifies the **creation frequency** (e.g., every 12 hours, daily, weekly), a start time, and a retention rule.
    - **Retention:** For each schedule, you define how the snapshots should be retained. This can be based on:
        - **Count:** Keep the last `N` snapshots (e.g., keep the last 7 daily snapshots).
        - **Age:** Keep snapshots for `X` days, weeks, months, or years (e.g., keep weekly snapshots for 4 weeks).
- **Key Features:**
    - **Crash-Consistent Snapshots:** By default, DLM takes a snapshot of a volume as-is. For volumes attached to a running instance, this is a "crash-consistent" snapshot.
    - **Cross-Region Copy:** Policies can be configured to automatically copy snapshots to other AWS Regions for disaster recovery purposes. You can set a separate retention period for the copied snapshots.
    - **Cost-Effective:** It's a free service; you only pay for the storage cost of the EBS snapshots it creates.
    - **Integration:** Works seamlessly with AWS Organizations to manage policies across multiple accounts.

### **2. Other Automation Methods**

- **Amazon EventBridge (formerly CloudWatch Events) + AWS Lambda:**
    - **How it works:** This is a more customizable, "serverless" approach. You create an EventBridge rule that runs on a schedule (e.g., using a cron expression). This rule's target is an AWS Lambda function. The Lambda function contains code (e.g., written in Python with the Boto3 SDK) that executes the `ec2:CreateSnapshot` API call.
    - **Use Case:** This method is used when you need more complex logic than what DLM provides. For example, if you need to quiesce a database (pause I/O) before taking the snapshot to ensure it's **application-consistent**, not just crash-consistent. The Lambda function could run SSM commands to freeze the database, create the snapshot, and then thaw the database.
    - **Complexity:** This approach is more powerful but also more complex to set up and maintain than DLM.
- **AWS Backup:**
    - **How it works:** AWS Backup is a fully managed, centralized backup service that simplifies protecting data across multiple AWS services, including EBS, EFS, RDS, DynamoDB, and more. You create a **backup plan** that defines frequency and retention, and you assign resources to it by tag or ARN.
    - **Use Case:** When you want a single pane of glass to manage backups for many different types of AWS resources, not just EBS volumes. It provides a more holistic backup management strategy.

For the SAA-C03 exam, **DLM is the primary and most direct answer** for automating EBS snapshots specifically.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to implement a policy to automatically create a snapshot of all EBS volumes tagged with `Environment:Production` every 24 hours and retain these snapshots for 7 days. What is the simplest and most direct AWS service to achieve this?**

- A. A scheduled cron job on an EC2 instance running a script.
- B. Amazon Data Lifecycle Manager (DLM).
- C. AWS CloudTrail.
- D. AWS Config.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon Data Lifecycle Manager (DLM) is the purpose-built service for this exact scenario. You can create a lifecycle policy that targets resources based on tags (Environment:Production), define a creation schedule (every 24 hours), and set a retention period (keep for 7 days).
</details>
    

---

**2. How does Amazon Data Lifecycle Manager (DLM) identify which EBS volumes to include in a backup policy?**

- A. By volume ID.
- B. By the instance the volume is attached to.
- C. By resource tags on the EBS volumes.
- D. By Availability Zone.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** DLM uses resource tags to identify target volumes. This is a very flexible and scalable approach. You can, for example, tag all your database volumes with App:Database, and the DLM policy will automatically find and back them up, including any new database volumes created with that tag in the future.
</details>
    

---

**3. A company has a compliance requirement to store copies of their EBS snapshots in a different AWS Region for disaster recovery. How can this be automated?**

- A. This must be done manually by copying snapshots after they are created.
- B. Use an S3 Cross-Region Replication rule on the bucket where snapshots are stored.
- C. Configure a cross-region copy action within the Amazon DLM lifecycle policy.
- D. Use AWS Backup with a cross-region vault.

<details>
<summary>View Answer</summary>
<br>

**Answer: C or D**

**Explanation:** Both C and D are valid automation methods.
    
- **DLM (C)** has a built-in feature to automatically copy snapshots created by a policy to one or more other regions. You can even set a different retention period for the copied snapshots.
- AWS Backup (D) also fully supports cross-region backups as part of a backup plan.
    
For a question focused purely on EBS snapshot automation, DLM is the most direct answer.
</details>
        

---

**4. A solutions architect needs to ensure that database snapshots are "application-consistent," meaning the database's in-memory data is flushed to disk before the snapshot is taken. DLM's default snapshot is only "crash-consistent." What is the best way to achieve this automation?**

- A. Use Amazon DLM with the application-consistent option enabled.
- B. Use Amazon EventBridge to trigger an AWS Lambda function on a schedule. The function uses SSM to run commands to freeze the DB, takes the snapshot, and then thaws the DB.
- C. Manually stop the EC2 instance every night before the snapshot is taken.
- D. Use an st1 volume type, which automatically flushes data to disk.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This scenario requires custom actions beyond what DLM natively provides. The combination of EventBridge (for scheduling) and a Lambda function is the standard serverless pattern for this. The Lambda function can orchestrate the entire process: use AWS Systems Manager (SSM) Run Command to freeze I/O on the database, call the CreateSnapshot API, and then run another SSM command to unfreeze the database.
</details>
    

---

**5. What type of retention rules can be configured in an Amazon DLM policy?**

- A. Only count-based retention (e.g., keep the last 5 snapshots).
- B. Only age-based retention (e.g., keep snapshots for 14 days).
- C. Both count-based and age-based retention.
- D. Only a single retention period for all snapshots created by the policy.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** DLM lifecycle policies are flexible. For each schedule you create (e.g., daily, weekly, monthly), you can define a retention rule based on either a specific count of snapshots to keep or the age of the snapshots.
</details>
    

---

**6. A DLM policy is created with a schedule to take snapshots daily at 02:00 UTC and retain them for 10 days. On the 12th day, how many snapshots created by this policy will exist?**

- A. 12
- B. 11
- C. 10
- D. 2

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The policy retains snapshots for 10 days. On Day 1, a snapshot is created. On Day 2, another is created (2 total). This continues until Day 10 (10 total). On Day 11, Snapshot 11 is created, and Snapshot 1 (which is now 10 days old) is deleted. On Day 12, Snapshot 12 is created, and Snapshot 2 is deleted. The total count remains at 10.
</details>
    

---

**7. A company wants to use a single service to manage the backup and retention policies for its EBS volumes, RDS databases, and EFS file systems. Which service is designed for this purpose?**

- A. Amazon Data Lifecycle Manager (DLM)
- B. AWS Backup
- C. AWS Systems Manager
- D. Amazon S3 Lifecycle Policies

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** While DLM is excellent for EBS specifically, AWS Backup is the centralized, managed service designed to handle backups across multiple AWS services. It provides a single pane of glass to configure, manage, and audit backup plans for EBS, RDS, DynamoDB, EFS, and more.
</details>
    

---

**8. What is a "crash-consistent" snapshot?**

- A. A snapshot that is guaranteed to be corrupted.
- B. A snapshot taken of a volume while the instance is running, equivalent to pulling the power cord from a server. Data in memory or in-flight is not captured.
- C. A snapshot where the application has been properly quiesced before the snapshot was taken.
- D. A snapshot that can only be restored by crashing the EC2 instance.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A crash-consistent snapshot captures all the data that has been written to the EBS volume at a single point in time. It's like taking a picture of the disk state as if the server suddenly lost power. For many applications this is fine, but for databases, it can lead to an inconsistent state that requires recovery procedures upon restoration.
</details>
    

---

**9. A DLM policy is created to target volumes with the tag `Backup:Yes`. An administrator launches a new EC2 instance with an EBS volume but forgets to add the tag. What happens regarding the backup of this new volume?**

- A. The DLM policy will automatically find and back up the new volume.
- B. The volume will not be backed up by this DLM policy until the tag `Backup:Yes` is added to it.
- C. The launch of the instance will fail because it does not meet the policy requirements.
- D. DLM will send a notification asking the administrator to add the tag.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** DLM relies entirely on the specified tags to identify its target resources. If a volume does not have the tag defined in the policy's target, it will be ignored by the policy. The automation is dependent on consistent and accurate tagging.
</details>
    

---

**10. When a DLM policy copies a snapshot to another region, can the retention period for the copy be different from the source snapshot's retention period?**

- A. No, the retention period must be identical in both regions.
- B. Yes, the cross-region copy rule allows you to specify a separate retention period for the copied snapshots.
- C. No, copied snapshots are never deleted automatically.
- D. Yes, but it can only be shorter than the source retention period.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The cross-region copy configuration within a DLM policy is highly flexible. It allows you to define a completely independent retention rule (either count-based or age-based) for the snapshots in the destination region. For example, you could keep 7 daily snapshots in the source region but keep 4 weekly snapshots in the DR region.
</details>
    

---

**11. What are the two main components of an Amazon DLM Lifecycle Policy? (Select TWO)**

- A. Target resources (defined by tags)
- B. An IAM role for permissions
- C. A schedule with creation and retention rules
- D. A Lambda function for custom actions
- E. An SNS topic for notifications

<details>
<summary>View Answer</summary>
<br>

**Answers: A and C**

**Explanation:** The fundamental building blocks of a DLM policy are the target resources (which volumes to back up, identified by tags) and one or more schedules that define the backup frequency and retention rules. The IAM role (B) is also required for DLM to have permission to act on your behalf, but A and C define the what and when of the policy itself.
</details>
    

---

**12. To automate snapshots for an EC2 instance, you need to grant permissions to the automation service. How does Amazon DLM get the permissions it needs to create and delete snapshots on your behalf?**

- A. It uses the permissions of the IAM user who created the policy.
- B. It uses a pre-defined, service-linked IAM role that you grant it permission to use.
- C. It has root-level access to the account by default.
- D. It requires you to provide IAM user access keys.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Like many AWS services, DLM uses a service-linked role. This is a special type of IAM role that is pre-defined by the service and linked to it. When you first use DLM, you grant it permission to create this role in your account. The role contains the specific permissions DLM needs (like ec2:CreateSnapshot, ec2:DeleteSnapshot, etc.) and can only be assumed by the DLM service principal.
</details>
    

---

**13. A company wants to keep snapshots based on a GFS (Grandfather-Father-Son) rotation scheme: keep 7 daily, 4 weekly, and 12 monthly snapshots. How can this be configured in a single DLM policy?**

- A. This is not possible; a separate policy is needed for each time frame.
- B. By creating a single schedule with three different retention rules.
- C. By creating three separate schedules (one for daily, one for weekly, one for monthly) within the same policy.
- D. By using a Lambda function, as DLM does not support GFS.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A single DLM policy can contain up to four schedules. To implement a GFS scheme, you would create three schedules in your policy: 1) A daily schedule with a count-based retention of 7. 2) A weekly schedule (e.g., runs every Sunday) with a count-based retention of 4. 3) A monthly schedule (e.g., runs on the 1st of the month) with a count-based retention of 12.
</details>
    

---

**14. What is the main benefit of using Amazon DLM over a custom cron script running on an EC2 instance?**

- A. DLM can take snapshots faster.
- B. DLM is a fully managed, serverless solution that reduces operational overhead and provides centralized management.
- C. A cron script can target volumes using tags, but DLM cannot.
- D. DLM is more expensive but provides better reporting.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Using a custom script on an EC2 instance means you are responsible for managing the instance, patching it, ensuring it's highly available, and maintaining the script. DLM is a fully managed service, meaning there is no infrastructure for you to manage. It's more reliable, scalable, and easier to maintain than a self-managed script.
</details>
    

---

**15. If a snapshot creation fails within a DLM policy execution, what happens?**

- A. The entire DLM policy is disabled permanently.
- B. DLM will automatically retry the snapshot creation.
- C. An email is sent to the AWS account root user.
- D. The associated EBS volume is terminated to prevent data corruption.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** DLM has built-in retry logic. If a transient error causes a snapshot creation to fail, the service will attempt the operation again. You can monitor the success and failure of policy executions in Amazon EventBridge and CloudWatch Logs.
</details>
    

---

**16. Which service allows you to create a "backup vault" with a Vault Lock policy to enforce WORM (Write-Once, Read-Many) principles for your EBS snapshots?**

- A. Amazon S3
- B. Amazon Data Lifecycle Manager (DLM)
- C. AWS Backup
- D. AWS IAM

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS Backup introduces the concept of a backup vault. A key feature is Backup Vault Lock, which allows you to deploy an immutable, WORM-locked policy on your vault. This can prevent anyone (including the root user) from deleting backups before their retention period expires, which is a powerful tool for compliance.
</details>
    

---

**17. A lifecycle policy in DLM is configured to retain 5 snapshots. When the 6th snapshot is successfully created, what happens to the oldest snapshot?**

- A. It is moved to S3 Glacier Deep Archive.
- B. It is marked as "expired" but kept for 30 days.
- C. It is immediately and permanently deleted.
- D. Its retention count is increased to 6 automatically.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** When using count-based retention, DLM strictly adheres to the number specified. As soon as the 6th snapshot is created, making the total count exceed 5, DLM will delete the oldest (6th oldest) snapshot to bring the total back down to the target of 5.
</details>
    

---

**18. You need to ensure a snapshot is only taken during a specific maintenance window (e.g., Sunday at 3:00 AM). Which automation tool is best suited for this precise scheduling?**

- A. Amazon Data Lifecycle Manager (DLM)
- B. AWS Health
- C. AWS Trusted Advisor
- D. A manual process.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** DLM schedules allow for precise timing. You can configure a schedule to run daily, weekly, or monthly at a specific time of day (in UTC), making it easy to align your backup operations with your defined maintenance windows.
</details>
    

---

**19. To create an "application-consistent" snapshot of a database on EC2, what step is often required just before creating the snapshot?**

- A. Rebooting the EC2 instance.
- B. Detaching the EBS volume.
- C. Temporarily pausing or freezing I/O operations to the database.
- D. Increasing the provisioned IOPS of the volume.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** To get a perfect, application-consistent snapshot, you need to ensure the application isn't in the middle of writing data. The standard procedure is to issue a command to the application (like a database) to flush all its in-memory data to disk and temporarily pause any new writes (a process called quiescing or freezing). The snapshot is taken during this brief pause, after which the application is unfrozen.
</details>
    

---

**20. A new EBS volume is created with the tag `Department:Finance`. An existing DLM policy targets volumes with the tag `Backup:True`. Will the new volume be backed up?**

- A. Yes, all new volumes are automatically backed up.
- B. No, because the volume's tags do not match the policy's target tags.
- C. Yes, because DLM backs up volumes based on department.
- D. Only if the volume is attached to an instance.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The automation is only as good as the tagging. The DLM policy is explicitly configured to look for the tag Backup:True. Since the new volume does not have this tag, it will not be included in the policy's execution.
</details>
    

---

**21. What is the most significant benefit of using resource tags to define the scope of a DLM policy?**

- A. It's the only method supported by DLM.
- B. It ensures that any new resource created with the correct tag is automatically included in the backup plan without manual intervention.
- C. It reduces the cost of the snapshots.
- D. It makes the snapshots application-consistent.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Tag-based targeting makes the backup strategy dynamic and scalable. You don't need to update your DLM policy every time you create a new instance or volume. As long as your provisioning process applies the correct tags, the new resources are automatically protected, reducing administrative overhead and the risk of missed backups.
</details>
    

---

**22. Which of the following is NOT a feature of Amazon Data Lifecycle Manager?**

- A. Ability to create snapshots on a recurring schedule.
- B. Ability to copy snapshots to other regions.
- C. Ability to restore a volume directly from a snapshot.
- D. Ability to define retention policies based on age or count.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** DLM is a service for automating the creation, copying, and deletion of snapshots. It does not handle the restoration process. Restoring a volume from a snapshot is a manual action you perform through the EC2 console or API, or a task you would script as part of a disaster recovery plan.
</details>