# Step 17: Placement Groups, ENI, EIP

## 🎯 Objective

- [x] Master: **Placement Groups, ENI, EIP**

## 📘 Notes

### **Deep Dive: Placement Groups, ENI, & EIP**

Beyond just launching an instance, AWS provides tools to control where your instances are placed relative to each other and how they are addressed on the network. This is crucial for applications that require high performance, high availability, or static network identities.

---

### **1. Placement Groups** 📍

A Placement Group is a logical grouping of interdependent instances. It influences the physical placement of your EC2 instances on the underlying hardware to meet the demands of specific workloads. You must choose the type of placement group when you create it, and it cannot be changed.

- **Cluster Placement Group:**
  - **Purpose:** Packs instances close together inside a single **Availability Zone**. The goal is to achieve the lowest network latency and highest network throughput possible between instances.
  - **Use Case:** High-Performance Computing (HPC), tightly-coupled scientific modeling, and any application where instance-to-instance communication speed is critical.
  - **Risk:** Because all instances are on a small set of "racks," a single hardware failure can impact all instances in the group simultaneously. It prioritizes performance over high availability.
- **Spread Placement Group:**
  - **Purpose:** Spreads a small group of critical instances across distinct underlying hardware **racks** to reduce correlated failures. Each instance is on a different rack.
  - **Use Case:** Maximizing high availability for a small number of critical instances, such as a multi-node database cluster (e.g., Cassandra, ZooKeeper) or a set of primary/secondary servers. It ensures that a single rack failure will only ever take down one instance in the group.
  - **Limit:** There is a hard limit of **7 running instances per Availability Zone** for a spread placement group.
- **Partition Placement Group:**
  - **Purpose:** Spreads instances across many different logical **partitions** within an Availability Zone (or across AZs in a region). Each partition is a set of racks that does not share hardware with any other partition. This gives you visibility into which instances share a rack.
  - **Use Case:** Large distributed and replicated workloads where you need to isolate the impact of hardware failures while still deploying many instances, such as Hadoop, Cassandra, and Kafka. It's a balance between the performance of a cluster group and the availability of a spread group, but on a larger scale.

---

### **2. Elastic Network Interface (ENI)** 🔌

An ENI is a **virtual network card** in your VPC. It's a logical networking component that represents a network interface that can be attached to an EC2 instance.

- **Attributes:** An ENI has a primary private IPv4 address, one or more secondary private IPv4 addresses, one Elastic IP address (EIP) per private IP, a MAC address, and one or more security groups.
- **Lifecycle:** An ENI is **independent of the EC2 instance lifecycle**. You can create an ENI, attach it to an instance, detach it, and then re-attach it to a different instance in the same Availability Zone.
- **Use Cases:**
  - **High Availability / Failover:** You can have a "standby" ENI pre-configured with a specific private IP and Elastic IP. If your primary instance fails, you can quickly detach the ENI and attach it to a hot standby instance, which then instantly assumes the network identity of the failed instance.
  - **Dual-Homing:** Attaching two ENIs from different subnets to a single instance allows it to be present on two different networks simultaneously (e.g., a management network and an application network).
  - **Security:** You can attach different security groups to different ENIs on the same instance to create granular network segmentation.

---

### **3. Elastic IP Address (EIP)** STATIC

An EIP is a **static, public IPv4 address** designed for dynamic cloud computing. Unlike a standard public IP that changes when you stop and start an instance, an EIP is an address that you allocate to your account and that is yours until you release it.

- **Purpose:** To provide a fixed, public endpoint for your application. If your instance fails, you can remap the EIP to a replacement instance, allowing your users to continue accessing the service at the same IP address.
- **Association:** An EIP is associated with either an **EC2 instance** or, more specifically, with a **network interface (ENI)**.
- **Billing (Important Nuance):**
  - EIPs are **free** as long as they are **attached to a running EC2 instance**.
  - You are **billed a small hourly fee** for an EIP when it is **NOT attached** to a running instance, or if it's attached to a stopped instance or an ENI that isn't attached to an instance. This is to discourage a wasteful practice called "IP hoarding." You are also only allowed 5 EIPs per region by default.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company is deploying a High-Performance Computing (HPC) cluster on EC2. The application requires the lowest possible network latency and highest throughput between nodes. Which EC2 Placement Group strategy should be used?**

