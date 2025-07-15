# Step 28: AWS Snow Family & Storage Gateway

## 🎯 Objective
- [x] Master: **AWS Snow Family & Storage Gateway**

## 📘 Notes

### **Deep Dive: AWS Snow Family & Storage Gateway**

Moving data is a core challenge in cloud computing. For massive datasets, network transfers can be too slow, costly, or unreliable. AWS Snow Family provides physical devices for offline data transfer and edge computing. Separately, AWS Storage Gateway provides a hybrid bridge to seamlessly connect your on-premises applications to AWS storage.

---

### **1. AWS Snow Family ❄️**

The AWS Snow Family is a collection of physical devices that help you migrate petabyte-scale data into and out of AWS. They are essentially secure, ruggedized, shippable storage and compute devices.

- **AWS Snowcone:**
    - **What it is:** The smallest member of the family. A small, portable, and rugged device.
    - **Capacity:** Two models: 8 TB HDD or 14 TB HDD. Also has 2 vCPUs and 4 GB of memory.
    - **Use Case:** Designed for small-scale data transfer, edge computing, and use in disconnected or mobile environments like vehicles or industrial settings. It can be battery-operated. It supports AWS DataSync for online transfer.
- **AWS Snowball Edge:**
    - **What it is:** A petabyte-scale data transfer device with onboard storage and compute capabilities.
    - **Device Types:**
        1. **Snowball Edge Storage Optimized:** Best for large-scale data migrations. Provides up to 80 TB of usable HDD storage. It also has 40 vCPUs and 80 GB of memory for light edge computing.
        2. **Snowball Edge Compute Optimized:** Designed for running more demanding edge compute workloads. It has more compute power (up to 104 vCPUs) and optional GPUs, but less storage capacity.
    - **Use Case:** Large data migrations (e.g., from data centers), machine learning at the edge, and running EC2 instances or Lambda functions in disconnected environments like a factory floor or a ship.
- **AWS Snowmobile:**
    - **What it is:** An exabyte-scale data transfer service. It is literally a **45-foot long shipping container** pulled by a semi-trailer truck.
    - **Capacity:** Up to 100 PB (Petabytes).
    - **Use Case:** For migrating extremely large datasets, such as entire data centers, video libraries, or scientific data repositories. AWS drives the truck to your data center, helps you connect it, and then drives it back to an AWS facility to import the data.

**The Snow Family Process:**

1. You request a device from the AWS Console.
2. AWS ships the secure, encrypted device to you.
3. You connect the device to your local network and use the AWS OpsHub software or a DataSync agent to transfer data onto it.
4. You ship the device back to AWS.
5. AWS imports the data from the device into your specified S3 bucket.

---

### **2. AWS Storage Gateway** 🌉

AWS Storage Gateway is a hybrid cloud storage service that provides a bridge between your on-premises environment and AWS Storage. It's deployed as a virtual machine appliance in your data center.

- **Gateway Types:**
    1. **File Gateway:**
        - **What it provides:** A file interface (**NFS** or **SMB**) on-premises.
        - **How it works:** Your on-premises applications write files to the gateway's file share. The gateway then converts these files into objects and stores them durably in **Amazon S3**. It maintains a local cache of recently accessed files for low-latency performance.
        - **Use Case:** Easily extend on-premises file storage to S3, back up on-premises file data to the cloud, and provide cloud access to on-premises data.
    2. **Volume Gateway:**
        - **What it provides:** Block storage (**iSCSI**) to on-premises applications.
        - **Modes:**
            - **Cached Volumes:** The primary data is stored in S3, and a cache of frequently accessed data is kept locally on the gateway. This minimizes the need for on-premises storage.
            - **Stored Volumes:** The entire dataset is stored on-premises for low-latency access, and the gateway asynchronously backs up this data as EBS Snapshots in AWS. This is a disaster recovery solution.
    3. **Tape Gateway:**
        - **What it provides:** A virtual tape library (VTL) using the **iSCSI** interface.
        - **How it works:** It integrates with your existing backup software (like Veeam, NetBackup, etc.). The backup software writes to "virtual tapes," which are stored in S3. You can then archive these virtual tapes to S3 Glacier or Glacier Deep Archive for long-term, low-cost retention.
        - **Use Case:** A modern, cost-effective, and durable replacement for physical tape backup infrastructure.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A video production studio has accumulated 500 TB of raw video footage in their on-premises data center. Their internet connection is slow, and they need to migrate this data to Amazon S3 as quickly as possible. Which AWS service is the most suitable for this one-time migration?**

