# Step 68: Scalability Patterns – Vertical & Horizontal

## 🎯 Objective

- [x] Master: **Scalability Patterns – Vertical & Horizontal**

## 📘 Notes

### **Architectural Deep Dive: Scalability Patterns – Vertical & Horizontal**

**Scalability** is the ability of a system to handle a growing amount of work by adding resources. In the cloud, this means being able to seamlessly add more compute, memory, or networking capacity to meet demand. Elasticity, a related concept, is the ability to scale _out_ and _in_ automatically. The two primary methods for achieving scalability are vertical and horizontal.

---

### **1. Vertical Scaling (Scaling Up/Down)** ⬆️

Vertical scaling means increasing the resources of a **single instance**. You make the server bigger and more powerful.

- **How it works:** You increase the capacity of an existing machine. On AWS, this means stopping an EC2 instance, changing its instance type to one with more CPU/RAM (e.g., from a `t3.large` to a `t3.2xlarge`), and then starting it again.
- **Analogy:** Swapping out the engine in your car for a more powerful one. You still have only one car, but it's much faster.
- **Pros:**
  - **Simplicity:** It can be very simple to implement, especially for monolithic applications that are not designed to be distributed.
- **Cons:**
  - **Downtime Required:** It almost always requires stopping and restarting the instance, causing downtime.
  - **Hard Limit:** There is an upper limit. Eventually, you will reach the largest EC2 instance type available, and you cannot scale any further.
  - **Single Point of Failure:** It does not improve availability. You still have only one server, which can fail.

---

### **2. Horizontal Scaling (Scaling Out/In)** ➡️

Horizontal scaling means increasing the number of instances to distribute the workload. You add more servers to your resource pool.

- **How it works:** You add more machines to your fleet. On AWS, this is achieved by using an **Auto Scaling Group (ASG)** to automatically launch new EC2 instances behind an **Elastic Load Balancer (ELB)**.
- **Analogy:** Instead of making your one car faster, you add more identical cars to your fleet to handle more passengers.
- **Pros:**
  - **High Availability:** By distributing instances across multiple Availability Zones, horizontal scaling inherently improves fault tolerance. If one instance fails, the others continue to operate.
  - **Elasticity:** It's the foundation of cloud elasticity. You can automatically scale out to handle peak load and scale back in during quiet periods to save money.
  - **Virtually Unlimited Scalability:** There is no hard limit to how many instances you can add to your fleet.
- **Cons:**
  - **Architectural Complexity:** The application must be designed to be stateless and distributed to work in a horizontally-scaled environment.

---

### **SAA-C03 Style Scenario Questions**

**1. A company runs a monolithic, stateful application on a single, large EC2 instance. The application is struggling to keep up with demand, and performance is degrading. The application cannot be easily re-architected to run on multiple instances. What is the most straightforward way to improve its performance in the short term?**

- A. Place the instance behind an Application Load Balancer.
- B. Deploy the instance into an Auto Scaling Group with a desired capacity of 2.
- C. Stop the instance, change its instance type to one with more CPU and RAM, and then start it again.
- D. Create a read replica of the instance.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This describes vertical scaling. Since the application is monolithic and stateful, it cannot be easily distributed across multiple servers (horizontal scaling). The most direct approach is to "scale up" by moving the application to a more powerful EC2 instance type. This will require some downtime.

</details>

---

**2. A popular e-commerce website is preparing for a major sales event. They expect a massive increase in traffic and need their architecture to automatically add more web servers to handle the load and then remove them after the event is over. Which AWS services are essential for this strategy?**

- A. AWS Direct Connect and AWS VPN
- B. Elastic Load Balancer (ELB) and an Auto Scaling Group (ASG)
- C. Amazon EFS and AWS Storage Gateway
- D. Amazon RDS Multi-AZ

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic horizontal scaling (scaling out and in) scenario. An Auto Scaling Group is used to automatically launch and terminate instances based on demand. An Elastic Load Balancer is required to distribute the incoming traffic across all the instances in that group.

</details>

---

**3. What is a major limitation of vertical scaling?**

- A. It is more expensive than horizontal scaling.
- B. It increases network latency.
- C. There is an upper limit to the size of a single instance, so you can't scale indefinitely.
- D. It requires the application to be stateless.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Vertical scaling has a hard limit. Eventually, you will reach the largest, most powerful instance type offered by AWS. Once you hit that ceiling, you cannot scale up any further. Horizontal scaling, in contrast, has no theoretical limit.

</details>

---

**4. Which architectural principle is a key benefit of horizontal scaling but NOT of vertical scaling?**

- A. Simplicity of management.
- B. Improved performance for a single task.
- C. Built-in high availability and fault tolerance.
- D. Lower cost for small workloads.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Horizontal scaling inherently improves availability. By running multiple instances (ideally across different Availability Zones), the failure of a single instance does not bring down your entire application. Vertical scaling does nothing to improve availability; you still have a single point of failure, even if it's a more powerful one.

</details>

---

**5. The ability of a system to automatically add resources to handle increased load and then remove them when the load decreases is known as:**

- A. Fault Tolerance
- B. High Availability
- C. Elasticity
- D. Durability

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Elasticity is the term for automatically scaling resources both out (adding) and in (removing) to precisely match demand. This is a core benefit of the cloud and is achieved through horizontal scaling with services like Auto Scaling Groups.

</details>

---

### **Key Exam Tips & Tricks**

- **Horizontal is the Cloud Way:** For the SAA-C03 exam, remember that **horizontal scaling is almost always the preferred pattern** in the cloud. It embodies the principles of elasticity and high availability. Vertical scaling is typically presented as a solution for legacy, monolithic, or stateful applications that cannot be easily distributed.
- **Keyword Association:**
  - **Vertical Scaling (Up):** Keywords are "increase instance size," "change instance type," "monolithic," "stateful application," "cannot be distributed." This is the solution when you're stuck with a single server.
  - **Horizontal Scaling (Out):** Keywords are "add more instances," "handle increased traffic," "elasticity," "high availability," "stateless application." This is the solution for modern, cloud-native applications.
- **The Holy Trinity of HA:** When you see a requirement for a highly available and scalable web application, your mind should immediately go to the combination of **Route 53**, an **Elastic Load Balancer (ELB)**, and an **Auto Scaling Group (ASG)** of EC2 instances spread across multiple AZs. This is the canonical horizontal scaling pattern on AWS.
- **Downtime is a Clue:** If a solution involves stopping and starting an instance, it's vertical scaling. If the solution is designed to handle load or failures without downtime, it's horizontal scaling.
- **Database Scaling:** Be careful how these concepts apply to databases.
  - **Vertical Scaling:** Changing the DB instance class of an RDS instance (e.g., from `db.t3.medium` to `db.m5.large`). This requires downtime for non-Aurora databases.
  - **Horizontal Scaling (for reads):** Adding **Read Replicas** to an RDS or Aurora database to distribute the read workload across multiple instances.
