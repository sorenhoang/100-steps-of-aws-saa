# Step 12: EC2 Pricing Models

## 🎯 Objective

- [x] Master: **EC2 Pricing Models**

## 📘 Notes

### **Deep Dive: EC2 Pricing Models**

Choosing the right EC2 pricing model is crucial for optimizing costs. AWS offers several models designed to match different use cases, from steady-state predictable workloads to fault-tolerant, interruptible jobs. A solutions architect must be able to recommend the most cost-effective model for a given scenario.

### **1. On-Demand** 🕒

This is the most flexible and straightforward pricing model. You pay for compute capacity by the second (with a 60-second minimum) with no long-term commitments.

- **Best for:**
  - Applications with short-term, spiky, or unpredictable workloads that cannot be interrupted.
  - Applications being developed or tested on EC2 for the first time.
  - Users who prefer the flexibility of starting and stopping instances without a financial penalty or commitment.
- **Pros:** Maximum flexibility, no upfront payment, no long-term contract.
- **Cons:** Highest cost per hour.

### **2. Savings Plans** 💰

Savings Plans are a flexible pricing model that offers lower prices compared to On-Demand, in exchange for a commitment to a consistent amount of usage (measured in $/hour) for a 1 or 3-year term.

- **How it works:** You commit to spending a certain amount (e.g., $10/hour) on compute. Any usage up to that commitment is charged at the lower Savings Plans rate. Any usage beyond the commitment is charged at the On-Demand rate.
- **Types:**
  - **Compute Savings Plans (Most Flexible):** These automatically apply to EC2 instance usage regardless of instance family, size, OS, tenancy, or AWS Region. They also apply to AWS Fargate and Lambda usage. This is the best option for users who want maximum flexibility and are comfortable with the commitment.
  - **EC2 Instance Savings Plans:** These provide the lowest prices (up to 72% discount) in exchange for a commitment to a specific instance **family** (e.g., M5) in a specific **Region**. You can still change the instance size (e.g., from `m5.large` to `m5.2xlarge`) or OS within that family and region.
- **Best for:** Workloads with a predictable, consistent amount of compute usage over the long term.

### **3. Reserved Instances (RIs)** rezervasyon

This is the more traditional discount model. You purchase a reservation for a specific instance configuration and receive a significant discount (up to 72%) compared to On-Demand pricing.

- **Key Attributes:** You reserve based on instance type, region, tenancy, and operating system.
- **Types:**
  - **Standard RI:** Offers the highest discount but can only be modified in limited ways (e.g., change the size within the same instance family). Cannot be exchanged.
  - **Convertible RI:** Offers a lower discount but allows you to exchange the reservation for another Convertible RI with different attributes (e.g., changing from a C5 to an M5 instance).
- **Scope:** RIs can be scoped to a specific Availability Zone (capacity reservation) or to a Region (more flexible).
- **Savings Plans vs. RIs:** For most use cases, **Savings Plans are now recommended over RIs** due to their increased flexibility. RIs are still used for specific needs like guaranteeing capacity in a specific AZ.

### **4. Spot Instances** 📉

Spot Instances allow you to bid on and use spare, unused EC2 capacity at steep discounts—often up to 90% off the On-Demand price.

- **The Catch:** The core characteristic of a Spot Instance is that **AWS can interrupt and terminate your instance with a two-minute warning** if they need the capacity back or if the spot price exceeds your maximum bid.
- **Best for:**
  - Workloads that are fault-tolerant, stateless, and can handle interruptions gracefully.
  - Batch processing jobs, data analysis, image rendering, and high-performance computing (HPC).
  - Testing and development environments that don't need to be up 24/7.
- **NOT for:** Critical databases, production web servers, or any workload that cannot be interrupted.
- **Best Practices:** Use Spot Instances as part of an Auto Scaling Group mixed with On-Demand instances (a "Spot Fleet") to maintain application availability while optimizing cost.

### **5. Dedicated Hosts** 🖥️

A Dedicated Host is a physical EC2 server dedicated for your use. This is the most extreme form of tenancy and is used to address compliance requirements.

