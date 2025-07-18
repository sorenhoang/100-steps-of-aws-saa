# Step 69: Backup & Restore Services

## 🎯 Objective

- [x] Master: **Backup & Restore Services**

## 📘 Notes

### **Architectural Deep Dive: Backup & Restore Services**

Backup and restore is a foundational strategy for both operational recovery (e.g., recovering from accidental deletion) and disaster recovery (e.g., recovering from a regional outage). AWS provides two main approaches: native backup features within individual services and AWS Backup, a centralized service to manage backups across multiple services.

---

### **1. Native Service Backups**

These are the backup features built directly into specific AWS services.

- **EBS Snapshots:** Point-in-time backups of your EBS volumes. They are incremental and stored in S3. You can automate their creation and retention using **Amazon Data Lifecycle Manager (DLM)**.
- **RDS Snapshots:** Backups of your relational databases.
  - **Automated Backups:** Daily snapshots plus transaction logs, enabling Point-in-Time Recovery (PITR) for up to 35 days.
  - **Manual Snapshots:** User-initiated backups that are kept until you manually delete them, perfect for long-term archival.
- **S3 Versioning & Cross-Region Replication (CRR):**
  - **Versioning:** Protects against accidental overwrites or deletes by keeping a history of all versions of an object. This is for operational recovery.
  - **CRR:** Automatically and asynchronously copies objects to a bucket in another region. This is a key tool for disaster recovery of your S3 data.

---

### **2. AWS Backup** 🗄️

AWS Backup is a **fully managed, centralized backup service** that simplifies the management of backups across multiple AWS services. Instead of managing backups service-by-service, you can do it from a single pane of glass.

- **How it works:**
  1. **Backup Plan:** You create a backup plan, which is a policy that defines your backup strategy. It includes:
     - **Backup Rule:** Defines the frequency (e.g., daily) and the backup window.
     - **Lifecycle Rule:** Defines when to transition backups to cold storage (e.g., move to Glacier after 90 days) and when they expire.
     - **Backup Vault:** The encrypted location where the backups are stored.
  2. **Resource Assignment:** You assign AWS resources to the plan, typically by using **tags**. For example, you can assign all resources with the tag `Backup:Daily` to your daily backup plan.
- **Key Features:**
  - **Centralized Management:** Configure backup policies for multiple services in one place.
  - **Supported Services:** Supports EC2 (via EBS snapshots), RDS, Aurora, DynamoDB, EFS, FSx, and more.
  - **Compliance & Governance:** Use **AWS Backup Vault Lock** to create immutable, WORM (Write-Once, Read-Many) backups to meet compliance requirements.
  - **Cross-Region & Cross-Account:** Can manage and copy backups to different regions and accounts for DR and security.

---

### **SAA-C03 Style Scenario Questions**

**1. A company has a wide range of AWS resources, including EC2 instances, RDS databases, and EFS file systems. They need a single, unified solution to manage and automate backup and retention policies for all these resources. Which service is designed for this?**

- A. Amazon Data Lifecycle Manager (DLM)
- B. AWS Backup
- C. Taking manual snapshots for each service.
- D. AWS Systems Manager.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the primary use case for AWS Backup. It provides a centralized, policy-driven service to manage backups across a wide variety of supported AWS services. DLM (A) is specifically for automating EBS snapshots, not RDS or EFS.

</details>

---

**2. To protect against accidental deletion of a critical `config.json` file in an S3 bucket, which feature should be enabled?**

- A. S3 Lifecycle Policies
- B. S3 Versioning
- C. S3 Cross-Region Replication
- D. AWS Backup

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 Versioning is the native feature for protecting against accidental overwrites and deletions at the object level. When enabled, deleting an object simply creates a "delete marker," and the previous versions of the object remain recoverable.

</details>

---

**3. What is the primary function of Amazon Data Lifecycle Manager (DLM)?**

- A. To manage the lifecycle of IAM user credentials.
- B. To automate the creation, retention, and deletion of EBS snapshots.
- C. To centrally manage backups for both RDS and EFS.
- D. To manage the lifecycle of EC2 instances from launch to termination.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** DLM is a targeted automation service specifically for EBS snapshots. It allows you to create sophisticated, tag-based policies to manage the entire lifecycle of your volume backups.

</details>

---

**4. A company needs to store its database backups for 10 years to meet compliance regulations. The database is running on Amazon RDS. What is the most appropriate backup method?**

- A. Configure the RDS automated backup retention period for 10 years.
- B. Take a manual snapshot of the RDS instance.
- C. Enable S3 Cross-Region Replication on the backup bucket.
- D. Use AWS Backup with a 10-year retention rule.

<details>
<summary>View Answer</summary>

**Answer: B or D**

**Explanation:** Both B and D are valid and correct solutions.

- A **manual RDS snapshot (B)** is kept until you explicitly delete it, making it perfect for long-term archival beyond the 35-day limit of automated backups.
- AWS Backup (D) is the centralized way to achieve this. You can create a backup plan with a lifecycle rule to retain backups for 10 years in a backup vault.

For an exam question, the choice would depend on whether the scenario implies a centralized or a single-service solution. Both are architecturally sound.

</details>

---

**5. Which AWS Backup feature allows you to enforce WORM (Write-Once, Read-Many) policies to make your backups immutable for a specified retention period?**

- A. Backup Plan Lifecycle Rules
- B. AWS Backup Vault Lock
- C. S3 Object Lock
- D. IAM permission boundaries

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS Backup Vault Lock is the compliance feature for AWS Backup. Once a vault is locked in compliance mode, it becomes immutable, preventing anyone (including the root user) from deleting backups before their retention period expires or from modifying the vault lock settings.

</details>

---

### **Key Exam Tips & Tricks**

- **Centralized vs. Native:** The exam will test your ability to choose the right tool for the scope of the problem.
  - If the question is about automating backups for **EBS volumes only**, the most direct answer is usually **Amazon Data Lifecycle Manager (DLM)**.
  - If the question is about backing up resources from **multiple different services** (e.g., RDS, EFS, and DynamoDB) and managing them with a **single policy**, the answer is always **AWS Backup**.
- **Operational Recovery vs. Disaster Recovery:**
  - **Operational Recovery** (e.g., "a developer deleted a file"): The answer is often a native feature like **S3 Versioning** or an RDS Point-in-Time Recovery.
  - **Disaster Recovery** (e.g., "a region has failed"): The answer involves cross-region features. For S3, it's **Cross-Region Replication (CRR)**. For EBS/RDS, it involves **copying snapshots** to another region. For AWS Backup, it's a **cross-region copy** rule in the backup plan.
- **Keyword Association:**
  - **"Centralized backup," "multiple services," "backup plan," "backup vault"**: Think **AWS Backup**.
  - **"Automate EBS snapshots," "tag-based EBS backup"**: Think **Amazon Data Lifecycle Manager (DLM)**.
  - **"Undelete an object," "protect from overwrites"**: Think **S3 Versioning**.
  - **"Point-in-Time Recovery," "restore to a specific second"**: Think **RDS Automated Backups**.
  - **"Long-term archive," "backup kept after instance deleted"**: Think **RDS Manual Snapshots**.
  - **"Immutable backups," "WORM," "compliance hold"**: Think **AWS Backup Vault Lock** (or S3 Object Lock if the context is only S3).
