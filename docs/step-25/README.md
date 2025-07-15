# Step 25: EBS Volume Types & Performance

## 🎯 Objective

- [x] Master: **EBS Volume Types & Performance**

## 📘 Notes

### **Deep Dive: EBS Volume Types & Performance**

Amazon Elastic Block Store (EBS) provides a range of volume types that differ in performance characteristics and price. Choosing the right volume type is a critical part of designing a cost-effective and performant architecture. The main choice is between Solid State Drives (SSD), optimized for transactional workloads (measured in IOPS), and Hard Disk Drives (HDD), optimized for streaming workloads (measured in Throughput).

---

### **1. Solid State Drives (SSD-Backed)** 🚀

SSD-backed volumes are designed for workloads that involve frequent, small, random read/write operations where the dominant performance metric is **IOPS** (Input/Output Operations Per Second).

- **General Purpose SSD (gp2 / gp3):**
  - **Use Case:** The default, all-around best choice for a wide variety of workloads. Perfect for system boot volumes, virtual desktops, development/test environments, and most applications.
  - **gp2 (Legacy):** Performance is tied to size. It provides a baseline of 3 IOPS per GiB of volume size and can "burst" up to 3,000 IOPS for short periods.
  - **gp3 (Latest & Recommended):** The key advantage is that you can provision **IOPS and throughput independently** of storage size. It provides a consistent baseline of 3,000 IOPS and 125 MiB/s throughput regardless of size, which you can scale up as needed. It's often cheaper and more performant than `gp2`.
- **Provisioned IOPS SSD (io1 / io2 / io2 Block Express):**
  - **Use Case:** Your most critical, I/O-intensive applications that require sustained, high IOPS performance. Ideal for large relational or NoSQL databases (e.g., MySQL, Oracle, Cassandra).
  - **Performance:** You provision a specific, guaranteed number of IOPS (e.g., 40,000 IOPS).
  - **io2 / io2 Block Express:** The latest generation, offering much higher durability (99.999% - "5 nines") and a better IOPS-to-size ratio than `io1`. `io2 Block Express` is the highest performance block storage on AWS, offering sub-millisecond latency.

---

### **2. Hard Disk Drives (HDD-Backed)** 💿

HDD-backed volumes are designed for large, streaming workloads where the dominant performance attribute is **Throughput** (measured in MiB/s). They are not suitable for boot volumes.

- **Throughput Optimized HDD (st1):**
  - **Use Case:** Frequently accessed, throughput-intensive workloads with large datasets and large I/O sizes. Perfect for big data processing frameworks like Hadoop and Kafka, data warehouses, and log processing.
  - **Performance:** Measured in MiB/s. Performance scales with the size of the volume.
- **Cold HDD (sc1):**
  - **Use Case:** The lowest-cost block storage. Designed for large volumes of infrequently accessed data where cost is the primary driver. Think of it as a "cold" storage tier for block data.
  - **Performance:** Lower throughput than `st1`. Suitable for data you might access only a few times a day.

---

### **Summary of Volume Types**

| Category | Volume Type                    | Primary Use Case                                                 | Performance Metric         |
| -------- | ------------------------------ | ---------------------------------------------------------------- | -------------------------- |
| **SSD**  | **gp3** (General Purpose)      | **Default for most workloads**, boot volumes, web servers        | Balanced IOPS & Throughput |
| **SSD**  | **io2** (Provisioned IOPS)     | Critical databases, low-latency apps needing **guaranteed IOPS** | **IOPS**                   |
| **HDD**  | **st1** (Throughput Optimized) | Big data, data warehouses, log processing                        | **Throughput (MiB/s)**     |
| **HDD**  | **sc1** (Cold)                 | Infrequently accessed data, lowest-cost block storage            | Throughput (MiB/s)         |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A large transactional database requires sustained performance of at least 20,000 IOPS for its primary data files. Which EBS volume type is specifically designed to meet this requirement?**

- A. General Purpose SSD (gp3)
- B. Provisioned IOPS SSD (io2)
- C. Throughput Optimized HDD (st1)
- D. Cold HDD (sc1)

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When a workload demands a specific, sustained, and guaranteed level of IOPS, the Provisioned IOPS SSD (io1/io2) family is the only correct choice. It is designed for mission-critical, I/O-intensive databases that cannot tolerate performance fluctuations.

</details>

---

**2. A company is running a big data processing job using a Hadoop cluster on EC2. The workload involves streaming large amounts of data sequentially to and from the disks. The company wants the most cost-effective storage solution for this task. Which EBS volume type is most suitable?**

- A. io2 Block Express
- B. gp3
- C. st1
- D. sc1

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This workload is throughput-bound (large, sequential streams), not IOPS-bound. The Throughput Optimized HDD (st1) volume type is engineered and priced specifically for this kind of big data workload, offering high throughput at a much lower cost than any of the SSD options.

</details>

---

**3. What is the primary advantage of using the `gp3` volume type over the older `gp2`?**