- A. AWS Storage Gateway - File Gateway
- B. AWS DataSync over a Direct Connect link.
- C. Multiple AWS Snowball Edge Storage Optimized devices.
- D. AWS Snowmobile.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** For a large, petabyte-scale offline data transfer, the AWS Snow Family is the correct choice. A single Snowball Edge Storage Optimized device holds up to 80 TB, so you would order several of them in parallel to move 500 TB. A network transfer (B) would be too slow, and a Snowmobile (D) is for exabyte-scale transfers (thousands of terabytes) and would be overkill.
</details>

---

**2. A company wants to replace its aging on-premises physical tape backup system. They want to continue using their existing backup software but want to store the backup archives in a durable, low-cost cloud solution. Which service is designed for this purpose?**

- A. AWS Storage Gateway - Tape Gateway
- B. AWS Backup
- C. Amazon S3 Glacier Deep Archive directly.
- D. AWS Snowball Edge.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** The Tape Gateway is the purpose-built solution for this. It emulates a physical tape library (a VTL), allowing it to integrate seamlessly with existing backup software. The software writes to the gateway, and the gateway stores the "virtual tapes" in S3 and archives them to Glacier, perfectly replacing the physical tape workflow.
</details>

---

**3. What is the primary use case for an AWS Snowmobile?**

- A. Migrating a 10 TB database to the cloud.
- B. Running edge compute applications in a moving vehicle.
- C. Performing an exabyte-scale data migration, such as moving an entire data center's worth of data.
- D. Providing a permanent hybrid storage solution.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The AWS Snowmobile is reserved for the largest data migrations imaginable, measured in many petabytes or exabytes. It is literally a data-center-on-wheels used to physically transport massive quantities of data.
</details>

---

**4. A company has an on-premises application that writes data to a local file server using the SMB protocol. They are running out of local storage and want to extend their storage into the cloud using Amazon S3 without changing their application. Which Storage Gateway configuration should they use?**

- A. Volume Gateway - Stored Mode
- B. Tape Gateway
- C. File Gateway configured with an SMB file share.
- D. File Gateway configured with an NFS file share.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The key requirements are a file-based interface (not block), the SMB protocol (for Windows), and storing the data in S3. A File Gateway is the only type that meets this need. It can be configured to present either an SMB or NFS share to the on-premises network while storing the backend data as objects in S3.
</details>

---

**5. Which member of the AWS Snow Family is the smallest, most portable device, suitable for use in environments with limited space and connectivity, like a drone or a mobile response vehicle?**

- A. AWS Snowball
- B. AWS Snowball Edge
- C. AWS Snowcone
- D. AWS Snowmobile

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The AWS Snowcone is the smallest and most portable device in the family. It's designed for edge computing and data transfer in space-constrained or mobile environments and can even be operated on battery power.
</details>

---

**6. A company needs to provide block storage via the iSCSI protocol to its on-premises servers. They want to keep their entire dataset locally for low-latency access and use the cloud primarily for taking off-site backups (snapshots) of this data. Which solution should they implement?**

- A. File Gateway
- B. Volume Gateway - Cached Mode
- C. Volume Gateway - Stored Mode
- D. Tape Gateway

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This describes the Volume Gateway in Stored mode. It provides iSCSI block storage, the entire primary dataset is stored on-premises, and it asynchronously creates EBS snapshots of the data for backup in S3.
</details>

---

**7. Which AWS Snow Family device offers an optional GPU, making it suitable for running machine learning inference models at the edge?**

- A. AWS Snowcone
- B. AWS Snowball Edge Compute Optimized
- C. AWS Snowball Edge Storage Optimized
- D. AWS Snowmobile

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Snowball Edge Compute Optimized device is designed for more intensive edge workloads. It offers more vCPUs, more memory, and an optional GPU, which is ideal for tasks like real-time video analysis or machine learning inference in disconnected environments.
</details>

