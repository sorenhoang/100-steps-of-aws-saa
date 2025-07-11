# Step 15: EBS vs Instance Store

## 🎯 Objective

- [x] Master: **EBS vs Instance Store**

## 📘 Notes

Of course. This is a fundamental EC2 topic that deals with the two primary storage options for your instances. Understanding when to use each is key. Let's begin the lecture on **"EBS vs. Instance Store."**

### **Deep Dive: EBS vs. Instance Store**

When you launch an EC2 instance, you need to decide where its operating system and data will be stored. AWS provides two main options: Elastic Block Store (EBS) and EC2 Instance Store. They differ primarily in their **persistence**, performance, and cost, making them suitable for very different workloads.

---

### **1. Elastic Block Store (EBS)** 💾

EBS provides **persistent block-level storage volumes** for use with EC2 instances. Think of an EBS volume as a network-attached hard drive that you can attach to your instance.

- **Persistence:** **Durable and Persistent**. The data on an EBS volume persists independently of the life of the EC2 instance. If you stop, hibernate, or terminate the instance, the attached EBS volume and its data remain. You can detach it from one instance and reattach it to another.
- **Networking:** It is a network-attached drive. This means there can be a small amount of latency for I/O operations compared to a physically attached drive.
- **Root Volume:** Most AMIs are "EBS-backed," meaning the root volume (where the OS is installed) is an EBS volume. This allows the instance to be stopped and started.
- **Snapshots:** You can take point-in-time snapshots of EBS volumes, which are stored durably in Amazon S3. This is the primary mechanism for backing up your data and creating new custom AMIs.
- **Encryption:** All EBS volume types support encryption at rest, which is handled transparently with minimal performance impact.

### **EBS Volume Types**

- **Solid State Drives (SSD):** Optimized for transactional workloads involving frequent read/write operations with small I/O size (high IOPS).
  - **General Purpose SSD (gp2/gp3):** The default and most common volume type. `gp3` is the latest generation, allowing you to provision IOPS and throughput independently of storage size. It provides a great balance of price and performance for a wide variety of workloads like boot volumes, virtual desktops, and most applications.
  - **Provisioned IOPS SSD (io1/io2):** Designed for critical, I/O-intensive database and application workloads that require sustained, high IOPS performance. You specify the exact number of IOPS you need, and AWS provisions it. `io2 Block Express` offers the highest performance and durability.
- **Hard Disk Drives (HDD):** Optimized for large streaming workloads requiring high throughput (MB/s).
  - **Throughput Optimized HDD (st1):** Low-cost magnetic storage designed for frequently accessed, throughput-intensive workloads with large datasets, such as big data processing (Kafka, Hadoop), data warehouses, and log processing.
  - **Cold HDD (sc1):** The lowest-cost HDD storage. Designed for less frequently accessed data where low storage cost is paramount.

---

### **2. EC2 Instance Store** ⚡

An EC2 Instance Store provides **temporary, ephemeral block-level storage**. It consists of one or more storage volumes that are **physically attached** to the host computer where your EC2 instance is running.

- **Persistence:** **Ephemeral (Non-Persistent)**. This is the most important characteristic. The data on an instance store **is lost** when the underlying EC2 instance is **stopped, hibernated, or terminated**. It also is lost if the underlying physical host fails. The data only survives a reboot.
- **Performance:** Because the storage is physically attached to the host, it offers extremely high, low-latency I/O performance. This is especially true for Storage Optimized instances (like the I family) that use NVMe SSDs for their instance stores.
- **Root Volume:** It is possible to have an "Instance Store-backed" AMI, but this is less common. An instance launched from such an AMI cannot be stopped; it can only be running or terminated.
- **Cost:** The cost of the instance store is included in the hourly price of the EC2 instance itself. There are no separate storage charges.

---

### **Summary of Key Differences**

