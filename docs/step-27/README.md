# Step 27: EFS vs FSx & Hybrid Storage

## 🎯 Objective
- [x] Master: **EFS vs FSx & Hybrid Storage**

## 📘 Notes

### **Deep Dive: EFS vs. FSx & Hybrid Storage**

While EBS provides block storage to a single EC2 instance, many applications require a shared file system that can be accessed by multiple instances simultaneously. AWS offers managed file storage services for this purpose.1 Additionally, AWS provides services to seamlessly integrate your on-premises storage with the cloud.2

---

### **1. Managed File Systems in AWS**

- **Amazon EFS (Elastic File System):**
    - **What it is:** A fully managed, scalable file storage service for use with Amazon EC2 instances.3 EFS provides a simple, serverless, set-and-forget elastic file system.4
    - **Protocol:** Uses the **NFSv4** protocol.
    - **Compatibility:** Designed for **Linux-based workloads**.5 Windows instances do not have a native NFSv4 client and cannot connect to EFS easily.
    - **Key Features:**
        - **Scalability:** Automatically grows and shrinks as you add and remove files, with no need to provision storage in advance.6
        - **Concurrency:** Can be mounted and accessed by thousands of EC2 instances from multiple Availability Zones concurrently.7
        - **Performance Modes:** Offers General Purpose (latency-sensitive use cases) and Max I/O (latency-tolerant, high-throughput use cases).8
        - **Storage Classes:** Has a Standard tier and an Infrequent Access (EFS IA) tier with lifecycle policies to move old files to the cheaper tier automatically.9
    - **Use Case:** The standard choice for a shared file system for Linux-based applications, such as web serving, content management systems (like WordPress), and home directories.
- **Amazon FSx (File System X):**
    - **What it is:** A service that provides fully managed third-party file systems. It allows you to run popular, feature-rich file systems on AWS without managing the underlying hardware or software.10
    - **Amazon FSx for Windows File Server:**
        - **What it is:** A fully managed, native Microsoft Windows file system.
        - **Protocol:** Uses the **SMB (Server Message Block)** protocol.
        - **Compatibility:** Designed for **Windows-based workloads**.
        - **Key Features:** Provides all the features of a Windows file server, including full support for Active Directory integration, user quotas, and Windows NTFS ACLs.
        - **Use Case:** The go-to choice for lifting-and-shifting Windows applications that rely on shared file storage, user home directories for Windows environments, and any application that requires SMB.
    - **Amazon FSx for Lustre:**
        - **What it is:** A fully managed, high-performance file system optimized for fast processing of workloads.11
        - **Protocol:** Lustre (a parallel distributed file system).12
        - **Compatibility:** Linux clients.
        - **Key Features:** Provides sub-millisecond latencies and up to hundreds of gigabytes per second of throughput. It can be linked to an S3 bucket, allowing you to process your S3 data with a high-performance file system interface.13
        - **Use Case:** High-Performance Computing (HPC), financial modeling, media rendering, and machine learning workloads that require extremely fast access to large datasets.

---

### **2. Hybrid Cloud Storage Solutions**

- **AWS Storage Gateway:**
    - **What it is:** A hybrid cloud storage service that gives your on-premises applications access to virtually unlimited cloud storage.14 It's typically deployed as a virtual machine (or hardware appliance) in your on-premises data center.
    - **Gateway Types:**
        1. **File Gateway:** Presents a **file interface (NFS or SMB)** to your on-premises servers, but stores the files as objects in Amazon S3.15 It provides a local cache for low-latency access to your most recently used files.
        2. **Volume Gateway:** Presents block storage to your on-premises applications using the **iSCSI protocol**.16
            - **Cached Volumes:** Stores your primary data in S3 and caches frequently accessed data locally on the gateway.
            - **Stored Volumes:** Stores your entire dataset locally on-premises and asynchronously backs up point-in-time snapshots to S3. This is a good backup/DR solution.
        3. **Tape Gateway:** Replaces physical backup tapes with virtual tapes stored in the cloud.17 It presents an iSCSI virtual tape library (VTL) to your existing backup software (like NetBackup or Veeam).18 The virtual tapes are stored in S3 and can be archived to S3 Glacier Deep Archive.19