---

**8. What is a key difference between AWS DataSync and AWS Storage Gateway?**

- A. DataSync is for online data transfer, while Storage Gateway is for offline data transfer.
- B. DataSync is a data migration and transfer service, while Storage Gateway is a hybrid storage integration service that provides persistent access.
- C. DataSync uses physical devices, while Storage Gateway is a purely software-based service.
- D. DataSync can only transfer data to S3, while Storage Gateway can transfer data to EFS and FSx.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the core distinction. DataSync is designed to move data from point A to point B efficiently. Storage Gateway is designed to live in your data center permanently, providing a seamless bridge that makes cloud storage look like a local on-premises resource.
</details>

---

**9. In a File Gateway configuration, where is the authoritative copy of the data stored?**

- A. On the on-premises gateway appliance's cache disk.
- B. In an Amazon EFS file system.
- C. In an Amazon S3 bucket.
- D. In an EBS volume.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A File Gateway uses S3 as its primary, authoritative data store. The on-premises gateway appliance maintains a local cache of frequently or recently accessed files to provide low-latency performance, but the complete dataset resides in the designated Amazon S3 bucket.
</details>

---

**10. A hospital is decommissioning a data center and needs to migrate 20 PB of medical archives to Amazon S3 Glacier Deep Archive. An online transfer would take years. What is the most feasible solution?**

- A. Order 250 Snowball Edge Storage Optimized devices.
- B. Use AWS DataSync over multiple 10 Gbps Direct Connect lines.
- C. Order an AWS Snowmobile.
- D. Use the S3 Multi-Part Upload API from the data center.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A 20 PB (20,000 TB) migration is an enormous amount of data, falling squarely into the exabyte-scale category. The AWS Snowmobile is the service purpose-built for this scale of migration.
</details>

---

**11. A company needs to provide its on-premises virtualization platform with iSCSI storage. They have limited on-premises storage capacity and want to use Amazon S3 as their primary data store, keeping only frequently accessed data on-site. Which Storage Gateway configuration is best?**

- A. File Gateway with NFS
- B. Volume Gateway - Cached Mode
- C. Volume Gateway - Stored Mode
- D. Tape Gateway

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The requirements are iSCSI block storage and using S3 as the primary data store due to limited on-premises capacity. This perfectly describes the Volume Gateway in Cached mode.
</details>

---

**12. When you receive an AWS Snowball Edge device, how do you transfer data onto it?**

- A. By connecting it directly to the internet to download data from S3.
- B. By using the AWS OpsHub for Snow Family application or an S3 adapter to copy data from local sources.
- C. By shipping your hard drives to AWS to be installed in the device.
- D. By using AWS DataSync to pull data from another AWS Region.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** AWS provides a client application called AWS OpsHub that you install in your local network. It gives you a graphical interface to unlock the device, configure it, and manage the data transfer from your local servers or NAS devices onto the Snowball.
</details>

---

**13. Which AWS service is NOT a valid target destination for data being migrated by AWS DataSync?**

- A. Amazon S3
- B. Amazon EFS
- C. Amazon FSx for Windows File Server
- D. Amazon EBS

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** AWS DataSync is designed to move data to managed file or object storage services. It supports S3, EFS, FSx for Windows, FSx for Lustre, and FSx for OpenZFS as destinations. It cannot be used to migrate data directly into an Amazon EBS block volume.
</details>

---

**14. What security features are built into AWS Snowball Edge devices to protect data in transit?**

- A. The data is transferred over an SSL/TLS connection.
- B. The device has a ruggedized, tamper-resistant enclosure, and all data written to it is automatically encrypted with 256-bit encryption.
- C. The device is escorted by armed guards during shipping.
- D. The device uses Network ACLs and Security Groups.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Security is paramount for the Snow Family. The devices use tamper-resistant enclosures with a Trusted Platform Module (TPM) chip. All data transferred to the device is automatically encrypted. The device is unlocked using a manifest file and unlock code that are retrieved separately from AWS, ensuring a secure chain of custody.
</details>

---

**15. A File Gateway provides access to objects in S3 using which two file protocols? (Select TWO)**

- A. iSCSI
- B. NFS
- C. SMB
- D. Lustre
- E. FTP

