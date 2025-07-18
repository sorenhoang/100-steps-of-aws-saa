# Step 67: Disaster Recovery Strategies

## 🎯 Objective

- [x] Master: **Disaster Recovery Strategies**

## 📘 Notes

### **Architectural Deep Dive: Disaster Recovery Strategies**

Disaster Recovery (DR) is about preparing for and recovering from a disaster that impacts your IT infrastructure. In the context of AWS, a "disaster" is typically defined as an event that makes an entire AWS Region unavailable. Your DR strategy is a trade-off between cost and complexity versus how quickly you can recover and how much data you can afford to lose.

---

### **Key DR Metrics**

- **RTO (Recovery Time Objective):** How quickly do you need to be back online after a disaster? This is your maximum acceptable **downtime**.
- **RPO (Recovery Point Objective):** How much data can you afford to lose? This is the maximum acceptable amount of time's worth of data loss, measured from the disaster event backwards.

---

### **The Four DR Strategies (From Slowest/Cheapest to Fastest/Most Expensive)**

1. **Backup and Restore:**
   - **How it works:** You regularly back up your data and infrastructure to another region. In a disaster, you provision a brand new environment in the DR region from these backups.
   - **AWS Services:** AWS Backup, EBS Snapshots copied to the DR region, RDS Snapshots copied to the DR region, S3 Cross-Region Replication, Infrastructure as Code (CloudFormation).
   - **RTO:** High (hours to days).
   - **RPO:** High (minutes to hours, depending on backup frequency).
   - **Analogy:** You have blueprints and all your materials in a safe. After a disaster, you use them to build a new house from scratch.
2. **Pilot Light:**
   - **How it works:** A small, minimal version of your core infrastructure is always running in the DR region. This "pilot light" typically includes your database, which is replicated from the primary region. The application servers are turned off. In a disaster, you "turn on the full flame" by scaling up the application servers and updating DNS.
   - **AWS Services:** RDS Cross-Region Read Replicas, Infrastructure as Code (CloudFormation), pre-configured AMIs.
   - **RTO:** Medium (tens of minutes to hours).
   - **RPO:** Low (seconds to minutes, equal to the replica lag).
   - **Analogy:** You have a new house with the electricity and water running, but all the lights are off and the furniture is covered. You just need to flip the switches to make it livable.
3. **Warm Standby:**
   - **How it works:** A scaled-down, but fully functional, version of your application is always running in the DR region on a smaller fleet of instances. The database is replicated. In a disaster, you scale up the fleet to handle the full production load and update DNS.
   - **AWS Services:** ELB, Auto Scaling Groups (with a small number of instances), RDS Cross-Region Read Replicas, Route 53.
   - **RTO:** Low (minutes).
   - **RPO:** Very Low (seconds to minutes).
   - **Analogy:** You have a fully furnished, functional house, but it's a smaller "vacation home" version. During a disaster, you can quickly move in and expand it.
4. **Multi-Site Active-Active:**
   - **How it works:** Your application is fully deployed and running with production capacity in **both** the primary and DR regions. Users are actively served from both regions. DNS (often with Latency-Based or Geolocation Routing) distributes traffic between the regions. In a disaster, you simply update DNS to route all traffic to the healthy region.
   - **AWS Services:** Route 53, ELB, ASG, and a multi-region data replication strategy (like Aurora Global Database or DynamoDB Global Tables).
   - **RTO:** **Near-zero**.
   - **RPO:** **Near-zero**.
   - **Analogy:** You own two identical, fully staffed houses in different cities and you live in both of them simultaneously. If one is unavailable, you just use the other one with no interruption.

---

### **SAA-C03 Style Scenario Questions**

**1. A company has a business requirement to be able to recover from a regional disaster within 15 minutes. They also cannot afford to lose more than 1 minute of data. Which disaster recovery strategy BEST meets these RTO and RPO requirements?**

- A. Backup and Restore
- B. Pilot Light
- C. Warm Standby
- D. Multi-Site Active-Active

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Warm Standby is the best fit. An RTO of 15 minutes is too fast for Backup and Restore or Pilot Light, which take longer to provision and scale up infrastructure. A Multi-Site Active-Active (D) would also work but is more complex and expensive than necessary. A Warm Standby provides a scaled-down, running environment that can be quickly scaled up to meet the low RTO, and a continuously replicated database meets the low RPO.