| Feature         | Elastic Block Store (EBS)                            | EC2 Instance Store                                   |
| --------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Persistence** | **Persistent** (survives instance termination)       | **Ephemeral** (data is deleted on stop/termination)  |
| **Analogy**     | Network Attached Storage (NAS) / SAN                 | Direct Attached Storage (DAS) / Internal Hard Drive  |
| **Best For**    | Databases, boot volumes, persistent application data | Caches, buffers, scratch space, temporary data       |
| **Backup**      | EBS Snapshots to S3                                  | Must replicate data elsewhere for durability         |
| **Stop/Start**  | Can be stopped                                       | Cannot be stopped (if root volume is instance store) |
| **Performance** | Good network-attached performance                    | Extremely high, low-latency performance              |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to store a critical relational database on an EC2 instance. The data must persist even if the EC2 instance is stopped or terminated. Which storage option should be used for the database files?**

- A. An EC2 Instance Store
- B. An Amazon EBS Volume
- C. An Amazon S3 Bucket
- D. A RAM disk in the instance's memory

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The key requirement is persistence. Amazon EBS is a persistent block storage service, meaning the data exists independently of the EC2 instance's lifecycle. An Instance Store (A) is ephemeral and would lose all data on termination. S3 (C) is object storage, not block storage suitable for a database's root files, and a RAM disk (D) is even more volatile than an instance store.

</details>

---

**2. A data processing application running on an EC2 instance needs a temporary scratch space for intermediate files that are generated during a job. The application requires extremely high, low-latency I/O for these temporary files. The files do not need to be kept after the job is complete. What is the most suitable storage solution for this scratch space?**

- A. An EBS General Purpose (gp3) volume.
- B. An EBS Provisioned IOPS (io2) volume.
- C. An EC2 Instance Store.
- D. An Amazon EFS file system.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is a classic use case for an EC2 Instance Store. The data is temporary ("scratch space"), and the primary requirement is very high performance. Because the instance store is physically attached to the host server, it provides the lowest latency and highest I/O performance, making it perfect for this scenario.

</details>

---

**3. An administrator stops an EC2 instance that has an attached EBS volume and a local instance store. What happens to the data on these volumes?**

- A. Data on both the EBS volume and the instance store is retained.
- B. Data on the EBS volume is retained, but data on the instance store is lost.
- C. Data on the EBS volume is lost, but data on the instance store is retained.
- D. Data on both the EBS volume and the instance store is lost.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This question tests the core concept of persistence. The EBS volume is persistent, so its data survives the instance stopping. The instance store is ephemeral, and its data is wiped clean when the instance is stopped, hibernated, or terminated.

</details>

---

**4. A company is running a large data warehousing application on EC2. The workload involves processing huge datasets with large, sequential read/write operations. Cost is a major concern. Which EBS volume type is most cost-effective for this workload?**

- A. General Purpose SSD (gp3)
- B. Provisioned IOPS SSD (io2)
- C. Throughput Optimized HDD (st1)
- D. Cold HDD (sc1)

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The workload is described as throughput-intensive (large, sequential operations), not IOPS-intensive (small, random operations). The Throughput Optimized HDD (st1) volume type is specifically designed and priced for this kind of big data workload, offering high throughput at a much lower cost than SSD-based volumes.

</details>

---

**5. A financial services application requires a high-performance database that needs a guaranteed level of at least 50,000 IOPS. Which EBS volume type should be used?**

- A. General Purpose SSD (gp3)
- B. Provisioned IOPS SSD (io2)
- C. Throughput Optimized HDD (st1)
- D. An EC2 Instance Store

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When a workload requires a specific, sustained, and high level of IOPS, the Provisioned IOPS SSD (io1/io2) family is the correct choice. It is the only EBS type that allows you to provision a guaranteed number of IOPS for your most critical, latency-sensitive applications.

</details>

---

**6. What is the primary mechanism for backing up data stored on an Amazon EBS volume?**

- A. Copying the volume to another Availability Zone.
- B. Creating an EBS Snapshot.
- C. Replicating the data to an EC2 Instance Store.
- D. Enabling versioning on the volume.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** EBS Snapshots are the standard, built-in mechanism for creating point-in-time backups of your volumes. These snapshots are stored durably in Amazon S3 and can be used to restore the volume or create new identical volumes.

</details>

---

**7. Which EC2 instance purchasing option is required to get a capacity reservation for a specific instance type in a specific Availability Zone?**

- A. A Zonal Reserved Instance
- B. A Regional Reserved Instance
- C. A Compute Savings Plan
- D. Spot Instances

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Only a Zonal Reserved Instance provides a capacity reservation. When you purchase an RI and scope it to a specific Availability Zone, AWS guarantees that you will be able to launch an instance of that type in that AZ. A Regional RI or Savings Plan provides a billing discount but does not guarantee capacity.

