# Step 66: High Availability vs Fault Tolerance

## 🎯 Objective

- [x] Master: **High Availability vs Fault Tolerance**

## 📘 Notes

### **Architectural Deep Dive: High Availability vs. Fault Tolerance**

While often used interchangeably, High Availability (HA) and Fault Tolerance (FT) represent two different levels of operational excellence and system reliability. As a solutions architect, you must choose the right approach based on business requirements, as achieving a higher level of reliability typically comes with a higher cost.

---

### **1. High Availability (HA) 🏃‍♂️**

**High Availability** means designing systems to **minimize downtime** and ensure a high level of performance by eliminating single points of failure. HA systems are designed to be resilient, but they accept that a brief period of downtime or service degradation may occur during a failure while the system recovers.

- **Core Principle:** Redundancy and rapid recovery.
- **Analogy:** A car with a **spare tire**. If you get a flat, the car is temporarily out of service. You have to stop, perform a recovery action (change the tire), and then you can continue your journey. There's a small amount of downtime, but you don't have to call a tow truck.
- **How it's measured:** Often by "nines" of uptime (e.g., 99.99% availability).
- **Typical AWS Architecture:** An **Elastic Load Balancer (ELB)** distributing traffic across an **Auto Scaling Group (ASG)** of EC2 instances spread across **multiple Availability Zones**.
  - If an instance fails, the ELB stops sending traffic to it, and the ASG launches a replacement. The service remains available, but there might be a brief period of reduced capacity.
  - If an entire AZ fails, the ELB routes traffic to the instances in the healthy AZs.

---

### **2. Fault Tolerance (FT) 💪**

**Fault Tolerance** is the ability of a system to **continue operating without interruption** when one or more of its components fail. The goal of a fault-tolerant system is to prevent failures from causing any downtime at all.

- **Core Principle:** Built-in redundancy with instantaneous, automatic failover.
- **Analogy:** A car with **run-flat tires**. If one tire is punctured, the car continues to drive without stopping. The driver might see a warning light, but the system automatically handles the failure without any interruption to the journey. There is **zero downtime**.
- **How it's measured:** The ability to withstand the failure of `N` components.
- **Typical AWS Architecture:** **Amazon RDS in a Multi-AZ configuration**.
  - You have a primary database and a synchronous, hot standby in another AZ.
  - If the primary database instance or its entire AZ fails, RDS automatically and instantly fails over to the standby.
  - The failover is handled by a DNS update, and from the application's perspective (with proper retry logic), the service continues to operate with only a brief pause, not a full outage. The same applies to services like S3, which has fault tolerance built-in by storing data across 3 AZs.

---

### **SAA-C03 Style Scenario Questions**

**1. A company needs to design a database architecture that can withstand the complete failure of a single Availability Zone with no data loss and minimal interruption. Which AWS solution provides this level of fault tolerance?**

- A. An EC2 instance running a database with frequent EBS snapshots.
- B. An Amazon RDS instance deployed in a Single-AZ configuration.
- C. An Amazon RDS instance deployed in a Multi-AZ configuration.
- D. An RDS Read Replica in the same region.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the primary use case for RDS Multi-AZ. It provides a fault-tolerant setup by maintaining a synchronous hot standby in a different AZ. If the primary AZ fails, RDS automatically fails over to the standby with no data loss (RPO of zero) and minimal downtime.

</details>

---

**2. An architecture uses an Application Load Balancer to distribute traffic to an Auto Scaling group of EC2 instances across three Availability Zones. If one of the EC2 instances fails its health check, the system continues to operate, albeit with slightly reduced capacity until a replacement is launched. What does this design demonstrate?**

- A. Fault Tolerance
- B. High Availability
- C. Elasticity
- D. A hybrid architecture

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic High Availability pattern. The system is resilient to the failure of a single component (the EC2 instance) and does not go down completely. However, there is a brief period of reduced capacity, meaning the failure had a minor, temporary impact. A fault-tolerant system would have had zero performance impact.

</details>

---

**3. What is the fundamental difference between High Availability and Fault Tolerance?**

- A. High Availability focuses on cost, while Fault Tolerance focuses on performance.
- B. High Availability aims to minimize downtime, while Fault Tolerance aims to ensure zero downtime.
- C. High Availability requires a multi-region setup, while Fault Tolerance is single-region.
- D. High Availability uses synchronous replication, while Fault Tolerance uses asynchronous replication.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the core distinction. High Availability (HA) accepts that a small amount of downtime is permissible during the recovery process. Fault Tolerance (FT) is a higher standard where the system is designed to continue operating without any user-perceptible interruption even when a component fails.

</details>

---

**4. A company needs to store critical documents and requires that the storage system be able to withstand the loss of two entire data centers simultaneously without any data loss. Which AWS service provides this level of fault tolerance natively?**

- A. Amazon EBS
- B. Amazon EFS
- C. Amazon S3 Standard
- D. An EC2 instance with an Instance Store.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Amazon S3 Standard is inherently fault-tolerant. It automatically and transparently stores your data across a minimum of three geographically distinct Availability Zones. This architecture is designed to sustain the concurrent loss of two data centers (AZs) without losing your data.

</details>

---

**5. Which of the following analogies best describes a High Availability system?**

- A. A car with run-flat tires that continue to work after a puncture.
- B. A car with a spare tire in the trunk.
- C. A single, highly reliable and expensive car.
- D. A car that can drive on both land and water.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The spare tire analogy perfectly represents High Availability. A failure (flat tire) causes a brief period of downtime while a recovery action is performed (changing the tire). The system is resilient and can recover, but the failure is not completely transparent. The run-flat tire (A) is the analogy for Fault Tolerance.

</details>

---

### **Key Exam Tips & Tricks**

- **Keywords are Everything:** The exam will use specific language to guide you.
  - If you see phrases like **"minimize downtime," "resilient," "recover quickly,"** or descriptions of systems that can handle an instance failure by launching a replacement, think **High Availability**. The key is that there's a _recovery process_.
  - If you see phrases like **"no downtime," "withstand failure without interruption," "no impact,"** or descriptions of automatic, instantaneous failover, think **Fault Tolerance**. The key is that the failure is handled _transparently_.
- **Service Mapping:** Associate specific AWS architectures with these concepts:
  - **High Availability (HA):** ELB + Auto Scaling Group across multiple AZs.
  - **Fault Tolerance (FT):** RDS Multi-AZ, S3, DynamoDB, an ELB itself (as it's managed and redundant). Services that have "Multi-AZ" in their name are typically providing fault tolerance.
- **Cost vs. Reliability:** Remember the trade-off. Fault Tolerance is generally more expensive than High Availability because it requires maintaining hot, synchronous, and often idle standby resources. If a question mentions cost-sensitivity, a High Availability solution might be a better choice than a fully Fault Tolerant one.
- **RPO and RTO:**
  - **Fault Tolerant** systems aim for a Recovery Point Objective (RPO) and Recovery Time Objective (RTO) of near-zero.
  - **Highly Available** systems have a low RPO/RTO (seconds to minutes) but not zero. If a question specifies an RTO of a few minutes, it's describing an HA system.
