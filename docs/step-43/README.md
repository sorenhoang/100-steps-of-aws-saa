# Step 43: Read Replicas & Cross‑Region DR

## 🎯 Objective

- [x] Master: **Read Replicas & Cross‑Region DR**

## 📘 Notes

### **Deep Dive: Read Replicas & Cross-Region DR**

While Multi-AZ deployments provide high availability, they don't help with performance scaling. For read-heavy applications or for disaster recovery (DR) in different geographic regions, you need Read Replicas.

---

### **1. RDS Read Replicas** 📖

A Read Replica is a **read-only copy** of a primary database instance. Its main purpose is to offload read queries from your primary database to improve performance and scale your application's read capacity.

- **How it Works:**
    - Replication is **asynchronous**. The primary database writes to its transaction logs, and these logs are then sent to and replayed on the Read Replica. This means there can be a small "replica lag."
    - You can create up to 5 Read Replicas for a given source instance (this limit can vary slightly for different engines).
    - Each Read Replica has its **own unique DNS endpoint**. Your application must be architected to direct write queries to the primary endpoint and read queries to the replica endpoint(s).
- **Key Use Cases:**
    1. **Scaling Read-Heavy Workloads:** If your application has many more reads than writes (e.g., a popular blog or a reporting dashboard), you can direct all read traffic to one or more Read Replicas. This frees up the primary instance to handle only write traffic, improving overall performance.
    2. **Business Intelligence (BI) / Reporting:** You can run long, complex analytics or reporting queries on a Read Replica without impacting the performance of your live, production (primary) database.
- **Important Characteristics:**
    - **Can be in a different AZ or Region:** You can create a Read Replica in the same AZ, a different AZ, or a completely different AWS Region than the primary.
    - **Can be Promoted:** You can **promote** a Read Replica to become its own standalone, primary database instance. When this happens, the replication link is permanently broken. This is a key part of the disaster recovery strategy.
    - **Can have its own Multi-AZ configuration:** You can create a Read Replica and then enable Multi-AZ on that replica. This creates a standby for the replica itself, which is a common pattern for critical reporting databases.

---

### **2. Cross-Region Disaster Recovery (DR)** 🌎

Using Read Replicas is the primary method for creating a disaster recovery (DR) solution for your RDS database in another geographic region.

- **The Architecture:**
    1. **Primary Site:** Your primary database is running in your main region (e.g., `us-east-1`), often in a Multi-AZ configuration for high availability.
    2. **DR Site:** You create a **Cross-Region Read Replica** in your DR region (e.g., `eu-west-1`).
    3. **Replication:** AWS handles the secure, encrypted replication of data over its backbone network from the primary region to the DR region.
    4. **Disaster Scenario:** If the entire `us-east-1` region becomes unavailable, your application goes down.
    5. **Recovery Process (Manual):**
        - You **promote** the Cross-Region Read Replica in `eu-west-1` to become a new, standalone primary database. This breaks the replication link.
        - You update your application's DNS records (e.g., in Route 53) to point to the DNS endpoint of the newly promoted database in `eu-west-1`.
        - Your application is now live again, running out of the DR region.
- **Recovery Point Objective (RPO) & Recovery Time Objective (RTO):**
    - **RPO (Data Loss):** Because replication is asynchronous, there can be a small replica lag. Your RPO is the amount of data you could lose, which is equal to the replica lag (can be seconds to minutes).
    - **RTO (Downtime):** Your RTO is the time it takes you to perform the manual steps of promoting the replica and updating DNS. This is typically measured in minutes.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. An online reporting dashboard is causing performance degradation on a company's primary production OLTP database. The company needs to offload these intensive read queries without impacting the production database. What is the best solution?**

- A. Enable Multi-AZ on the primary database.
- B. Create a Read Replica of the database and direct all reporting queries to the replica's endpoint.
- C. Increase the instance size of the primary database.
- D. Take a manual snapshot and restore it to a new instance for reporting.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is the primary use case for a Read Replica. It creates a read-only copy of the database, allowing you to divert all the heavy, long-running reporting queries to the replica. This completely isolates the reporting workload from the primary production database, which can then dedicate its resources to handling transactional writes.
</details>
---