- A. Spread Placement Group
- B. Partition Placement Group
- C. Cluster Placement Group
- D. A combination of all three types.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Cluster Placement Group is specifically designed for this use case. It co-locates instances on the same underlying hardware rack within a single Availability Zone to minimize the network path between them, providing the best possible performance for tightly-coupled workloads.

</details>


---

**2. A solutions architect is designing a high-availability solution for a critical web server. If the primary instance fails, a standby instance must take over its network identity, including its private and public IP addresses, without any DNS changes. What is the most effective way to achieve this?**

- A. Use an Application Load Balancer to distribute traffic between the two instances.
- B. Create an Elastic Network Interface (ENI) with the required IP addresses, attach it to the primary instance, and move the ENI to the standby instance upon failure.
- C. Write a script that assigns a new Elastic IP to the standby instance upon failure.
- D. Place both instances in a Spread Placement Group.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic use case for an Elastic Network Interface (ENI). Because an ENI is an independent network object, you can pre-configure it with the necessary private IP, Elastic IP, and security groups. In a failover scenario, you simply detach the ENI from the failed instance and attach it to the standby instance, which instantly assumes the exact network identity.

</details>


---

**3. An administrator has allocated an Elastic IP (EIP) address but has not associated it with any running EC2 instance. What are the cost implications of this action?**

- A. There is no cost, as EIPs are always free.
- B. The user will be charged a small hourly fee for the unattached EIP.
- C. The user will be charged the same as running a t2.micro instance.
- D. The cost is a one-time allocation fee.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AWS discourages "IP hoarding." An EIP is only free when it is associated with a running EC2 instance. If it is allocated to your account but is unattached, or attached to a stopped instance, you will incur a small hourly charge until it is either attached to a running instance or released.

</details>


---

**4. A company is deploying a distributed NoSQL database like Cassandra on AWS. They need to deploy a large number of instances and want to minimize the impact of a single underlying hardware failure. Which Placement Group type is best suited for this large-scale, resilient workload?**

- A. Cluster Placement Group
- B. Spread Placement Group
- C. Partition Placement Group
- D. Dedicated Host

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Partition Placement Group is designed for large, distributed workloads. It spreads instances across logical partitions (groups of racks) and gives you visibility into this placement. This allows you to deploy many instances while ensuring that a hardware failure in one partition does not affect instances in other partitions, which is ideal for partition-aware applications like Cassandra, HDFS, and Kafka.

</details>


---

**5. What is the maximum number of instances you can have in a Spread Placement Group within a single Availability Zone?**

- A. 2
- B. 7
- C. 10
- D. There is no limit.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a hard limit you should memorize for the exam. A Spread Placement Group guarantees that each instance is on a distinct hardware rack, and there is a limit of 7 running instances per AZ that can have this guarantee.

</details>


---

**6. An EC2 instance with a single ENI is stopped and then started again. It was configured to have a public IP address. What will happen to its public and private IP addresses?**

- A. Both the public and private IP addresses will change.
- B. The public IP will change, but the private IP will remain the same.
- C. The public IP will remain the same, but the private IP will change.
- D. Both the public and private IP addresses will remain the same.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A standard, auto-assigned public IP is ephemeral and is released when an instance is stopped. Upon starting again, it will receive a new public IP from the pool. However, the private IP address assigned to the ENI is retained for the life of the ENI, so it will remain the same after the stop/start cycle. The only way to get a persistent public IP is to use an Elastic IP.

</details>


---

**7. A company wants to run a workload with a 3-node primary/standby database cluster and wants to ensure the highest possible availability by placing each node on separate underlying hardware. Which Placement Group type should they use?**

- A. Cluster
- B. Spread
- C. Partition
- D. None, this should be done by launching in different AZs.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Spread Placement Group is the perfect solution for a small number of critical instances that must be isolated from each other to prevent simultaneous failure. It ensures that each of the 3 database nodes will be on a different physical rack. While launching in different AZs (D) also provides high availability, a spread group provides an additional layer of protection against rack-level failures within an AZ.

</details>


---

**8. What is the primary purpose of an Elastic IP address?**