- A. `gp3` offers higher durability than `gp2`.
- B. `gp3` allows you to provision IOPS and throughput performance independently of the storage size.
- C. `gp3` is an HDD-backed volume, making it cheaper.
- D. `gp3` supports encryption, while `gp2` does not.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The key innovation of gp3 is the decoupling of performance from size. With gp2, you had to provision a larger disk just to get more IOPS. With gp3, you get a solid baseline performance (3,000 IOPS / 125 MiB/s) and can pay to scale performance up separately, which often results in better performance at a lower cost.

</details>

---

**4. For which of the following workloads would an HDD-backed volume like `st1` or `sc1` be an unsuitable choice?**

- A. A data warehouse.
- B. A server's boot volume.
- C. Log processing.
- D. Storing video archives.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** HDD-backed volumes (st1 and sc1) cannot be used as boot volumes for EC2 instances. Boot volumes require the low-latency, small I/O characteristics of SSDs to function properly. You must use an SSD type (gp2, gp3, io1, or io2) for boot volumes.

</details>

---

**5. A financial application requires the highest possible level of storage durability for its database volumes, specified as 99.999% durability. Which EBS volume type meets this requirement?**

- A. gp3
- B. st1
- C. io2
- D. All EBS volumes offer this level of durability.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While all EBS volumes are highly durable, the io2 and io2 Block Express volumes are designed for a higher durability rate of 99.999% (five nines) compared to the 99.8% - 99.9% (three nines) of other volume types. This is intended for mission-critical enterprise workloads.

</details>

---

**6. A company needs an inexpensive storage solution for a large amount of infrequently accessed data. The data needs to be readily available when requested, so Glacier is not an option. Cost is the absolute primary driver. Which EBS volume type is the best fit?**

- A. Throughput Optimized HDD (st1)
- B. Cold HDD (sc1)
- C. General Purpose SSD (gp3)
- D. Provisioned IOPS SSD (io1)

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Cold HDD (sc1) is designed to provide the lowest possible cost per gigabyte for block storage. It's specifically for large volumes of data that are accessed infrequently, making it the most cost-effective choice when performance is not a major concern.

</details>

---

**7. A database administrator is running a report that performs a large table scan. What is the primary performance metric for this operation?**

- A. IOPS
- B. Throughput (MiB/s)
- C. Durability
- D. Latency

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A table scan is a large, sequential read operation. This type of workload is measured by Throughput, which is the amount of data that can be transferred per second (MiB/s). IOPS (A) measures performance for small, random read/write operations, like those in a transactional database.

</details>

---

**8. An EC2 instance is configured with a `gp2` volume. The administrator notices that performance is sometimes very fast but at other times becomes very slow. What is the likely cause?**

- A. The volume is running out of burst credits.
- B. The `gp2` volume is not a bootable volume type.
- C. The volume has been detached from the instance.
- D. The instance is in a different Availability Zone from the volume.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** gp2 volumes use a burst bucket model for performance. The volume accumulates I/O credits when it is idle or under its baseline performance level. During periods of high I/O, it spends these credits to "burst" to a higher performance level. If the workload is sustained, the burst credits can be exhausted, causing performance to throttle down to the much lower baseline level.

</details>

---

**9. What is the recommended default EBS volume type for most common workloads, such as system boot volumes and web servers?**

- A. io1
- B. st1
- C. gp3
- D. sc1

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS recommends gp3 as the go-to General Purpose volume type for the vast majority of use cases. It provides an excellent balance of price and performance and offers more flexibility than its predecessor, gp2.

</details>

---

**10. You need to modify an existing EBS volume to increase its provisioned IOPS. Can this be done without detaching the volume from the instance?**

- A. No, the instance must be stopped and the volume detached first.
- B. Yes, you can use the EBS Elastic Volumes feature to modify the IOPS of a live volume with no downtime.
- C. No, you must create a new volume with the desired IOPS and migrate the data.
- D. Yes, but only if the volume is an HDD-backed type.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The EBS Elastic Volumes feature allows you to dynamically modify the size, type, and IOPS (for io1/io2/gp3) of an EBS volume while it is still attached to a running instance. This allows for performance and capacity changes without service interruption.

</details>

---

**11. An application performs many small, random read and write operations per second. Which performance metric is most important for this workload?**

- A. Throughput (MiB/s)
- B. IOPS (Input/Output Operations Per Second)
- C. Storage Capacity (GB)
- D. Durability percentage

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Small, random I/O is characteristic of transactional workloads like databases. The key performance metric for this access pattern is IOPS. SSD-backed volumes (gp3, io2) are designed to provide high IOPS.

</details>

---

**12. Why would a company choose `io2` over `gp3` for a database?**