</details>

---

**2. A company archives its data nightly to S3. They have a disaster recovery plan that states in the event of a regional outage, they will use CloudFormation to provision a new environment in a different region and restore their data from the S3 backups. This strategy is the least expensive but has the longest recovery time. What is this DR strategy called?**

- A. Pilot Light
- B. Warm Standby
- C. Backup and Restore
- D. Multi-Site Active-Active

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This describes the Backup and Restore strategy. It involves backing up data and infrastructure templates to a DR location but does not provision any active resources until a disaster occurs. It has the lowest cost but the highest RTO.

</details>

---

**3. What are RTO and RPO?**

- A. RTO is the amount of data you can lose; RPO is the time it takes to recover.
- B. RTO is the Recovery Time Objective; RPO is the Recovery Point Objective.
- C. RTO is the Regional Tolerance Objective; RPO is the Replication Point Objective.
- D. RTO is the cost of recovery; RPO is the cost of data loss.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** RTO (Recovery Time Objective) is the target time for recovering your application (downtime). RPO (Recovery Point Objective) is the target for the maximum amount of data loss, measured in time.

</details>

---

**4. A company is using the Pilot Light disaster recovery strategy. What does this typically involve in the DR region?**

- A. A fully scaled-out, active copy of the production environment.
- B. Only backups of data and infrastructure templates stored in S3.
- C. The core infrastructure, such as a replicated database, is running, but the application servers are stopped or running at minimal capacity.
- D. A DNS record that is ready to be switched over.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Pilot Light strategy involves keeping the most critical and time-consuming-to-provision parts of the infrastructure (the "pilot light") running in the DR region. This is almost always the database, which is kept in sync via replication. The application servers are defined in an ASG with a desired count of zero or are stopped, ready to be quickly launched and scaled up when needed.

</details>

---

**5. Which disaster recovery strategy provides the lowest (near-zero) RTO and RPO?**

- A. Backup and Restore
- B. Pilot Light
- C. Warm Standby
- D. Multi-Site Active-Active

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Multi-Site Active-Active provides the lowest RTO and RPO because your application is fully deployed and running with production capacity in both the primary and DR regions. Users are actively served from both regions, so in a disaster, you simply update DNS to route all traffic to the healthy region with near-zero downtime and data loss.

</details>

---

### **Key Exam Tips & Tricks**

- **Map RTO/RPO to Strategy:** This is the most important skill for DR questions. The exam will give you a business requirement for RTO/RPO, and you must choose the right strategy.
  - **RTO in Hours/Days:** Almost always **Backup and Restore**.
  - **RTO in Tens of Minutes:** Leans towards **Pilot Light**.
  - **RTO in Minutes/Seconds:** Points to **Warm Standby** or **Multi-Site**.
  - **RTO/RPO of Near-Zero:** The answer is always **Multi-Site Active-Active**.
- **Cost is a Clue:** The four strategies exist on a spectrum of cost and complexity.
  - `Backup and Restore (Lowest Cost)` -> `Pilot Light` -> `Warm Standby` -> `Multi-Site (Highest Cost)`
  - If a question emphasizes the need for a "cost-effective" DR solution, it's likely pointing you towards Backup and Restore or Pilot Light. If it emphasizes business continuity at any cost, it's pointing towards Warm Standby or Multi-Site.
- **Identify the Core AWS Services:** Be able to associate the key services with each strategy.
  - **Backup and Restore:** AWS Backup, S3 Cross-Region Replication, CloudFormation.
  - **Pilot Light:** RDS Cross-Region Read Replica, ASG with min/desired=0.
  - **Warm Standby:** RDS Cross-Region Read Replica, ASG with min/desired > 0 (but small).
  - **Multi-Site Active-Active:** Aurora Global Database, DynamoDB Global Tables, Route 53 (Latency/Geolocation routing).
- **Read the Question Carefully:** Don't confuse High Availability (across AZs) with Disaster Recovery (across Regions). If a question talks about surviving an **AZ failure**, the answer is likely Multi-AZ RDS or an ELB/ASG setup. If it talks about surviving a **Regional outage**, the answer will be one of these four DR strategies.