- A. To provide a private IP address for an instance.
- B. To provide a static, persistent public IP address that can be remapped between instances.
- C. To allow an instance to have multiple network interfaces.
- D. To encrypt network traffic to and from an instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An Elastic IP (EIP) is a static public IP address that you can allocate to your AWS account and attach to instances. Unlike the default auto-assigned public IPs that change when an instance is stopped/started, an EIP remains constant and can be reassigned between instances as needed, providing continuity for applications that require a fixed public IP.

</details>
    
    Explanation: The key features of an EIP are that it is public and static. This provides a fixed entry point for your services that survives instance failures, reboots, or replacements, as you can simply re-associate the EIP with a new, healthy instance.


---

**9. Can an Elastic Network Interface created in Availability Zone `us-east-1a` be attached to an instance running in `us-east-1b`?**

- A. Yes, ENIs are regional resources.
- B. No, an ENI is locked to the Availability Zone in which it was created.
- C. Yes, but only if the instances are in the same placement group.
- D. Yes, but you will incur data transfer charges.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a key constraint. An ENI is a zonal resource. It can only be attached to instances that are running in the same Availability Zone. To achieve a similar outcome across AZs, you would typically use a service like an Elastic Load Balancer.

</details>


---

**10. You are trying to launch a new `m5.xlarge` instance into an existing Cluster Placement Group that already contains several other instances. The launch fails with a capacity error. What is the most likely reason?**

- A. The placement group has reached its limit of 7 instances.
- B. There isn't enough contiguous capacity on a single rack within the Availability Zone to host the new instance close to the existing ones.
- C. You cannot mix instance types within a Cluster Placement Group.
- D. The account has reached its EC2 instance limit.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The entire purpose of a Cluster Placement Group is to pack instances close together. If the underlying hardware in that specific location is already heavily utilized, AWS may not be able to find a suitable physical server close enough to the existing members of the group to satisfy the low-latency requirement. The best practice is to stop and restart all instances in the group, which may migrate them to a new rack with more capacity, and then try the launch again.

</details>


---

**11. An EC2 instance needs to be part of two different subnets simultaneously to route traffic between a public-facing network and a private management network. How can this be achieved?**

- A. By creating a VPC peering connection between the two subnets.
- B. By creating two Elastic IPs and attaching them to the instance.
- C. By creating two Elastic Network Interfaces, each in a different target subnet, and attaching both to the instance.
- D. This is not possible; an instance can only belong to one subnet.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This scenario, often called "dual-homing," is a primary use case for multiple ENIs. You can attach a primary ENI from the public subnet and a secondary ENI from the private subnet to the same instance, allowing it to have a network presence in both and act as a bridge or router between them.

</details>


---

**12. When would you be charged for an Elastic IP address? (Select TWO)**

- A. When it is attached to a running EC2 instance.
- B. When it is allocated to your account but not attached to any resource.
- C. When it is attached to a stopped EC2 instance.
- D. When it is attached to an ENI that is in use by a running instance.
- E. When you release it back to the AWS pool.

<details>
<summary>View Answer</summary>

**Answers: B and C**

**Explanation:** You are billed for an EIP when it is not being "used efficiently." This means you pay a small hourly charge when it is allocated but unattached (B) and when it is attached to a stopped instance (C), as it is holding a public IP from the pool without active compute behind it. It is free when attached to a running instance (A/D).

</details>


---

**13. A Partition Placement Group is created with 5 partitions. You then launch 10 instances into this group. How will AWS distribute the instances?**

- A. All 10 instances will be placed in the first partition.
- B. Two instances will be placed in each of the 5 partitions.
- C. The launch will fail because you cannot have more instances than partitions.
- D. The instances will be randomly distributed across the partitions, with no specific balance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When you launch instances into a Partition Placement Group, AWS attempts to balance the instances evenly across the number of partitions you specified. With 10 instances and 5 partitions, it would place 2 instances in each partition.

</details>


---

**14. What happens to the network interfaces and IP addresses when you terminate an EC2 instance?**

- A. The primary ENI is deleted, and its Elastic IP is released.
- B. The primary ENI is detached and becomes available, and any EIP associated with it is disassociated.
- C. The primary ENI and its EIP are retained for 30 days.
- D. The entire VPC is deleted along with the instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The default behavior is that the primary network interface is detached when the instance is terminated. Any Elastic IP that was associated with it is disassociated but remains allocated to your account (and you will start being charged for it). Note that you can change a setting to have the ENI be deleted on termination, but the default is to detach it.