- **AWS DataSync:**
    - **What it is:** An online data transfer service that simplifies, automates, and accelerates moving large amounts of data between on-premises storage systems and AWS Storage services (like S3, EFS, or FSx).20
    - **How it works:** You deploy a DataSync agent in your on-premises environment. DataSync automatically handles scripting copy jobs, scheduling, monitoring, data integrity validation, and optimizing network utilization.21
    - **Use Case:** The primary tool for **large-scale data migrations** from on-premises file systems to AWS. It is much faster and more reliable than open-source tools like `rsync`.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to provide a shared, scalable file system for a fleet of Linux-based EC2 instances that host a content management system. The file system must be accessible from multiple instances concurrently. Which AWS service is the most appropriate choice?**

- A. Amazon EBS
- B. Amazon EFS
- C. Amazon FSx for Windows File Server
- D. EC2 Instance Store

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon EFS is the standard, fully managed solution for providing a shared file system to Linux-based EC2 instances using the NFS protocol. It is designed for concurrent access and scales automatically, making it perfect for use cases like web serving and content management.
</details>

---

**2. An enterprise is migrating its on-premises user home directories to AWS. The existing environment is entirely Windows-based and uses Active Directory for authentication. Users need to access their files via the SMB protocol. Which service should be used?**

- A. Amazon S3
- B. Amazon EFS
- C. Amazon FSx for Windows File Server
- D. Amazon FSx for Lustre

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The key requirements are Windows compatibility, Active Directory integration, and the SMB protocol. Amazon FSx for Windows File Server is the purpose-built service that meets all these needs, providing a fully managed, native Windows file system experience.
</details>

---

**3. A research institute is running a High-Performance Computing (HPC) cluster on thousands of EC2 instances. The application needs to process a massive dataset stored in S3 and requires a parallel file system that offers sub-millisecond latencies and extremely high throughput. Which service is designed for this workload?**

- A. Amazon EFS with Max I/O mode.
- B. Amazon FSx for Lustre.
- C. An EBS io2 Block Express volume.
- D. AWS Storage Gateway.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon FSx for Lustre is specifically designed for HPC and high-performance workloads. It provides a massively parallel, high-speed file system that can be linked directly to an S3 bucket, allowing you to process S3 data at extremely high speeds.
</details>

---

**4. A company wants to back up its on-premises application data to the cloud. They want to present block storage (iSCSI) to their on-premises servers, store the full dataset on-premises for fast access, and asynchronously back up snapshots of this data to Amazon S3. Which AWS Storage Gateway type should they use?**

- A. File Gateway
- B. Volume Gateway - Cached Volumes
- C. Volume Gateway - Stored Volumes
- D. Tape Gateway

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This describes the Volume Gateway in Stored mode. The primary data resides on-premises on your local hardware for low-latency access. The gateway then takes care of creating and uploading EBS snapshots of this data to S3 for a durable, off-site backup.
</details>

---

**5. What is the primary use case for AWS DataSync?**

- A. Providing low-latency access to files stored in S3 from on-premises servers.
- B. Replacing physical backup tapes with a virtual tape library in the cloud.
- C. Automating and accelerating large-scale data migrations from on-premises file systems to AWS storage services like S3 or EFS.
- D. Providing a shared file system for Windows-based applications.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS DataSync is a data transfer and migration service. Its core purpose is to make it faster, simpler, and more reliable to move large quantities of data from on-premises NAS or file systems into AWS. It is not a file system itself but a tool for moving data to one.
</details>

---

**6. A company has an on-premises application that produces log files. They want these files to be stored durably in Amazon S3, but they want the application to continue writing to a local file share using the NFS protocol. Which hybrid storage solution fits this requirement?**

- A. AWS DataSync
- B. Volume Gateway - Cached Volumes
- C. Tape Gateway
- D. File Gateway

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** A File Gateway provides an NFS or SMB endpoint on-premises. Your applications can write to this endpoint just like a traditional file share. The gateway then transparently converts these files into objects and stores them in your specified S3 bucket, providing a seamless bridge between a file interface and object storage.
</details>

---