**2. What is the key difference in the replication method between an RDS Multi-AZ standby instance and an RDS Read Replica?**

- A. Multi-AZ is synchronous, while Read Replica replication is asynchronous.
- B. Multi-AZ is asynchronous, while Read Replica replication is synchronous.
- C. Both use synchronous replication.
- D. Both use asynchronous replication.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** This is a critical distinction. Multi-AZ uses synchronous replication to ensure the standby is an exact, up-to-the-second copy for high availability with minimal data loss. Read Replicas use asynchronous replication, which is more performant for the primary but introduces a small "replica lag."
</details>
---

**3. A company needs to implement a disaster recovery plan for its critical RDS for PostgreSQL database located in the `us-west-2` region. In the event of a regional outage, they must be able to fail over to the `eu-central-1` region. What is the recommended approach?**

- A. Enable Multi-AZ in the `us-west-2` region.
- B. Create a Cross-Region Read Replica in the `eu-central-1` region.
- C. Set up a daily automated snapshot copy to the `eu-central-1` region.
- D. Use AWS Database Migration Service (DMS) for continuous replication.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The standard and most effective pattern for cross-region database DR is to create a Cross-Region Read Replica. This provides a continuously updated, warm standby database in the DR region. In a disaster, this replica can be quickly promoted to a full, standalone master database to recover the application.
</details>
---

**4. Can you directly write data to an RDS Read Replica?**

- A. Yes, it functions as a fully writable copy.
- B. No, a Read Replica is read-only by default.
- C. Yes, but only if the primary instance is unavailable.
- D. Only for Amazon Aurora Read Replicas.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** As the name implies, a Read Replica is, by default, a read-only copy of the master database. Its purpose is to serve read queries. If you need to write to it, you must first promote it to be a standalone instance, which breaks the replication link.
</details>
---

**5. What happens when you promote a Read Replica to become a standalone database instance?**

- A. The original primary instance is automatically deleted.
- B. The replication link between the old primary and the replica is broken, and the replica becomes a new, independent, writable database.
- C. The replica continues to receive updates from the primary but also accepts write traffic.
- D. A new DNS endpoint must be created manually.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Promoting a Read Replica is a one-way operation. It severs the asynchronous replication connection, turning the read-only copy into a fully independent, read/write database instance. It retains its own unique DNS endpoint.
</details>
---

**6. A company has a primary RDS instance and a Read Replica in the same region. If the primary instance fails, will RDS automatically fail over to the Read Replica?**

- A. Yes, the failover is automatic and takes less than a minute.
- B. No, failing over to a Read Replica is a manual process.
- C. Yes, but only if the Read Replica is in a different Availability Zone.
- D. No, Read Replicas cannot be used for failover.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Automatic failover is a feature of Multi-AZ deployments, not Read Replicas. If a primary instance fails, you must manually promote the Read Replica to become the new primary and update your application's connection strings to point to the newly promoted instance's endpoint.
</details>
---

**7. A read-heavy application is using an RDS Read Replica to scale. Users are complaining that they sometimes don't see the data immediately after it has been written. What is the most likely cause?**

- A. The security group is blocking traffic to the Read Replica.
- B. The application is experiencing "replica lag" due to the asynchronous nature of the replication.
- C. The Read Replica is in a different region.
- D. The TTL on the DNS record for the replica is too high.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Because Read Replica replication is asynchronous, there is always a small delay (from milliseconds to seconds or more) between when data is written to the primary and when it appears on the replica. This delay is known as replica lag. Applications must be designed to tolerate this eventual consistency.
</details>
---

**8. Which RDS feature is designed for high availability, and which is designed for performance scaling?**

- A. High Availability: Read Replicas; Scaling: Multi-AZ
- B. High Availability: Manual Snapshots; Scaling: Automated Backups
- C. High Availability: Multi-AZ; Scaling: Read Replicas
- D. High Availability: Read Replicas; Scaling: Manual Snapshots
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This is the fundamental distinction: Multi-AZ is for High Availability (HA) and Disaster Recovery (DR). Read Replicas are for scaling read performance.
</details>
---

