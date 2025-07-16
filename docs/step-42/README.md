# Step 42: RDS Backups, Snapshots, Restores

## 🎯 Objective

- [x] Master: **RDS Backups, Snapshots, Restores**

## 📘 Notes

### **Deep Dive: RDS Backups, Snapshots, & Restores**

Amazon RDS provides robust, automated features to ensure your data is backed up and can be recovered in case of data corruption, user error, or disaster. There are two primary methods for backing up your RDS database: automated backups and manual snapshots.

---

### **1. Automated Backups & Point-in-Time Recovery (PITR)**

This is the default, managed backup feature of RDS, enabled by default when you create an instance.

- **How it Works:**
    1. **Daily Snapshots:** RDS automatically takes a full snapshot of your database instance every day during a configurable 30-minute backup window.
    2. **Transaction Logs:** In addition to the daily snapshot, RDS captures the transaction logs from your database (e.g., binary logs for MySQL, WAL for PostgreSQL) and uploads them to S3 every 5 minutes.
- **Backup Retention Period:** You can configure a retention period between **1 and 35 days**. RDS will store the daily snapshots and the transaction logs for this duration. Setting the retention period to 0 disables automated backups.
- **Point-in-Time Recovery (PITR):**
    - This is the most powerful feature of automated backups. The combination of the daily snapshot and the continuous transaction logs allows you to restore your database to **any specific second** within your retention period.
    - For example, if a developer runs a bad `DELETE` query at 2:15:30 PM, you can restore the database to its exact state at 2:15:29 PM, minimizing data loss.
- **Restore Process:** When you perform a restore (either full or point-in-time), RDS **always creates a brand new DB instance** with a new DNS endpoint. You cannot restore directly on top of an existing instance. You must update your application's connection string to point to the new instance's endpoint.
- **Location:** Automated backups are stored in S3 and are managed entirely by RDS.

---

### **2. Manual Snapshots** 📸

A manual snapshot is a user-initiated, full backup of your database instance at a specific point in time.

- **How it Works:** You trigger it manually via the AWS Console, CLI, or API at any time. RDS creates a complete storage volume snapshot of your instance.
- **Lifecycle:** Unlike automated backups, manual snapshots **do not have a retention period and are kept forever** until you explicitly delete them. They are not subject to the 1-35 day automated backup retention window.
- **Use Cases:**
    - **Long-Term Archival:** Creating a final snapshot of a database before decommissioning it, which you might need to keep for years for compliance.
    - **Creating Test Environments:** Taking a snapshot of a production database to create a new, identical instance for development or testing purposes.
- **Sharing and Copying:**
    - You can **copy** manual snapshots to other AWS Regions for disaster recovery or to create database instances in other regions.
    - You can **share** manual snapshots with other specific AWS accounts. The snapshots can be encrypted or unencrypted. If sharing an encrypted snapshot, you must also share the KMS key used to encrypt it.

---

### **Summary of Key Differences**

| Feature | Automated Backups | Manual Snapshots |
| --- | --- | --- |
| **Initiation** | Automatic (daily) | User-initiated (manual) |
| **Retention** | 1-35 days (configurable) | **Kept until manually deleted** |
| **Key Feature** | **Point-in-Time Recovery (PITR)** | Long-term retention, sharing |
| **Deletion** | Deleted when the RDS instance is deleted (unless a final snapshot is taken) | **Persist even after the original instance is deleted** |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A developer accidentally ran a script that corrupted a large number of records in a production RDS database at 3:20 PM. The database has automated backups enabled with a 7-day retention period. What is the BEST way to recover from this with minimal data loss?**

- A. Restore the latest daily snapshot, which was taken at 3:00 AM.
- B. Perform a Point-in-Time Recovery (PITR) to a new instance with the state of the database at 3:19 PM.
- C. Manually connect to the database and try to reverse the corrupted transactions.
- D. Promote the Multi-AZ standby to become the new primary.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is the exact purpose of Point-in-Time Recovery (PITR). Because RDS captures transaction logs every 5 minutes, you can restore the database to a new instance at any specific second within your retention period. Restoring to 3:19 PM, just before the corrupting script ran, will minimize data loss. A Multi-AZ failover (D) would not help, as the corrupted data would have been synchronously replicated to the standby.
</details>
---

**2. A company needs to keep a final backup of a project database for 7 years for compliance reasons after the project is complete and the RDS instance is terminated. What should be done?**

- A. Set the automated backup retention period to 2555 days (7 years).
- B. Create a final manual snapshot of the instance before deleting it.
- C. Rely on the final automated backup, which is kept indefinitely.
- D. Copy the database files to an S3 bucket using a Lambda function.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Automated backups are deleted when the RDS instance is deleted (unless you opt to create a final snapshot). The correct solution for long-term retention is to create a manual snapshot. Manual snapshots persist indefinitely until you explicitly delete them, making them perfect for archival.
</details>
---

**3. What is the maximum retention period for RDS automated backups?**