</details>

---

**8. An instance with an EBS-backed root volume is running. What operations can be performed on this instance?**

- A. It can only be running or terminated.
- B. It can be running, stopped, hibernated, or rebooted.
- C. It can only be stopped, not rebooted.
- D. It can only be rebooted, not stopped.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The persistence of the EBS root volume allows for a flexible instance lifecycle. You can stop the instance (which shuts it down and allows you to not pay for compute), and then start it again later, with its root volume and data intact. In contrast, an instance with an Instance Store root volume cannot be stopped.

</details>

---

**9. When an EC2 instance with an Instance Store is rebooted (either from the OS or via the AWS console), what happens to the data in the Instance Store?**

- A. The data is deleted.
- B. The data persists through the reboot.
- C. The data is automatically backed up to S3.
- D. The instance store volume is detached.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Data on an instance store survives a reboot. The instance remains on the same physical host server, so the physically attached disks are unaffected. The data is only lost on a stop, hibernate, or terminate action, or if the underlying host fails.

</details>

---

**10. A solutions architect is choosing an EBS volume for the boot volume of a fleet of general-purpose EC2 instances. The workload is variable, but performance and cost need to be balanced. The architect wants to provision IOPS and throughput independently of the storage size. Which EBS volume type is the best choice?**

- A. io2 Block Express
- B. gp2
- C. st1
- D. gp3

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** gp3 is the latest generation of General Purpose SSDs and is the recommended default for most workloads. Its key advantage over the older gp2 is that you can provision performance (IOPS and throughput) separately from capacity (size in GiB). This provides better performance and is often more cost-effective than gp2.

</details>

---

**11. A user wants to move an EBS volume from an EC2 instance in `us-east-1a` to a new instance in `us-east-1b`. What is the correct sequence of steps?**

- A. This is not possible; EBS volumes are locked to a single Availability Zone.
- B. Detach the volume, move it to `us-east-1b`, and then attach it to the new instance.
- C. Create a snapshot of the volume, create a new volume from the snapshot in `us-east-1b`, and then attach it to the new instance.
- D. Use the AWS CLI to directly modify the volume's Availability Zone.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** EBS volumes are zonal resources, meaning they are tied to the specific Availability Zone where they were created. You cannot directly move a volume between AZs. The standard procedure is to create a snapshot (which is a regional resource stored in S3) and then create a new volume from that snapshot, specifying the new target AZ.

</details>

---

**12. Which statement best describes the performance of an EC2 Instance Store?**

- A. It has high latency because it is a network-attached resource.
- B. It provides moderate, burstable throughput suitable for boot volumes.
- C. It offers very low latency and high I/O performance because it is physically attached to the host computer.
- D. Its performance is directly tied to the amount of provisioned IOPS.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The direct physical attachment to the host is the key reason for the instance store's high performance. It eliminates the network hop required for EBS, resulting in the lowest possible latency for storage I/O on EC2.

</details>

---

**13. A company needs to run a NoSQL database like Cassandra, which manages its own data replication for fault tolerance. The database requires very high random IOPS. Which combination of instance type and storage would be most suitable and cost-effective?**

- A. A General Purpose (M5) instance with an EBS gp3 volume.
- B. A Storage Optimized (I3) instance using its local NVMe SSD instance store.
- C. A Memory Optimized (R5) instance with an EBS io2 volume.
- D. A Compute Optimized (C5) instance with an EBS st1 volume.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Storage Optimized (I-family) instances are designed for this exact use case. They provide extremely high random IOPS via their local instance stores. Since the database (Cassandra) handles its own replication, the ephemeral nature of the instance store is not a problem; if one node fails, the data still exists on other nodes in the cluster. This is a very common pattern.

</details>

---

**14. An EBS volume's data is encrypted at rest. What does this mean?**

- A. Data is encrypted while it is in transit between the EC2 instance and the volume.
- B. Data is encrypted on the disk where it is stored within the AWS data center.
- C. The EBS snapshot created from the volume will be unencrypted by default.
- D. Only the root user can access the data.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Encryption "at rest" refers to the encryption of data while it is stored on the physical media. AWS transparently handles the encryption and decryption of data as your instance reads from and writes to the volume. Snapshots created from an encrypted volume are also automatically encrypted.