**9. Can you create a Read Replica of another Read Replica?**

- A. No, this is not supported.
- B. Yes, this is supported for certain database engines like MySQL and PostgreSQL.
- C. Yes, but only if they are in the same Availability Zone.
- D. Only Amazon Aurora supports this feature.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Yes, for engines like RDS for MySQL and PostgreSQL, you can create a second-tier Read Replica from an existing first-tier Read Replica. This can be useful for taking some of the replication load off the primary instance.
</details>
---

**10. To create a Cross-Region Read Replica, what must first be enabled on the source RDS instance?**

- A. Multi-AZ Deployment.
- B. Performance Insights.
- C. Automated Backups (with a backup retention period greater than 0).
- D. IAM Database Authentication.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Cross-Region Read Replicas are created from a snapshot of the source instance, and the replication is kept in sync using transaction logs. This mechanism requires that automated backups be enabled on the source instance, as this is the feature that enables both snapshotting and transaction log capture.
</details>
---

**11. A company has a Cross-Region Read Replica for disaster recovery. What is the Recovery Point Objective (RPO) primarily dependent on?**

- A. The time it takes to promote the replica.
- B. The amount of cross-region replica lag at the time of the disaster.
- C. The speed of the DNS propagation.
- D. The size of the database.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Recovery Point Objective (RPO) refers to the amount of data loss you can tolerate. In an asynchronous replication setup, the potential data loss is equal to the replica lag—the transactions that were committed on the primary but had not yet been replayed on the replica at the moment of the outage.
</details>
---

**12. When you create a Read Replica, how does your application connect to it?**

- A. It connects to the primary instance's endpoint, which automatically routes read queries.
- B. The Read Replica shares the same DNS endpoint as the primary.
- C. The Read Replica is given its own unique DNS endpoint.
- D. You must connect using the replica's private IP address.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Each Read Replica is a separate database instance and gets its own distinct DNS endpoint. Your application's data access layer must be configured to be aware of both the primary endpoint (for writes) and the replica endpoint(s) (for reads).
</details>
---

**13. Which Amazon RDS engine provides up to 15 Read Replicas that all share the same underlying storage volume as the primary instance, resulting in minimal replica lag?**

- A. Amazon RDS for MySQL
- B. Amazon RDS for PostgreSQL
- C. Amazon Aurora
- D. Amazon RDS for SQL Server
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This describes the unique architecture of Amazon Aurora. Aurora Replicas are not traditional replicas; they are read-only endpoints that all point to the same shared storage volume as the writer instance. Because there is no log shipping, replica lag is typically in the sub-10 millisecond range, and you can have up to 15 of them.
</details>
---

**14. A Read Replica can be created from which of the following? (Select TWO)**

- A. A Single-AZ DB instance.
- B. A Multi-AZ DB instance.
- C. An existing Read Replica.
- D. A manual DB snapshot.
- E. An automated backup file in S3.
<details>
<summary>View Answer</summary>

**Answers:** A, B, and C

**Explanation:** You can create a Read Replica from a Single-AZ instance or a Multi-AZ instance (the source for the replication will be the primary node). As mentioned earlier, for some engines, you can also create a replica of a replica (C). You cannot create a continuously replicating Read Replica from a static snapshot (D) or backup file (E).
</details>
---

**15. A company performs a disaster recovery failover by promoting its Cross-Region Read Replica. The original primary region is now back online. What is the state of the old primary database?**

- A. It automatically becomes a Read Replica of the newly promoted database.
- B. It is automatically terminated.
- C. It remains as a standalone, independent database instance that is now out of sync.
- D. It automatically tries to fail back and become the primary again.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The failover process is not automatically reversible. After you promote the DR replica, the old primary instance in the original region is simply an independent database that is now "behind" the new primary. To fail back, you would need to manually set up replication in the reverse direction or use another method to resynchronize the data.
</details>
---

**16. What is required to encrypt a Read Replica if the primary source instance is unencrypted?**