- **How it works:** You have control over instance placement on the physical server. This allows you to use your existing server-bound software licenses (e.g., Windows Server, SQL Server) and can help you meet strict compliance regulations (e.g., HIPAA, PCI DSS) that require physical isolation.
- **Billing:** Billed per host, not per instance. It is the most expensive option.
- **Dedicated Instances:** A slightly less extreme option where your instances run on hardware dedicated to you, but you don't have control over the specific host. Both are expensive and used for compliance or licensing reasons. For the exam, Dedicated Host gives you more visibility and control over the underlying physical server.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A media company runs a daily video transcoding job that takes about four hours to complete. The job can be stopped and restarted without losing progress. The company wants to run this job at the lowest possible cost. Which EC2 pricing model should they use?**

- A. On-Demand Instances
- B. Reserved Instances
- C. Spot Instances
- D. Dedicated Hosts

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is a perfect use case for Spot Instances. The workload is not time-critical and is fault-tolerant (can be stopped and restarted). This allows the company to take advantage of the massive discounts (up to 90%) offered by Spot Instances. If the instance is terminated, the job can simply be restarted later on a new Spot Instance.

</details>


---

**2. A large enterprise runs a steady-state web application that will be active 24/7 for the next three years. They want to achieve the maximum possible discount on their EC2 costs but want the flexibility to change instance sizes within the M5 family (e.g., from `m5.large` to `m5.xlarge`) in the `us-east-1` region. Which pricing model offers the best discount while meeting these requirements?**

- A. On-Demand
- B. Compute Savings Plan
- C. EC2 Instance Savings Plan
- D. Spot Instances

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** For a predictable, long-term workload, a commitment-based plan is best. An EC2 Instance Savings Plan provides the highest discount (up to 72%) by committing to a specific instance family (M5) in a specific region (us-east-1). This plan explicitly allows the flexibility to change the instance size within that family, perfectly matching the requirements. A Compute Savings Plan (B) is more flexible but offers a slightly lower discount.

</details>


---

**3. A startup is launching a new application. They are unsure about the future traffic patterns and want to avoid any long-term commitments or upfront payments. Which pricing model is most appropriate for their launch phase?**

- A. Reserved Instances
- B. On-Demand Instances
- C. Savings Plans
- D. Spot Fleet

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** On-Demand is designed for flexibility and unpredictable workloads. It allows the startup to pay by the second for the capacity they use without being locked into any contracts. This is ideal when traffic is unknown and they need the ability to scale up or down freely.

</details>


---

**4. A company needs to run a critical production database that must be available 24/7 and cannot tolerate any interruptions. Which pricing model is LEAST suitable for this workload?**

- A. On-Demand Instances
- B. Savings Plans
- C. Reserved Instances
- D. Spot Instances

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Spot Instances are fundamentally unsuitable for critical, uninterruptible workloads. The core characteristic of a Spot Instance is that AWS can terminate it with a two-minute warning. Running a critical production database on a Spot Instance would be a major architectural flaw, risking data loss and downtime.

</details>


---

**5. A financial services company has a regulatory requirement to run their application on a physically isolated server. They also need to bring their own per-socket software licenses to AWS. Which EC2 purchasing option should they choose?**

- A. Dedicated Instances
- B. On-Demand Instances in a private subnet
- C. Reserved Instances with a regional scope
- D. Dedicated Hosts

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Dedicated Hosts are physical servers dedicated to a single customer. This option provides the highest level of isolation and, crucially, gives you visibility into the underlying hardware (sockets, cores), which is necessary for many "Bring Your Own License" (BYOL) scenarios. Dedicated Instances (A) provide hardware isolation but less visibility and control than Dedicated Hosts.

</details>


---

**6. A company has committed to a 3-year Compute Savings Plan. They are currently running a mix of C5 (Compute Optimized) and M5 (General Purpose) instances in `us-east-1`. They now want to deploy a new workload using T3 instances in `eu-west-1`. How will the Savings Plan apply?**

- A. The plan will only apply to the C5 and M5 instances in `us-east-1`.
- B. The plan will apply to the C5, M5, and T3 instances across both regions.
- C. The plan will not apply to any instances because the instance types and regions are different.
- D. The plan will only apply to the T3 instances in `eu-west-1`.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This question highlights the flexibility of a Compute Savings Plan. It automatically applies to instance usage regardless of family, size, OS, tenancy, or Region. The discount will apply seamlessly to the compute usage from all three instance types in both regions.

</details>


---

**7. What is the primary characteristic of Spot Instances?**