- A. 7 days
- B. 35 days
- C. 90 days
- D. There is no limit.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The configurable retention period for automated backups ranges from 1 day up to a maximum of 35 days. For retention beyond this, you must use manual snapshots.
</details>
---

**4. When you perform a restore from an RDS snapshot (either manual or automated), what is the result?**

- A. The existing database instance is overwritten with the data from the snapshot.
- B. A new RDS instance with a new DNS endpoint is created.
- C. The snapshot is attached to the existing instance as a read-only volume.
- D. The data is restored to an S3 bucket.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** An RDS restore operation always creates a new DB instance. This new instance will have its own unique DNS endpoint. To complete the recovery, you must update your application's configuration to point to this new endpoint. You cannot restore "in-place" over an existing instance.
</details>
---

**5. Which RDS feature depends on both daily snapshots and continuously backed-up transaction logs?**

- A. Manual Snapshots
- B. Multi-AZ Failover
- C. Point-in-Time Recovery (PITR)
- D. Read Replicas
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Point-in-Time Recovery works by first restoring the most recent daily snapshot that is before your desired recovery time, and then "replaying" the transaction logs from the time of the snapshot up to the specific second you requested. This process requires both components.
</details>
---

**6. An administrator deletes an RDS instance that has automated backups enabled. What happens to the automated backups?**

- A. They are retained for the configured retention period (e.g., 7 days).
- B. They are all immediately and permanently deleted.
- C. They are automatically converted into a single manual snapshot.
- D. By default, they are deleted, but AWS gives you the option to create a final manual snapshot upon deletion.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** By default, when you terminate an RDS instance, the automated backups are deleted. However, the console and API provide a crucial checkbox option to "Create final snapshot?" before deletion. If you select this, a final manual snapshot is created, which will be retained indefinitely.
</details>
---

**7. A development team in your company needs a copy of the production database for a testing cycle. The production database is encrypted. What is the most secure way to provide them with a copy?**

- A. Give them read-only access to the production database instance.
- B. Create a manual snapshot of the production database, and share the encrypted snapshot with the development AWS account.
- C. Take a snapshot, restore it to a new unencrypted instance, and give them the endpoint.
- D. Use AWS DataSync to copy the data.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The best practice is to isolate production and non-production environments. Taking a manual snapshot and sharing it allows the development team to restore it into their own isolated account. Since the snapshot is encrypted, you would also need to share the KMS key with their account, maintaining a secure posture.
</details>
---

**8. When are automated backups performed on a Multi-AZ RDS instance?**

- A. They are taken from the primary instance, which may cause a brief I/O suspension.
- B. They are taken from the standby instance to avoid impacting the performance of the primary instance.
- C. They are taken from both instances simultaneously.
- D. Backups are disabled in a Multi-AZ configuration.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** One of the secondary benefits of a Multi-AZ deployment is that maintenance activities, including backups, are performed on the standby replica. This prevents the I/O freeze that can occur during a snapshot, ensuring the primary instance's performance is not affected by the backup process.
</details>
---

**9. What happens to manual RDS snapshots when the source DB instance is deleted?**

- A. They are deleted along with the instance.
- B. They are retained for 30 days and then deleted.
- C. They are retained indefinitely until you manually delete them.
- D. They are automatically converted into automated backups.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Manual snapshots have a lifecycle that is completely independent of the source instance. They persist after the instance is deleted and will continue to incur storage costs until you take explicit action to delete them.
</details>
---

**10. You need to restore a database to a new instance in a different AWS Region for a disaster recovery test. What is the first step you must take?**

- A. Copy the automated backup files from S3 to the new region.
- B. Create a manual snapshot of the source database and copy the snapshot to the target region.
- C. Launch a new empty instance in the target region and use DataSync to copy the data.
- D. This is not possible; restores can only happen in the same region.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** RDS backups and snapshots are regional resources. To restore an instance in a different region, you must first have a snapshot available in that region. The process is to create a manual snapshot in the source region and then use the "Copy Snapshot" feature to create a copy of it in the disaster recovery region. You can then restore from that copied snapshot.
</details>
---

**11. What is the minimum configurable retention period for automated backups?**

- A. 0 days (disabled)
- B. 1 day
- C. 7 days
- D. 30 days
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** While you can set the retention to 0 to disable automated backups, the minimum enabled retention period you can configure is 1 day.
</details>
---

**12. When sharing an encrypted RDS snapshot with another AWS account, what additional step is required?**

- A. The other account must have an identical RDS instance running.
- B. You must also share the KMS key that was used to encrypt the snapshot with the other account.
- C. You must first decrypt the snapshot before sharing it.
- D. You must open a support ticket to authorize the cross-account share.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** For another account to be able to restore an encrypted snapshot, they need permission to use the key it was encrypted with. You must go into the KMS key policy of the encryption key and add the other AWS account as a principal, granting it permissions like kms:DescribeKey and kms:CreateGrant.
</details>
---

**13. Which of the following is a primary use case for taking a manual snapshot over relying on automated backups?**