- A. This is not possible; the replica must have the same encryption state as the primary.
- B. You must first encrypt the primary instance, which automatically encrypts the replica.
- C. You can enable encryption when you create the Read Replica, even if the source is unencrypted.
- D. You must encrypt the replica after it has been created and synchronized.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** This is a key rule. A Read Replica of an unencrypted DB instance must also be unencrypted. A Read Replica of an encrypted instance must also be encrypted. You cannot change the encryption state during the replica creation process. To create an encrypted copy, you would need to take a snapshot, copy the snapshot while enabling encryption, and then restore from the new encrypted snapshot.
</details>
---

**17. What is the main purpose of a Multi-AZ deployment?**

- A. To scale read traffic.
- B. To provide a low-cost backup solution.
- C. To provide high availability through synchronous replication and automatic failover.
- D. To enable cross-region disaster recovery.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The sole purpose of a Multi-AZ deployment is High Availability. It is designed to protect a database from an infrastructure failure (like an instance or AZ outage) by providing a fully automated, rapid failover to a synchronous standby.
</details>
---

**18. What is the Recovery Time Objective (RTO) dependent on in a cross-region DR scenario using a Read Replica?**

- A. The amount of data in the database.
- B. The network latency between the regions.
- C. The time it takes to manually promote the replica and update DNS.
- D. The size of the transaction logs.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Recovery Time Objective (RTO) refers to the duration of the downtime. In this DR scenario, the downtime is the time it takes for an administrator to perform the necessary manual actions: promoting the Read Replica to a standalone instance and then re-pointing the application's DNS to the new database endpoint.
</details>
---

**19. You have a Multi-AZ primary instance with a Read Replica. If the primary instance's AZ fails, what happens?**

- A. RDS automatically fails over to the Read Replica.
- B. RDS automatically fails over to the Multi-AZ standby instance. The Read Replica will then automatically reconfigure itself to replicate from the newly promoted primary.
- C. RDS fails over to both the standby and the Read Replica simultaneously.
- D. The Read Replica is promoted, and the Multi-AZ standby is deleted.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The Multi-AZ failover takes precedence. RDS will automatically fail over to the synchronous standby instance. After this failover is complete, RDS will then automatically update the replication source for the Read Replica so that it begins replicating from the new primary instance.
</details>
---

**20. Can you create a Read Replica for an Amazon Aurora database?**

- A. No, Aurora does not support Read Replicas.
- B. Yes, and you can have up to 15 Aurora Replicas that share the same underlying storage volume.
- C. Yes, but only one is allowed per primary instance.
- D. Yes, but they must be in the same Availability Zone as the writer instance.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Amazon Aurora has a unique implementation of Read Replicas (often called Aurora Replicas). You can have up to 15 of them, and they are highly performant because they all read from the same shared storage volume as the writer instance, resulting in very low replica lag.
</details>
---

**21. A reporting tool connects to a Read Replica. The replica lag is higher than expected. What is a potential cause?**

- A. The primary database is experiencing a very high volume of write operations.
- B. There is not enough storage allocated to the Read Replica.
- C. The Read Replica is in the same Availability Zone as the primary.
- D. The security group is throttling the connection.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** Replica lag occurs when the replica cannot keep up with the changes being made on the primary. A sudden spike in write activity on the primary database will generate a large volume of transaction logs that need to be sent to and applied by the replica, which can cause the lag to increase.
</details>
---

**22. Which two statements accurately describe RDS Read Replicas? (Select TWO)**

- A. They use synchronous replication.
- B. They are primarily used for scaling read traffic.
- C. The standby instance is not accessible.
- D. Each replica has its own DNS endpoint.
- E. Failover to a replica is automatic.
<details>
<summary>View Answer</summary>

**Answers:** B and D

**Explanation:** The two defining characteristics are that they are used for scaling read traffic (B) and that each one is a distinct database instance with its own unique DNS endpoint (D) that your application must connect to. They use asynchronous replication (A is false), the instance is accessible (C refers to a Multi-AZ standby), and failover is manual (E is false).
</details>