- A. They offer a capacity reservation in a specific Availability Zone.
- B. They can be terminated by AWS with a two-minute warning if capacity is needed elsewhere.
- C. They provide the most flexibility with no upfront commitment.
- D. They are physical servers dedicated to a single customer.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The defining trade-off for the huge discount of a Spot Instance is its interruptible nature. You get to use spare AWS capacity, but you must design your application to handle the possibility that the instance will be reclaimed by AWS at any time.

</details>


---

**8. When comparing a Standard Reserved Instance to a Convertible Reserved Instance, which statement is true?**

- A. Standard RIs offer a lower discount but more flexibility to change instance families.
- B. Convertible RIs offer a higher discount but cannot be exchanged.
- C. Standard RIs offer a higher discount but have limited modification options.
- D. Convertible RIs can be sold on the Reserved Instance Marketplace.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The trade-off is between discount and flexibility. Standard RIs give you the larger discount but are more rigid; you can typically only change the instance size within the same family. Convertible RIs give you a smaller discount but allow you to exchange them for a different RI with a different instance family, OS, or tenancy.

</details>


---

**9. For which of the following workloads would On-Demand pricing be the most appropriate choice?**

- A. A long-term, steady-state application running for 3 years.
- B. A stateless, fault-tolerant batch processing job.
- C. A short-term development project with an uncertain duration and workload pattern.
- D. A workload that requires a physically isolated server for compliance.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** On-Demand shines when flexibility is key and commitment is undesirable. A short-term project with unknown characteristics is a perfect fit, as you can start and stop instances as needed without any financial penalty.

</details>


---

**10. What is a key benefit of using a Savings Plan over a Reserved Instance?**

- A. Savings Plans can provide a capacity reservation.
- B. Savings Plans offer much higher discounts than RIs.
- C. Savings Plans are more flexible and automatically apply to usage across instance families and regions (with Compute Savings Plans).
- D. Savings Plans can be sold on a marketplace if no longer needed.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The primary advantage of Savings Plans is their flexibility. Instead of being tied to a specific instance type, you commit to a dollar amount of usage per hour. A Compute Savings Plan then automatically finds the most discounted resources to apply this to, simplifying management and reducing waste as your usage patterns change. RIs can be sold on the marketplace; Savings Plans cannot.

</details>


---

**11. Which two pricing models require a 1 or 3-year commitment? (Select TWO)**

- A. Spot Instances
- B. Reserved Instances
- C. On-Demand Instances
- D. Dedicated Hosts
- E. Savings Plans

<details>
<summary>View Answer</summary>

**Answers: B and E**

**Explanation:** Both Reserved Instances and Savings Plans derive their discounts from a customer's commitment to use a certain amount of compute for a 1 or 3-year term. On-Demand and Spot have no commitment. Dedicated Hosts are typically purchased with a commitment, but RIs and Savings Plans are the primary models defined by their terms.

</details>


---

**12. An application is designed using a "Spot Fleet." What does this architecture typically involve?**

- A. Using only Dedicated Hosts for all instances.
- B. Running a mix of Spot Instances and On-Demand Instances to balance cost savings and availability.
- C. Exclusively using Reserved Instances for all nodes in a cluster.
- D. A fleet of instances that can only run in a single Availability Zone.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Spot Fleet (or an Auto Scaling Group with a mixed instances policy) is a best practice for using Spot. It allows you to define a target capacity and have the fleet fulfill that capacity using the cheapest available Spot Instances. You can also specify a certain percentage of the fleet to be On-Demand, ensuring that the application remains available even if all Spot capacity is temporarily reclaimed.

</details>


---

**13. A company has purchased an EC2 Instance Savings Plan for the `c5` family in `eu-west-1`. They are currently running a `c5.large` instance. If they stop the `c5.large` and launch a `c5.xlarge` instance instead, what happens?**

- A. The Savings Plan will not apply to the new instance.
- B. The Savings Plan benefit will automatically apply to the new `c5.xlarge` instance.
- C. They must first convert their Savings Plan to a `c5.xlarge` plan.
- D. The `c5.xlarge` instance will be billed at the Spot price.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An EC2 Instance Savings Plan is flexible within its committed family and region. Since both instances belong to the c5 family and are in eu-west-1, the plan will apply seamlessly. The plan is based on a normalized usage amount, so it will cover the c5.xlarge usage up to the committed dollar amount.

</details>


---

