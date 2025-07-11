# Step 11: EC2 Basics – AMI & Instance Types

## 🎯 Objective
- [x] Master: **EC2 Basics – AMI & Instance Types**

## 📘 Notes

### **Deep Dive: EC2 Basics – AMI & Instance Types**

Amazon Elastic Compute Cloud (EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers. Understanding the two most fundamental components of an EC2 instance—what software it runs (AMI) and what hardware it runs on (Instance Type)—is essential.

### **1. Amazon Machine Image (AMI)** 💿

An AMI is a pre-configured template for your EC2 instance. It includes the operating system, an initial state of patches, and can also include application software. Essentially, the AMI determines the initial software state of your instance when it is launched.

- **What an AMI Includes:**
    1. A template for the root volume for the instance (e.g., the OS, an application server).
    2. Launch permissions that control which AWS accounts can use the AMI to launch instances.
    3. A block device mapping that specifies the volumes to attach to the instance when it's launched.
- **Sources of AMIs:**
    - **AWS Provided AMIs (Quick Start):** These are official images provided and maintained by AWS. They include popular operating systems like Amazon Linux 2, Ubuntu, Windows Server, etc. They are fully supported and kept up-to-date by AWS.
    - **AWS Marketplace AMIs:** These are pre-configured images sold by third-party vendors. They often include commercial software with the licensing taken care of (e.g., a firewall appliance, a specialized database, or a security scanner). You launch the instance and pay for the software usage through your AWS bill.
    - **Community AMIs:** A vast collection of AMIs shared by other AWS users. Use these with caution, as they are not vetted by AWS. You should only use AMIs from sources you trust.
    - **My AMIs (Custom AMIs):** These are AMIs you create yourself. The typical process is to launch an instance from a trusted AWS AMI, install your own applications, patches, and configurations, and then create a new custom AMI from that instance. This is a powerful best practice for ensuring consistency and faster deployment times.
- **AMI Regionality:** AMIs are a **regional** resource. An AMI created in `us-east-1` can only be used to launch instances in `us-east-1`. If you want to use the same AMI in a different region, you must copy it to that target region.

### **2. EC2 Instance Types** 💻

The instance type determines the hardware of the host computer used for your instance. Each instance type offers a different combination of CPU, memory, storage, and networking capacity. AWS groups them into "families" optimized for different workloads.

- **Understanding the Naming Convention:**
    - Example: `m5.2xlarge`
    - `m`: The instance **family** (in this case, 'm' for "General Purpose").
    - `5`: The instance **generation** (5th generation).
    - `2xlarge`: The instance **size** within the family. As the size increases (e.g., from `large` to `xlarge` to `2xlarge`), the resources (CPU, RAM, network bandwidth) double.
- **Key Instance Families (Must-Know for SAA-C03):**
    - **General Purpose (T, M families):**
        - **T-family (e.g., t2, t3, t4g):** Burstable performance. They provide a baseline level of CPU performance with the ability to "burst" to a higher level when needed. Ideal for low-traffic websites, development/test environments, and small databases where CPU usage is not consistently high. They operate on a system of CPU credits.
        - **M-family (e.g., m5, m6g):** A balance of compute, memory, and networking. They are the workhorses for a wide variety of applications, such as web servers, code repositories, and backend servers for enterprise applications.
    - **Compute Optimized (C family):**
        - **C-family (e.g., c5, c6g):** Highest performance processors. They have a high ratio of CPU power to memory. Ideal for compute-intensive workloads like batch processing, media transcoding, high-performance web servers, scientific modeling, and dedicated gaming servers.
    - **Memory Optimized (R, X, Z families):**
        - **R-family (e.g., r5, r6g):** "R" for RAM. These instances have a large amount of memory relative to their vCPU count. Ideal for memory-intensive applications such as high-performance databases (SQL and NoSQL), distributed web-scale in-memory caches (like Redis or Memcached), and real-time processing of big unstructured data.
        - **X-family:** "X" for extra-large memory. For even larger in-memory databases like SAP HANA.
    - **Storage Optimized (I, D, H families):**
        - **I-family:** "I" for I/O. These instances provide very fast, low-latency local NVMe SSD storage (Instance Store). Ideal for workloads that require high random I/O performance, like NoSQL databases (Cassandra, MongoDB), search engines (Elasticsearch), and data warehousing.
        - **D-family:** "D" for dense storage. These provide massive amounts of local HDD storage. Ideal for distributed file systems like Hadoop/HDFS, and large-scale data warehousing.
    - **Accelerated Computing (P, G, F families):**
        - **P-family:** "P" for Pictures (Graphics). High-end GPUs for general-purpose GPU compute, typically for machine learning and high-performance computing (HPC).
        - **G-family:** "G" for Graphics. Cost-effective GPUs for graphics-intensive applications like video rendering and game streaming.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to deploy a third-party firewall appliance onto AWS. The firewall vendor provides a pre-configured, ready-to-launch image with all the necessary software and licensing included. Where would the company typically find this image?**

- A. In the Community AMIs section.
- B. In the AWS Marketplace.
- C. In the Quick Start AMIs list provided by AWS.
- D. They must create their own custom AMI.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The AWS Marketplace is specifically designed for third-party vendors to sell their software solutions as pre-packaged AMIs. This simplifies the procurement and deployment process, as licensing is handled through AWS. Community AMIs (A) are not vetted, and Quick Start (C) contains OS images provided by AWS, not commercial software.
</details>
    

---

**2. A scientific research team needs to run a set of simulations that are extremely CPU-intensive. The application requires a very high ratio of CPU power to memory. Which EC2 instance family is the most suitable for this workload?**

- A. R5 (Memory Optimized)
- B. T3 (General Purpose - Burstable)
- C. C5 (Compute Optimized)
- D. I3 (Storage Optimized)

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The C-family is specifically engineered for compute-intensive workloads. It offers the highest-performing processors and the best price-performance for tasks that are bound by CPU speed. Memory Optimized (A) would be wasteful, and Burstable (B) would not provide the sustained performance required.
</details>
    

---

**3. A company wants to create a standardized "golden image" for their web server fleet. The image should include the operating system, all necessary security patches, monitoring agents, and the web server software. What is the AWS best practice for creating and using this image?**

- A. Launch a new instance for each deployment and run a script to install all the software.
- B. Create a custom AMI from a fully configured EC2 instance and use this AMI for all future web server deployments.
- C. Find a similar image in the Community AMIs and use it for all deployments.
- D. Use an AWS provided Amazon Linux 2 AMI and hope it contains the necessary software.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Creating a custom AMI is the standard best practice for this scenario. It ensures that every instance launched from it is identical, secure, and compliant from the start. This significantly reduces boot times and eliminates configuration drift compared to running scripts on boot (A). Using Community AMIs (C) is risky as they are not trusted.
</details>
    

---

**4. An application runs a large in-memory database using SAP HANA. The primary performance bottleneck is the amount of available RAM. Which EC2 instance family is specifically designed for such large-scale, memory-intensive workloads?**

- A. M5 (General Purpose)
- B. C5 (Compute Optimized)
- C. X1 (Memory Optimized)
- D. T3 (General Purpose - Burstable)

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** While the R-family is for general memory-optimized workloads, the X-family ("X" for eXtra large memory) is specifically designed for the largest in-memory databases like SAP HANA, offering terabytes of RAM. This makes it the most suitable choice.
</details>
    

---

**5. An AMI is created in the `us-east-1` Region. A solutions architect needs to launch an identical instance in the `eu-west-2` Region. What must be done first?**

- A. Nothing, AMIs are a global resource.
- B. Copy the AMI from `us-east-1` to `eu-west-2`.
- C. Launch an instance in `us-east-1` and then move the instance to `eu-west-2`.
- D. Modify the AMI's launch permissions to include the `eu-west-2` region.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** AMIs are a regional resource. To use an AMI in a different region, you must first use the "Copy AMI" feature to create a copy of it in the target region. Once the copy is complete, it can be used to launch instances in that new region.
</details>
    

---

**6. A company runs a small development blog that has very low traffic overnight but experiences short, unpredictable bursts of traffic during the day. The company wants the most cost-effective solution. Which instance family should they choose?**

- A. M5 (General Purpose)
- B. C5 (Compute Optimized)
- C. T3 (General Purpose - Burstable)
- D. R5 (Memory Optimized)

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The T-family is ideal for workloads with bursty traffic patterns. It provides a low-cost baseline performance and accumulates CPU credits when idle. It then uses these credits to automatically burst to higher CPU performance to handle the unpredictable spikes in traffic, making it the most cost-effective choice for this use case.
</details>
    

---

**7. A data analytics application requires extremely high, low-latency random read/write access to a large dataset stored locally on the instance. Which instance family is designed for this high I/O workload?**

- A. M5, using an EBS gp3 volume.
- B. C5, using an EBS io2 Block Express volume.
- C. I3, using the local instance store NVMe SSDs.
- D. R5, using an EBS st1 volume.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The I-family ("I" for I/O) is specifically designed for I/O-intensive workloads. The key feature is the included local NVMe SSD instance store, which provides millions of IOPS at very low latency, far exceeding the performance of even the fastest EBS volumes for this specific access pattern.
</details>
    

---

**8. What does the "m" in the `m5.large` instance type name signify?**

- A. The size of the instance (Medium).
- B. The instance family (General Purpose).
- C. The region the instance is located in (Multi-region).
- D. The generation of the hardware (Modern).

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The first letter in an instance type name denotes its family. m stands for the "General Purpose" family, which provides a balance of CPU, memory, and network resources.
</details>
    

---

**9. When you create a custom AMI from a running EC2 instance, what is captured as part of the AMI?**

- A. Only the operating system files.
- B. The data on the root volume and any attached EBS data volumes.
- C. Only the data on the attached EBS data volumes.
- D. The RAM state of the instance.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Creating an AMI from an instance involves taking snapshots of the root EBS volume and any other EBS volumes attached to it. These snapshots are then used as the template for the block device mapping of the new AMI. The instance's RAM state is not captured (unless you use the hibernate feature, which is a different process).
</details>
    

---

**10. A machine learning application requires powerful GPUs for model training. Which instance family would be the most appropriate choice?**

- A. C6g (Compute Optimized)
- B. M6g (General Purpose)
- C. R6g (Memory Optimized)
- D. P4d (Accelerated Computing)

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** The P-family is part of the Accelerated Computing group and is specifically equipped with high-performance GPUs (like NVIDIA A100 Tensor Core GPUs) designed for machine learning and high-performance computing (HPC) workloads.
</details>
    

---

**11. A developer wants to try out a new open-source database. They find a pre-configured AMI for it that was published by another developer in the community. What is the most important consideration before using this AMI?**

- A. The cost of the AMI, as community AMIs are always expensive.
- B. The trustworthiness of the publisher, as community AMIs are not vetted by AWS and could contain malware.
- C. The AWS region it was published in.
- D. The instance type it was created from.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Security is the paramount concern when using Community AMIs. Since anyone can publish them, they could be misconfigured or contain malicious software. You should only ever use a community AMI from a publisher you know and trust.
</details>
    

---

**12. How does an `m5.2xlarge` instance compare to an `m5.xlarge` instance?**

- A. It has half the resources (vCPU, RAM).
- B. It has double the resources (vCPU, RAM).
- C. It has the same resources but is a newer generation.
- D. It has the same resources but includes an attached GPU.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Within the same instance family and generation, the size (large, xlarge, 2xlarge, etc.) indicates the scale of resources. Each step up in size typically doubles the vCPU, memory, and often the networking bandwidth and EBS throughput. Therefore, a 2xlarge has double the resources of an xlarge.
</details>
    

---

**13. A company is running a high-performance relational database that benefits greatly from having a large amount of RAM to cache the working set of data. Which instance family is the best fit?**

- A. T4g (Burstable)
- B. C5 (Compute Optimized)
- C. R5 (Memory Optimized)
- D. D3 (Storage Optimized)

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The R-family ("R" for RAM) is designed for memory-intensive workloads. It offers a high ratio of memory to vCPU, making it perfect for caching large datasets in memory to improve the performance of relational and NoSQL databases.
</details>
    

---

**14. An organization needs to store and process several terabytes of data using a Hadoop cluster on AWS. The performance of the cluster is highly dependent on sequential disk throughput. Which instance family is designed for this type of dense storage workload?**

- A. I3 (I/O Optimized)
- B. R5 (Memory Optimized)
- C. M5 (General Purpose)
- D. D2 (Dense Storage Optimized)

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** The D-family ("D" for Dense) provides instances with a very large number of high-capacity local HDD storage drives. This makes them ideal for workloads that need to store and process massive datasets, such as nodes in a Hadoop/HDFS cluster or other distributed file systems.
</details>
    

---

**15. What are the main components of an AMI? (Select TWO)**

- A. A template for the root volume.
- B. An IAM role to attach to the instance.
- C. A block device mapping for attached storage.
- D. The VPC and subnet configuration.
- E. A security group configuration.

<details>
<summary>View Answer</summary>

**Answers: A and C**

**Explanation:** At its core, an AMI consists of a template for the root volume (usually a snapshot of an EBS volume) and a block device mapping, which tells the instance which storage volumes to attach at launch time and how to configure them. IAM roles, VPC/subnet settings, and security groups are all specified separately during the instance launch process, not as part of the AMI itself.

</details>
    

---

**16. Which of the following is an example of an AWS Provided AMI?**

- A. A custom image containing a company's proprietary web application.
- B. A Windows Server 2022 image provided by AWS.
- C. A Debian image shared by a user in the community.
- D. A Barracuda Firewall appliance image.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS provides and maintains a set of "Quick Start" AMIs for popular operating systems like Amazon Linux, Windows, Ubuntu, and SUSE. These are considered official and are fully supported. The custom image (A), community image (C), and Marketplace image (D) come from other sources.

</details>
    

---

**17. A T3 instance has used up all of its CPU credits. What happens to its performance?**

- A. The instance is automatically terminated.
- B. The instance's performance is throttled down to its baseline level.
- C. The instance is converted to an M5 instance automatically.
- D. The instance continues to burst at high performance, but at an additional cost if T3 Unlimited mode is enabled.

<details>
<summary>View Answer</summary>

**Answer: B or D**

**Explanation:** Both answers can be correct depending on the configuration, which is a key nuance of the T-family.

- **Answer B** describes the default "Standard" mode: Once credits are depleted, the CPU performance is throttled to its low baseline level until it earns more credits.
- **Answer D** describes "Unlimited" mode: If enabled, the instance can continue to burst even after its credits are gone, and you pay a small, flat hourly rate for the extra CPU usage. For the SAA-C03 exam, you should be aware of both possibilities.

</details>

---

**18. You are analyzing an instance type named `c6g.large`. What does the "g" signify?**

- A. The instance has an attached GPU.
- B. The instance uses an AWS Graviton (ARM-based) processor.
- C. The instance is a general-purpose type.
- D. The instance has a faster network connection.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The "g" in an instance type name indicates that it is powered by AWS's own custom-built Graviton processors, which are based on the ARM architecture. These often provide better price-performance for many workloads compared to their x86-based counterparts.

</details>
    

---

**19. When choosing between instance families for a new application, what is the primary factor to consider?**

- A. The AWS Region you are deploying in.
- B. The nature of the application's workload (e.g., CPU-bound, memory-bound, or I/O-bound).
- C. The cost of the AMI you plan to use.
- D. The number of security groups you need to attach.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The most important factor in selecting an instance family is matching the hardware capabilities to the application's performance characteristics. Choosing the wrong family (e.g., a compute-optimized instance for a memory-intensive database) leads to poor performance and wasted money.

</details>
    

---

**20. What is an AMI used for?**

- A. To provide the hardware specification for an EC2 instance.
- B. To provide the software template, including the operating system, for an EC2 instance.
- C. To provide a secure network connection to an EC2 instance.
- D. To provide the IAM permissions for an EC2 instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An AMI is a template that defines the initial software configuration of an instance. The hardware is defined by the instance type, the network connection by the VPC/subnet/security group, and the permissions by the IAM role.

</details>