- A. To perform a point-in-time recovery.
- B. To satisfy a requirement for long-term archival of a database state beyond 35 days.
- C. To ensure backups are taken every 5 minutes.
- D. To reduce database storage costs.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Automated backups are limited to a 35-day retention period. If you need to keep a backup for months or years for compliance or archival purposes, you must create a manual snapshot, as it will persist until you delete it.
</details>
---

**14. If you set the automated backup retention period to 0, what is the impact?**

- A. Automated backups are disabled, and Point-in-Time Recovery is not possible.
- B. Backups are taken, but only retained for 1 hour.
- C. Only transaction logs are backed up.
- D. This will automatically delete all existing manual snapshots.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** Setting the retention period to 0 is how you disable automated backups. This means no daily snapshots will be taken, and no transaction logs will be captured. As a result, you lose the ability to perform a Point-in-Time Recovery.
</details>
---

**15. Can you restore an RDS snapshot of a MySQL database to an Aurora cluster?**

- A. No, snapshots can only be restored to the same database engine.
- B. Yes, you can migrate a MySQL RDS snapshot directly to an Aurora MySQL-compatible cluster.
- C. No, you must first export the data to S3 and then import it into Aurora.
- D. Yes, but only if the original database was not encrypted.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** AWS provides a direct migration path for this. You can take a snapshot of your RDS for MySQL instance and then, in the "Restore Snapshot" wizard, choose to restore it as a new Amazon Aurora DB cluster, leveraging Aurora's MySQL compatibility.
</details>
---

**16. What does "Point-in-Time Recovery" allow you to do?**

- A. Restore a database to the time a specific manual snapshot was taken.
- B. Restore a database to any second within your configured backup retention period.
- C. Instantly recover a single corrupted table without restoring the entire database.
- D. Roll back a running database without creating a new instance.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** PITR provides very granular recovery. By using the combination of a daily snapshot and the 5-minute transaction logs, RDS can reconstruct the database state to a specific point in time (down to the second) within your retention window.
</details>
---

**17. Where are RDS automated backups and manual snapshots stored?**

- A. On an EC2 Instance Store.
- B. In an Amazon EFS file system.
- C. In Amazon S3.
- D. On the standby instance in a Multi-AZ configuration.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Both automated backups and manual snapshots are stored as snapshots in Amazon S3 for high durability. AWS manages the S3 storage for you, so you don't see the snapshots in your regular S3 buckets, but S3 is the underlying storage service.
</details>
---

**18. What is the impact on your application when you initiate a manual snapshot of a Single-AZ RDS instance?**

- A. There is no impact.
- B. Storage I/O may be briefly suspended while the snapshot is being taken, which can cause increased latency.
- C. The database instance is rebooted after the snapshot completes.
- D. The database is converted to read-only for the duration of the snapshot.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** During the snapshot creation process on a Single-AZ instance, there can be a brief I/O "freeze" as RDS ensures a consistent state of the volume. This can lead to a temporary increase in latency for your application. This is one reason why performing backups on the standby in a Multi-AZ setup is beneficial.
</details>
---

**19. What is a "final snapshot"?**

- A. The last automated backup taken before its retention period expires.
- B. A manual snapshot that you can optionally create when you delete an RDS instance.
- C. A snapshot that cannot be deleted.
- D. The first full backup taken when an instance is created.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A final snapshot is an option offered to you during the instance termination process. It is functionally a manual snapshot, meaning it will be retained indefinitely after the instance is gone, providing a final recovery point.
</details>
---

**20. A database snapshot was taken from an unencrypted RDS instance. Can you create an encrypted RDS instance from this snapshot?**

- A. No, the restored instance must have the same encryption state as the snapshot.
- B. Yes, during the restore process, you can choose to encrypt the new DB instance.
- C. No, you must first encrypt the snapshot before you can restore it as an encrypted instance.
- D. Yes, but only if you use the AWS CLI, not the console.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The restore process is very flexible. When you restore any snapshot (encrypted or unencrypted), you are given the option to enable encryption on the new DB instance that will be created. This is a common method for encrypting a previously unencrypted database.
</details>
---

**21. You have a database with a 10-day automated backup retention period. On Day 12, you realize you need data from Day 1. Can you recover it?**

- A. Yes, by using Point-in-Time Recovery.
- B. Yes, by restoring the automated snapshot from Day 1.
- C. No, the automated backup from Day 1 has been deleted.
- D. No, but you can request that AWS Support recover it for you.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The 10-day retention period is a hard limit. On Day 11, the automated backup from Day 1 would have been permanently deleted by RDS. On Day 12, the backup from Day 2 would be deleted. The data from Day 1 is gone and cannot be recovered.
</details>
---

**22. Which backup method's lifecycle is tied to the lifecycle of the DB instance itself?**

- A. Manual Snapshots
- B. Automated Backups
- C. Both Manual and Automated
- D. Neither
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Automated Backups exist only as long as the instance exists (and for its configured retention period). If you delete the instance without taking a final snapshot, the automated backups are also deleted. Manual snapshots are independent and persist after the instance is deleted.
</details>