</details>


---

**15. Which Placement Group strategy is designed to protect against a single point of failure at the hardware rack level for a small number of critical instances?**

- A. Cluster
- B. Spread
- C. Partition
- D. Any placement group provides this protection.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Spread Placement Group is the only type whose explicit purpose is to ensure that every instance in the group resides on a different physical rack. This directly mitigates the risk of a single rack failure (e.g., power supply or top-of-rack switch failure) taking down your entire application.

</details>


---

**16. An ENI is created with a primary private IP of `10.0.1.10` and a secondary private IP of `10.0.1.11`. Can you attach two different Elastic IPs to this single ENI?**

- A. No, an ENI can only have one Elastic IP address.
- B. Yes, you can associate one Elastic IP with the primary private IP and another Elastic IP with the secondary private IP.
- C. No, Elastic IPs can only be attached to instances, not ENIs.
- D. Yes, but both Elastic IPs must map to the primary private IP.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** You can associate one Elastic IP address with each private IP address on a network interface. This allows a single ENI (and thus a single instance) to be reachable via multiple static public IP addresses.

</details>


---

**17. What is the key risk associated with using a Cluster Placement Group?**

- A. High network latency between instances.
- B. Increased cost due to physical hardware reservation.
- C. An increased risk of simultaneous instance failure due to a localized hardware issue.
- D. Inability to use Elastic IP addresses.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The trade-off for the high performance of a Cluster Placement Group is reduced availability. Because all instances are packed closely together on the same underlying hardware, a failure of that single piece of hardware (like a network card or power supply on the host) can cause all instances in the group to fail at the same time.

</details>


---

**18. To what AWS resource is an Elastic IP address most directly associated?**

- A. An IAM Role
- B. An Availability Zone
- C. A Network Interface
- D. A Subnet

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While we often say we are attaching an EIP to an "instance," the association is technically made with a Network Interface (ENI) on that instance. This is why when the ENI moves to another instance, the EIP moves with it.

</details>


---

**19. You try to launch a new instance into a Spread Placement Group that already has 7 instances running in `us-east-1a`. What will happen?**

- A. The launch will succeed, and the 8th instance will be placed on a new rack.
- B. The launch will fail due to capacity limits for that placement group type.
- C. The launch will succeed, but the instance will be placed on a rack that already contains another instance from the group.
- D. The placement group will automatically be converted to a Partition group.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Spread Placement Group has a hard limit of 7 running instances per Availability Zone. Attempting to launch an 8th instance into the same AZ within that group will result in a launch failure.

</details>


---

**20. You have an instance with a primary ENI (`eth0`). You create and attach a second ENI (`eth1`). Which of the following is TRUE?**

- A. `eth1` can be in a different Availability Zone than `eth0`.
- B. The instance can now send and receive traffic from two different subnets.
- C. Both ENIs must share the same security group.
- D. Terminating the instance will automatically release any EIPs attached to `eth1`.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the "dual-homing" use case. By attaching a second ENI that is configured for a different subnet, the instance gains a network presence in both subnets. Both ENIs must be in the same AZ (A is false). They can have different security groups (C is false). EIPs are disassociated, not released, upon termination (D is false).

</details>


---

**21. A legacy application requires a license that is tied to the MAC address of the server's network card. How can you ensure this application can run on EC2 even if the instance fails and needs to be replaced?**

- A. Launch the instance in a Cluster Placement Group.
- B. Use an Elastic Network Interface (ENI).
- C. Use an Elastic IP (EIP).
- D. This is not possible on AWS.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An Elastic Network Interface (ENI) has a persistent MAC address. You can attach this ENI to your primary instance and license your software against its MAC address. If the instance fails, you can detach the ENI and attach it to a new standby instance. The new instance will now have the same ENI with the same MAC address, satisfying the software's license check.

</details>


---

**22. Which placement group strategy gives you visibility into the logical segments of hardware your instances are running on, making it suitable for partition-aware applications like Kafka?**

- A. Cluster
- B. Spread
- C. Partition
- D. Dedicated

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The defining feature of a Partition Placement Group is that it divides instances into logical, non-overlapping partitions and gives you visibility into which instances are in which partition. This allows applications that manage their own replication (like Kafka or HDFS) to intelligently place replicas in different partitions to ensure a hardware failure only affects one copy of the data.

</details>