<details>
<summary>View Answer</summary>
<br>

**Answers: B and C**

**Explanation:** A File Gateway acts as a protocol bridge. It presents a standard file share to your on-premises network using either the Network File System (NFS) protocol, common in Linux/Unix environments, or the Server Message Block (SMB) protocol, common in Windows environments.
</details>

---

**16. What is the primary reason to use an offline data transfer service like the AWS Snow Family instead of an online method like DataSync?**

- A. When you have a very large amount of data to move and a limited or slow internet connection, making the network transfer time-prohibitive.
- B. When the data requires application-consistent backups.
- C. When you need to provide file-based access to S3.
- D. When the data needs to be encrypted.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** The decision between online and offline transfer is almost always a calculation of time and cost. If you have petabytes of data and a standard internet connection, it could take months or years to transfer. In these cases, it is much faster to physically ship the data using a Snowball or Snowmobile.
</details>

---

**17. Which Storage Gateway mode keeps the primary copy of the data on-premises?**

- A. File Gateway
- B. Volume Gateway - Cached Mode
- C. Volume Gateway - Stored Mode
- D. Tape Gateway

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Only the Volume Gateway in Stored mode maintains the full, primary dataset on your local storage hardware. All other gateway types (File, Tape, and Volume Cached) store the primary, authoritative copy of the data in the AWS cloud (specifically S3).
</details>

---

**18. You can run EC2 instances and Lambda functions directly on which of the following devices?**

- A. An AWS Storage Gateway appliance.
- B. An AWS Snowball Edge device.
- C. An AWS Direct Connect router.
- D. An AWS Snowmobile.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** AWS Snowball Edge devices (both Storage Optimized and Compute Optimized) have onboard compute capacity. This allows them to be used for edge computing use cases, where you can run EC2 instances or Lambda functions to process data locally in disconnected or remote environments before sending results back to an AWS Region.
</details>

---

**19. What does the "Tape Gateway" use as its backend storage for its virtual tapes?**

- A. Amazon EBS
- B. Amazon EFS
- C. Amazon S3 and S3 Glacier
- D. Local disk on the gateway appliance.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** When your backup software writes to the Tape Gateway's virtual tape library, the gateway stores these virtual tapes as objects in Amazon S3. For long-term, low-cost archival, you can then move these virtual tapes from S3 to S3 Glacier or S3 Glacier Deep Archive.
</details>

---

**20. A remote mining site has no stable internet connection but needs to collect and pre-process 5 TB of seismic data before sending it to an S3 bucket in the nearest AWS region. Which device is most suitable for this task?**

- A. An AWS Storage Gateway - File Gateway
- B. An AWS Snowcone
- C. An AWS Snowmobile
- D. An Application Load Balancer

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** An AWS Snowcone is ideal for this scenario. It is small, rugged, and can operate in disconnected environments. It provides enough storage (8 or 14 TB) for the 5 TB of data and has onboard compute to perform the pre-processing. Once the data is processed, the device can be shipped back to AWS for data import.
</details>

---

**21. A company wants to use AWS Storage Gateway to provide low-latency file access to their on-premises users while storing all data in S3. What is a key component of the on-premises gateway appliance that enables this low latency?**

- A. A dedicated GPU.
- B. A local disk cache.
- C. A connection to AWS Global Accelerator.
- D. A built-in AWS WAF.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** All gateway types that store primary data in S3 (File Gateway and Volume Gateway - Cached Mode) utilize a local disk cache on the on-premises appliance. This cache stores the most recently or frequently accessed data locally, so that when a user requests that data, it can be served directly from the cache with very low latency, avoiding a round trip to the cloud.
</details>

---

**22. Which service is designed as a long-term hybrid solution that remains in your data center, as opposed to a service for a one-time data transfer project?**

- A. AWS Snowball
- B. AWS Snowmobile
- C. AWS Storage Gateway
- D. AWS DataSync

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS Storage Gateway is designed to be a permanent bridge between your on-premises environment and the AWS cloud, providing ongoing, integrated access to cloud storage. The Snow Family devices and DataSync are primarily used for migration projects or periodic large-scale data movement, not for continuous, real-time access.
</details>

---