</details>

---

**15. What storage option is referred to as "ephemeral storage"?**

- A. Amazon EBS
- B. Amazon S3
- C. Amazon EFS
- D. EC2 Instance Store

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** "Ephemeral" means temporary or fleeting. This term is synonymous with the EC2 Instance Store because the data stored on it is not persistent and will be lost if the instance is stopped or terminated.

</details>

---

**16. The `gp2` EBS volume type's performance is tied to its size. How does it work?**

- A. The larger the volume, the lower the IOPS.
- B. IOPS performance is fixed regardless of size.
- C. The volume has a baseline of 3 IOPS per GiB, and can burst up to 3,000 IOPS.
- D. You can provision IOPS and size independently.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the key characteristic of the older gp2 volumes. They use a credit-based system similar to T-class instances. The baseline performance scales linearly with the size of the volume (3 IOPS per GiB), and smaller volumes can burst to higher performance for short periods. This is why the newer gp3 is often preferred, as it decouples performance from size.

</details>

---

**17. You want to change the volume type of an EBS volume attached to a running EC2 instance from `gp2` to `gp3` to improve performance. What is the procedure?**

- A. You must stop the instance, detach the volume, create a new `gp3` volume, copy the data, and attach the new volume.
- B. You can modify the volume's type directly in the EC2 console while the instance is running, without any downtime.
- C. You must take a snapshot of the `gp2` volume and then create a new `gp3` volume from the snapshot.
- D. It is not possible to change the type of an EBS volume after it has been created.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** EBS has a feature called "Elastic Volumes" that allows you to modify the size, volume type, or IOPS of a volume while it is in use, with no downtime or service interruption. This is a powerful feature for dynamically adjusting storage performance and capacity as needs change.

</details>

---

**18. Which storage type is physically located on the same host server as the EC2 instance?**

- A. Amazon S3
- B. Amazon EBS
- C. Amazon EFS
- D. EC2 Instance Store

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The EC2 Instance Store consists of disks that are physically part of the same server that is running your EC2 instance. This direct attachment is what gives it its very low latency. EBS, S3, and EFS are all network-attached storage services.

</details>

---

**19. A database on EC2 requires the highest possible level of durability and IOPS. Which EBS volume type should be selected?**

- A. gp3
- B. st1
- C. sc1
- D. io2 Block Express

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** io2 Block Express is the highest-performance EBS SSD volume, designed for the most demanding, I/O-intensive, business-critical applications. It offers the highest IOPS, highest throughput, and the highest durability (99.999%) of any EBS volume type.

</details>

---

**20. An application writes large log files sequentially. The primary goal is to achieve high throughput for writing these logs at the lowest possible cost. Which EBS volume type is most suitable?**

- A. Provisioned IOPS SSD (io1)
- B. General Purpose SSD (gp3)
- C. Throughput Optimized HDD (st1)
- D. EC2 Instance Store

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This workload is characterized by large, sequential writes (high throughput), not small random I/O (high IOPS). The st1 volume type is specifically designed and cost-optimized for these exact workloads, making it a much better fit than the more expensive SSD options.

</details>

---

**21. When can the data on an EC2 Instance Store be considered safe?**

- A. It is never safe, as it can be lost at any time.
- B. It is safe during a reboot of the instance.
- C. It is safe when the instance is stopped.
- D. It is safe when the instance is terminated.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A reboot operation keeps the instance on the same underlying physical host, so the physically attached instance store disks are not affected. Any other lifecycle change (stop, terminate, hibernate) or an underlying hardware failure will result in the loss of all data on the instance store.

</details>

---

**22. You detach an EBS root volume from a running instance and attach it to another instance. What is the likely result?**

- A. The original instance will crash, and the volume will seamlessly work on the new instance.
- B. This action is not permitted. You cannot detach the root volume from a running instance.
- C. The volume will be moved to a "pending" state until the original instance is terminated.
- D. The new instance will now have two root volumes.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** To ensure OS stability, you cannot detach the root volume (/dev/sda1 or /dev/xvda on Linux) from an EC2 instance while it is running. The instance must be in a stopped state before its root volume can be detached.

</details>