**14. A user wants to guarantee they will have capacity for 10 `m5.large` instances in the `us-east-1a` Availability Zone for a major upcoming event. What is the BEST way to ensure this capacity is reserved for them?**

- A. Purchase a Regional Reserved Instance.
- B. Purchase a Zonal Reserved Instance for `us-east-1a`.
- C. Purchase a Compute Savings Plan.
- D. Launch the instances using the Spot pricing model.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Only a Zonal Reserved Instance provides a capacity reservation. When you purchase an RI and scope it to a specific Availability Zone (like us-east-1a), AWS guarantees that you will be able to launch an instance of that type in that AZ. A Regional RI or Savings Plan provides a billing discount but does not guarantee capacity.

</details>


---

**15. Under which pricing model do you pay the least for your EC2 instances, assuming your workload can be interrupted?**

- A. On-Demand
- B. Reserved Instances (3-year, all upfront)
- C. Savings Plans (3-year, all upfront)
- D. Spot Instances

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Spot Instances offer the largest potential discount, often up to 90% off the On-Demand price. While a 3-year, all-upfront RI or Savings Plan offers a very large discount (up to 72%), it cannot match the potential savings of using spare Spot capacity.

</details>


---

**16. What happens when a Spot Instance is terminated by AWS?**

- A. You are charged for the full hour, regardless of when it was terminated.
- B. You are not charged for the partial hour of usage before termination.
- C. You are charged a termination penalty fee.
- D. The instance is automatically relaunched as an On-Demand instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** One of the benefits of Spot pricing is that if AWS interrupts and terminates your instance, you are not charged for the final partial hour of usage. If you terminate the instance yourself, you pay for the seconds used, just like with On-Demand.

</details>


---

**17. What is the primary difference between a Dedicated Host and a Dedicated Instance?**

- A. Dedicated Hosts are more expensive than Dedicated Instances.
- B. Dedicated Hosts give you host-level visibility and control for licensing, while Dedicated Instances provide instance-level isolation without host affinity.
- C. Dedicated Instances can be purchased with a Savings Plan, but Dedicated Hosts cannot.
- D. There is no difference; the terms are interchangeable.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the key distinction. A Dedicated Host gives you an entire physical server and you can control which instances are placed on it, which is essential for certain BYOL scenarios. A Dedicated Instance runs on hardware that is dedicated to you, but AWS controls the instance placement automatically. You get instance isolation without the host-level control.

</details>


---

**18. A company has a highly variable and unpredictable compute workload. They have some fault-tolerant components and some critical components. What would be a good cost-optimization strategy?**

- A. Run all components on On-Demand instances.
- B. Purchase a 3-year Savings Plan that covers the peak workload.
- C. Use an Auto Scaling group with a mixed instances policy, using On-Demand for critical components and Spot Instances for fault-tolerant components.
- D. Run all components on Dedicated Hosts.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is a sophisticated cost-optimization strategy. By using a mixed instances policy, you can specify that a certain percentage of your fleet should be reliable On-Demand instances (for the critical parts) while the rest can be low-cost Spot Instances (for the fault-tolerant parts). This provides a blended approach that maximizes cost savings without sacrificing the availability of the core application.

</details>


---

**19. A company buys a Convertible RI. Six months later, they realize a newer generation instance type offers better performance. What can they do?**

- A. They are locked into the original RI for the full term.
- B. They can sell the Convertible RI on the RI Marketplace.
- C. They can exchange the Convertible RI for another Convertible RI with the new instance type.
- D. They can convert the RI into a Savings Plan.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The key feature of a Convertible RI is the ability to exchange it for another one of equal or greater value. This allows companies to adapt to new technology and changing needs during the RI term, which is not possible with a Standard RI.

</details>


---

**20. You have a workload that must run continuously for the next 6 months. After that, it will be shut down permanently. Which pricing model is most suitable?**

- A. A 1-year Standard Reserved Instance.
- B. A 3-year Savings Plan.
- C. On-Demand Instances.
- D. Spot Instances.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Since the workload duration is only 6 months, it does not make sense to enter into a 1-year or 3-year commitment required by RIs (A) or Savings Plans (B). Spot Instances (D) are too risky for a workload that "must run continuously." Therefore, On-Demand is the best choice, providing the required reliability without a long-term contract that would extend beyond the project's lifecycle.

</details>