- A. `io2` is significantly cheaper than `gp3`.
- B. The database requires a guaranteed, consistent level of IOPS that exceeds the baseline performance of `gp3`.
- C. `gp3` cannot be used for databases.
- D. `io2` has lower durability than `gp3`.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** While gp3 is excellent for general use, its performance is not guaranteed in the same way as io2. A company would choose io2 when their application's performance is critically dependent on a consistent, non-variable, and high level of IOPS that gp3 cannot provide.

</details>

---

**13. Which of the following is a characteristic of `st1` volumes?**

- A. They are ideal for small, random I/O workloads.
- B. They cannot be used as a boot volume.
- C. They are SSD-backed for maximum performance.
- D. They have a fixed IOPS performance regardless of size.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A key limitation of all HDD-backed volumes (st1 and sc1) is that they cannot be used as a boot volume for an EC2 instance. They are designed as low-cost data volumes for specific throughput-intensive workloads.

</details>

---

**14. A user wants to attach a single EBS volume to multiple EC2 instances simultaneously to be read from and written to by all of them. What feature enables this?**

- A. This is not possible with EBS.
- B. EBS Multi-Attach
- C. EBS Snapshots
- D. EBS Elastic Volumes

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The EBS Multi-Attach feature allows a single Provisioned IOPS (io1 or io2) volume to be concurrently attached to multiple EC2 instances within the same Availability Zone. This is a specialized feature for applications that manage their own storage consistency, such as some clustered databases.

</details>

---

**15. You are designing a system that processes a large queue of incoming log files. The processing involves reading each large file sequentially from start to finish. Which volume type would offer the best price-to-performance ratio?**

- A. io1
- B. gp3
- C. st1
- D. sc1

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Reading large files sequentially is a throughput-bound workload. The st1 volume type is optimized for high throughput at a low cost, making it the perfect fit for log processing, ETL jobs, and data warehousing.

</details>

---

**16. What happens to the performance of a `gp2` volume as its size increases?**

- A. The baseline IOPS performance increases.
- B. The baseline IOPS performance decreases.
- C. The performance remains constant.
- D. The throughput decreases.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** The baseline performance of a gp2 volume is directly proportional to its size, calculated at a rate of 3 IOPS per GiB. Therefore, a larger gp2 volume will have a higher baseline IOPS performance than a smaller one.

</details>

---

**17. Which volume type would be considered a "General Purpose" volume?**

- A. io2
- B. gp3
- C. st1
- D. sc1

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The "gp" in gp3 stands for General Purpose. This volume type is designed to be a well-balanced default choice for the majority of application workloads.

</details>

---

**18. Which statement about HDD-backed volumes is TRUE?**

- A. They provide the highest IOPS performance.
- B. They are optimized for small, random I/O operations.
- C. Their performance is measured primarily in throughput (MiB/s).
- D. They are more expensive than SSD-backed volumes.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The strength of HDD volumes lies in their ability to deliver high sequential data transfer rates at a low cost. Therefore, their performance is measured in throughput (MiB/s), not IOPS.

</details>

---

**19. A company is migrating its on-premises Oracle database to EC2. The database has extremely high I/O requirements and sub-millisecond latency is critical. Which EBS volume offers the highest performance for this use case?**

- A. gp3 with maximum provisioned IOPS.
- B. io2 Block Express
- C. A RAID 0 array of st1 volumes.
- D. sc1

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** io2 Block Express is the highest performance block storage offered by AWS. It is built on a different architecture (the Scalable Reliable Datagram) to provide sub-millisecond latency and the highest IOPS/throughput, making it ideal for the most demanding enterprise databases like Oracle, SAP HANA, and Microsoft SQL Server.

</details>

---

**20. A volume is needed to store files that are accessed very rarely, perhaps once or twice a month. The primary requirement is the lowest possible storage cost for block storage. Which volume type should be selected?**

- A. st1
- B. gp2
- C. sc1
- D. io1

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Cold HDD (sc1) is designed for this exact scenario. It offers the lowest price point per gigabyte of any EBS volume type, making it suitable for large amounts of data that are infrequently accessed but still need to be on block storage.

</details>

---

**21. What are the two main categories of EBS volumes?**

- A. Ephemeral and Persistent
- B. Boot and Data
- C. Solid State Drive (SSD) and Hard Disk Drive (HDD)
- D. Encrypted and Unencrypted

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The primary classification of EBS volumes is based on the underlying storage technology: Solid State Drives (SSD), which are fast for random I/O (IOPS), and Hard Disk Drives (HDD), which are economical for sequential throughput.

</details>

---

**22. An administrator has chosen a `gp3` volume. Which two performance characteristics can they configure independently of the volume's size? (Select TWO)**

- A. Durability
- B. IOPS
- C. Throughput
- D. Latency
- E. Availability Zone

<details>
<summary>View Answer</summary>

**Answers: B and C**

**Explanation:** The defining feature of a gp3 volume is the ability to provision IOPS and Throughput separately from the storage capacity. You can have a small 10 GB volume with very high IOPS and throughput, or a large 1 TB volume with only the baseline performance, providing maximum flexibility.

</details>