**7. An organization is retiring its on-premises physical tape library. They want to move to a cloud-based solution but want to continue using their existing backup software (e.g., NetBackup, Veeam) with minimal changes. Which Storage Gateway type is designed for this?**

- A. File Gateway
- B. Volume Gateway
- C. Tape Gateway
- D. S3 Gateway

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The Tape Gateway is the perfect solution for this "tape lift-and-shift." It presents a standard iSCSI Virtual Tape Library (VTL) interface that is compatible with all major backup applications. The backup software sees a tape library, but the "tapes" are actually virtual tapes stored durably and cost-effectively in S3 and Glacier.
</details>

---

**8. Which AWS managed file system is based on the NFS protocol and is designed for Linux-based workloads?**

- A. Amazon S3
- B. Amazon EFS
- C. Amazon FSx for Windows File Server
- D. Amazon EBS

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon EFS is AWS's managed Network File System (NFS) v4 service. It is the standard choice for providing a shared, POSIX-compliant file system to Linux EC2 instances.
</details>

---

**9. What is the key difference between the Cached and Stored modes for a Volume Gateway?**

- A. Cached mode uses SMB, while Stored mode uses NFS.
- B. Cached mode stores the primary data in S3 and caches it locally, while Stored mode stores the primary data locally and backs it up to S3.
- C. Stored mode is for file storage, while Cached mode is for block storage.
- D. Stored mode stores data in EFS, while Cached mode stores data in S3.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the core distinction. Cached mode is for when you want to use S3 as your primary data store to save on-premises storage costs, keeping only a small, frequently accessed subset locally. Stored mode is for when you need low-latency access to your entire dataset locally, using S3 primarily as a backup and disaster recovery target.
</details>

---

**10. A company needs to perform a one-time migration of 50 TB of data from their on-premises NAS to an Amazon S3 bucket. They need the transfer to be fast, secure, and include data integrity checks. Which service is most appropriate?**

- A. AWS Storage Gateway - File Gateway
- B. AWS Snowball Edge
- C. AWS DataSync
- D. A custom script using the AWS CLI over a Direct Connect link.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS DataSync is purpose-built for large-scale online data migrations. It includes built-in acceleration, encryption, scheduling, and validation, making it significantly more performant and reliable than a custom script. A File Gateway is for ongoing access, not a one-time migration. A Snowball is for offline transfers, which may be an option, but DataSync is the primary online transfer service.
</details>

---

**11. Which protocol is used by Amazon FSx for Windows File Server?**

- A. NFS
- B. iSCSI
- C. SMB
- D. Lustre

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Amazon FSx for Windows File Server provides a native Windows file system, which uses the Server Message Block (SMB) protocol for network file sharing.
</details>

---

**12. A fleet of EC2 instances running a web application needs to access a shared directory for storing user-uploaded images. The instances are spread across multiple Availability Zones for high availability. Which storage service can be mounted by all instances simultaneously?**

- A. A single EBS gp3 volume.
- B. An EC2 Instance Store on one of the instances.
- C. Amazon EFS.
- D. An S3 bucket mounted as a file system.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Amazon EFS is designed for this exact use case. It allows thousands of instances, even those in different Availability Zones, to connect to and mount the same file system concurrently. A standard EBS volume can only be attached to one instance at a time (unless using the specialized Multi-Attach feature).
</details>

---

**13. The main benefit of using Amazon FSx for Lustre linked to an S3 bucket is:**

- A. To provide a low-cost, long-term archive for S3 data.
- B. To provide a high-performance, POSIX-compliant file system interface for data that is stored in S3.
- C. To provide SMB access to objects stored in S3.
- D. To automatically encrypt all data in the linked S3 bucket.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** FSx for Lustre allows you to "present" your S3 objects as files in a high-performance file system. This lets you use existing file-based applications and tools to run compute-intensive workloads on your S3 data without having to modify them to use S3 APIs directly.
</details>

---

**14. Where is the primary data stored when using a Volume Gateway in Cached mode?**

- A. On the on-premises storage array.
- B. In an Amazon EFS file system.
- C. In Amazon S3.
- D. In an EBS volume attached to the gateway.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** In Cached mode, the authoritative copy of the data is stored in Amazon S3. The on-premises gateway hardware is used to store a cache of the most frequently accessed data for low-latency performance.
</details>

---

**15. Which of the following is a key feature of Amazon EFS?**

- A. It provides block storage via the iSCSI protocol.
- B. It can be accessed natively from Windows EC2 instances.
- C. It automatically scales its storage capacity as files are added or removed.
- D. It is designed for single-instance access.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The "E" in EFS stands for Elastic. A key benefit is that you do not need to provision storage capacity in advance. The file system grows and shrinks automatically, and you only pay for the storage you use.
</details>

---

**16. A company has a custom Linux application that requires a POSIX-compliant shared file system. Which service should they use?**

- A. Amazon S3
- B. Amazon EFS
- C. Amazon FSx for Windows File Server
- D. Amazon EBS

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon EFS provides a standard NFSv4 interface, which is POSIX-compliant. This means it behaves like a traditional Linux file system, supporting file permissions, ownership, and other POSIX attributes that are required by many Linux applications.
</details>

---

**17. What component of AWS Storage Gateway is deployed in the customer's on-premises data center?**

- A. A physical hardware appliance shipped by AWS.
- B. A software agent installed on every application server.
- C. A virtual machine appliance running on a hypervisor like VMware ESXi or Microsoft Hyper-V.
- D. An AWS Direct Connect router.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** To use AWS Storage Gateway, you deploy a virtual appliance (provided by AWS as a VM image) onto a hypervisor in your local data center. This VM acts as the on-premises gateway that communicates with the AWS cloud. AWS also offers a hardware appliance option, but the virtual appliance is the most common deployment method.
</details>

---

**18. Which statement about AWS DataSync is TRUE?**

- A. It is a fully managed file system.
- B. It can only be used to transfer data from S3 to on-premises.
- C. It is a data transfer service that includes features like scheduling, bandwidth throttling, and data validation.
- D. It requires a dedicated AWS Direct Connect connection to function.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** DataSync is a feature-rich data transfer service. Unlike simple copy tools, it's a managed solution that handles the entire workflow, including built-in scheduling, task management, monitoring, and ensuring the data that arrives is identical to the source data. It can operate over Direct Connect or the public internet.
</details>

---

**19. Which AWS file system is the best choice for a "lift-and-shift" migration of a Windows-based application that relies on a shared network drive?**

- A. Amazon FSx for Lustre
- B. Amazon S3 with a custom client
- C. Amazon EFS
- D. Amazon FSx for Windows File Server

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** For a "lift-and-shift" of a Windows application, you want the AWS environment to look as much like the on-premises environment as possible. Amazon FSx for Windows File Server provides a native SMB file share that can integrate with Active Directory, making it the ideal target for applications that expect a traditional Windows network drive.
</details>

---

**20. A user needs to provide iSCSI block storage to their on-premises servers while using S3 as the primary data store. Which gateway configuration should be used?**

- A. File Gateway using NFS
- B. Volume Gateway in Cached mode
- C. Tape Gateway
- D. Volume Gateway in Stored mode

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The requirements are iSCSI (which means Volume Gateway) and using S3 as the primary data store (which means Cached mode). Therefore, Volume Gateway in Cached mode is the correct answer.
</details>

---

**21. Can an Amazon EFS file system be accessed from an on-premises server?**

- A. No, EFS is only accessible from within a VPC.
- B. Yes, if there is a Direct Connect or VPN connection between the on-premises network and the VPC.
- C. No, EFS uses a proprietary protocol that is not compatible with on-premises servers.
- D. Yes, but only if the on-premises server is running Windows.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Yes, EFS can be accessed from on-premises. If you have established network connectivity to your VPC via AWS Direct Connect or a VPN, your on-premises Linux servers can mount the EFS file system just like an EC2 instance can.
</details>

---

**22. Which service is designed to help customers phase out their physical tape backup infrastructure?**

- A. AWS Backup
- B. AWS Storage Gateway - Tape Gateway
- C. Amazon S3 Glacier
- D. AWS DataSync

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Tape Gateway is the purpose-built solution for this. It directly emulates a physical tape library, allowing companies to integrate it with their existing backup software and processes while replacing the underlying physical tapes with highly durable and cost-effective virtual tapes stored in S3 and Glacier.
</details>