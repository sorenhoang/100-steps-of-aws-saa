# Step 33: Internet Gateway & NAT Gateway

## 🎯 Objective

- [x] Master: **Internet Gateway & NAT Gateway**

## 📘 Notes

### **Deep Dive: Internet Gateway & NAT Gateway**

To connect a VPC to the internet, you need a gateway. AWS provides two main types: an Internet Gateway (IGW) for two-way traffic and a NAT Gateway for one-way, outbound-only traffic. They serve different subnets and are fundamental to securing your network perimeter.

---

### **1. Internet Gateway (IGW)** ↔️

An Internet Gateway is a horizontally scaled, redundant, and highly available VPC component that allows **two-way communication** between your VPC and the internet.

- **Function:** An IGW serves two main purposes:
    1. It provides a target in your VPC route tables for internet-routable traffic.
    2. It performs network address translation for instances that have been assigned public or Elastic IP addresses.
- **How it Works:**
    1. You create and attach one (and only one) IGW to your VPC.
    2. You create a **custom route table** for your public subnet(s).
    3. In this route table, you add a route: `0.0.0.0/0` (all internet traffic) with the target set to your IGW.
    4. Any instance in a subnet associated with this route table is now in a "public subnet." If that instance has a public IP, it can now communicate with the internet.
- **Key Characteristics:**
    - **Highly Available:** It is fully managed and redundant by design; it cannot fail.
    - **No Bottlenecks:** It scales horizontally and does not impose any bandwidth constraints on your network traffic.
    - **Free:** There is no charge for having an IGW attached to your VPC.

---

### **2. NAT Gateway (Network Address Translation)** ➡️

A NAT Gateway is a managed AWS service that enables instances in a **private subnet** to connect to the internet or other AWS services, but prevents the internet from initiating a connection with those instances.

- **Function:** To provide **outbound-only internet access** for private resources.
- **How it Works:**
    1. You must launch the NAT Gateway into a **public subnet**.
    2. The NAT Gateway must be allocated an **Elastic IP address** (a static public IP).
    3. You create a route in the route table for your **private subnet(s)**: `0.0.0.0/0` with the target set to your NAT Gateway.
    4. When an instance in the private subnet tries to access the internet (e.g., `yum update`), its traffic is routed to the NAT Gateway. The NAT Gateway then translates the instance's private source IP to its own public Elastic IP and sends the traffic to the internet.
- **Key Characteristics:**
    - **Managed Service:** AWS handles the administration, high availability, and scaling of the NAT Gateway for you. This is the main advantage over the older "NAT Instance" model.
    - **Stateful:** It automatically tracks outbound connections, so return traffic is allowed back to the originating private instance.
    - **Availability:** A NAT Gateway resides in a single Availability Zone. For a highly available architecture, you must deploy a NAT Gateway in **each AZ** and configure the route tables for private subnets to use the NAT Gateway in their respective AZ.
    - **Cost:** You are billed for each hour the NAT Gateway is provisioned and for each gigabyte of data it processes.

---

### **NAT Gateway vs. NAT Instance**

For the exam, it's important to know the difference:

| Feature | NAT Gateway | NAT Instance |
| --- | --- | --- |
| **Management** | **Fully managed by AWS** | A standard EC2 instance you manage |
| **Availability** | Highly available within its AZ | You are responsible for HA (scripts, etc.) |
| **Bandwidth** | Scales up to 45 Gbps | Depends on the EC2 instance type |
| **Maintenance** | None (handled by AWS) | You must handle patching, updates, etc. |
| **Security** | No security group to manage | Must manage its security group |
| **Recommendation** | **Preferred by AWS** | Legacy, not recommended for new designs |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A fleet of EC2 instances in a private subnet must download software updates from the internet. However, for security reasons, these instances cannot be directly accessible from the internet. Which combination of services is required?**

- A. An Internet Gateway in the private subnet and Elastic IPs on the instances.
- B. A NAT Gateway in a public subnet and a route in the private subnet's route table pointing to it.
- C. A VPC Peering connection to a public VPC.
- D. A Direct Connect link to the on-premises data center.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is the classic use case for a NAT Gateway. Placing it in a public subnet allows it to reach the internet. By routing the private subnet's internet-bound traffic (0.0.0.0/0) to the NAT Gateway, the private instances can initiate outbound connections without having any inbound connectivity from the internet.
</details>
---

**2. What is the primary purpose of an Internet Gateway (IGW)?**

- A. To provide outbound-only internet access for private subnets.
- B. To provide a target in a VPC's route tables to enable two-way communication with the internet.
- C. To filter incoming traffic based on source IP address.
- D. To connect multiple VPCs together.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** An Internet Gateway is the fundamental component for enabling internet access for a VPC. It serves as the gateway for all internet-routable traffic, allowing resources with public IPs to communicate bidirectionally with the internet.
</details>
---

**3. In which type of subnet must a NAT Gateway be deployed to function correctly?**

- A. A private subnet.
- B. An isolated subnet.
- C. A public subnet.
- D. A VPN-only subnet.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A NAT Gateway's job is to forward traffic to the internet. To do this, it must have a path to the internet itself. Therefore, a NAT Gateway must be placed in a public subnet, which is defined as a subnet with a route to an Internet Gateway.
</details>
---

**4. A company has deployed a NAT Gateway in `us-east-1a`. All private subnets in `us-east-1a` and `us-east-1b` are routed through this single NAT Gateway. What is a key risk in this design?**

- A. The NAT Gateway will not be able to handle the traffic from two Availability Zones.
- B. If the `us-east-1a` Availability Zone fails, all instances in the `us-east-1b` private subnets will lose their outbound internet connectivity.
- C. The design will incur high data transfer costs between the NAT Gateway and the Internet Gateway.
- D. This design is not possible, as a NAT Gateway can only serve one AZ.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A NAT Gateway is a zonal resource. If you only deploy one in a single AZ, it becomes a single point of failure for your architecture's internet connectivity. If that AZ experiences an outage, any resources in other AZs that depend on it will lose their internet path. The best practice is to deploy a NAT Gateway in each AZ.
</details>
---

**5. Which statement accurately compares a NAT Gateway to a NAT Instance?**

- A. A NAT Gateway is less expensive but requires more management overhead.
- B. A NAT Instance provides higher availability and bandwidth by default.
- C. A NAT Gateway is a fully managed AWS service, whereas a NAT Instance is an EC2 instance that you must manage yourself.
- D. A NAT Gateway requires a security group, but a NAT Instance does not.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This is the core difference. AWS recommends using a NAT Gateway because it is a managed service where AWS handles the high availability, scaling, and maintenance. A NAT Instance requires you to handle all of those operational burdens yourself (patching, security, failure recovery).
</details>
---

**6. For an EC2 instance in a public subnet to be able to communicate with the internet, which two components are required? (Select TWO)**

- A. A public IP address or Elastic IP address on the instance.
- B. A NAT Gateway in the same subnet.
- C. A route in the subnet's route table pointing to an Internet Gateway.
- D. A VPC Endpoint for the `com.amazonaws.global.internet` service.
- E. An AWS Direct Connect connection.
<details>
<summary>View Answer</summary>

**Answers:** A and C

**Explanation:** Two things are necessary for public communication: 1) The instance must have a publicly routable IP address, which can be a public IP or an Elastic IP (A). 2) The instance's subnet must have a path to the internet, which is provided by a route table entry (0.0.0.0/0) that points to an Internet Gateway (C).
</details>
---

**7. Can you associate a security group with an Internet Gateway?**

- A. Yes, to control all traffic entering the VPC.
- B. No, an Internet Gateway is not a resource that can have a security group attached.
- C. Yes, but only to control outbound traffic.
- D. Only if you are using a NAT Instance instead of a NAT Gateway.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** An Internet Gateway is a logical construct and managed service at the VPC level. You cannot attach a security group to it. Traffic filtering is handled by Network ACLs at the subnet level and Security Groups at the instance (ENI) level.
</details>
---

**8. How do you achieve a highly available architecture for outbound internet access from private subnets spread across multiple Availability Zones?**

- A. Deploy a single, large NAT Gateway and have all private subnets route to it.
- B. Deploy a NAT Gateway in each Availability Zone and configure each AZ's private subnets to route to the NAT Gateway in their respective AZ.
- C. Use an Application Load Balancer in front of multiple NAT Instances.
- D. Use an Internet Gateway, as it is highly available by default.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Since a NAT Gateway is a zonal resource, the best practice for high availability is to deploy one in each AZ. You then configure the route tables for the private subnets in an AZ to use the local NAT Gateway. This ensures that an outage in one AZ will not affect the internet connectivity of resources in other AZs.
</details>
---

**9. What kind of IP address must be associated with a NAT Gateway?**

- A. A private IP address from the subnet it resides in.
- B. An auto-assigned public IP address.
- C. An Elastic IP address.
- D. No IP address is required.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** When you create a NAT Gateway, you must allocate an Elastic IP address and associate it with the gateway. This provides a static, public IP address that the NAT Gateway uses as its source IP when sending traffic to the internet.
</details>
---

**10. An EC2 instance in a private subnet makes a request to a public web server on the internet. The traffic is routed through a NAT Gateway. What is the source IP address of the request as seen by the web server?**

- A. The private IP address of the EC2 instance.
- B. The private IP address of the NAT Gateway.
- C. The Elastic IP address associated with the NAT Gateway.
- D. The public IP address of the Internet Gateway.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The NAT Gateway performs Network Address Translation. It takes the packet from the private instance and replaces the private source IP with its own public Elastic IP address before sending it to the internet.
</details>
---

**11. An EC2 instance in a public subnet with a public IP address is able to reach the internet. What happens if you detach the Internet Gateway from the VPC?**

- A. The instance immediately loses its ability to communicate with the internet.
- B. The instance can still send outbound traffic but cannot receive inbound traffic.
- C. Nothing happens, as the instance has a public IP address.
- D. The instance's route table is automatically updated to point to a NAT Gateway.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** The Internet Gateway is the required exit and entry point for all internet traffic. Even if an instance has a public IP and a correct route table, detaching the IGW from the VPC removes the target of that route, breaking all internet connectivity.
</details>
---

**12. A company needs to provide instances in a private subnet with access to Amazon S3. They want to ensure the traffic does not traverse the public internet. Which is the more secure and cost-effective option than using a NAT Gateway?**

- A. An Internet Gateway
- B. A VPC Gateway Endpoint for S3
- C. An AWS Direct Connect link
- D. An Elastic IP on each private instance
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** While a NAT Gateway would work, it sends traffic over the internet. A VPC Gateway Endpoint for S3 is the superior solution. It creates a private connection between your VPC and the S3 service, so traffic stays on the secure AWS backbone network. It is also more cost-effective as there are no data processing charges, unlike with a NAT Gateway.
</details>
---

**13. Which statement about an Internet Gateway is true?**

- A. It performs stateful packet filtering.
- B. It is a highly available, managed component that cannot become a bottleneck.
- C. You are billed for the amount of data that passes through it.
- D. It must be deployed into a specific Availability Zone.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** An Internet Gateway is a logical construct that is fully managed, horizontally scaled, and highly available by design. It is not a single device that can fail or become a bandwidth bottleneck.
</details>
---

**14. A NAT Instance differs from a NAT Gateway in that you must:**

- A. Pay for the data it processes.
- B. Disable the source/destination check on the EC2 instance.
- C. Assign it an Elastic IP address.
- D. Place it in a public subnet.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A standard EC2 instance will only process traffic that is specifically destined for it. To make an EC2 instance act as a router or NAT device (forwarding traffic on behalf of others), you must disable the source/destination check in its network settings. This is a key configuration step for a NAT Instance that is not required for a managed NAT Gateway.
</details>
---

**15. What is the path of a packet originating from an EC2 instance in a private subnet destined for the internet?**

- A. EC2 -> Internet Gateway -> Internet
- B. EC2 -> NAT Gateway -> Internet Gateway -> Internet
- C. EC2 -> Internet
- D. EC2 -> VPC Router -> Internet
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The packet leaves the EC2 instance, and the subnet's route table directs it to the NAT Gateway. The NAT Gateway, being in a public subnet, has its traffic directed by its route table to the Internet Gateway, which then sends the packet to the internet.
</details>
---

**16. How many NAT Gateways are you billed for if you deploy one in `us-east-1a` and another in `us-east-1b` for high availability?**

- A. You are only billed for one, as they are part of a high-availability pair.
- B. You are billed for two NAT Gateways (one for each).
- C. You are not billed for the gateways, only the data they process.
- D. You are only billed for the one that is actively processing traffic.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** NAT Gateway pricing includes a per-hour charge for each gateway that is provisioned. If you deploy two gateways for high availability, you will pay the hourly charge for two gateways, plus the data processing charges for the traffic that flows through them.
</details>
---

**17. What is the default route destination for all internet-bound traffic in a route table?**

- A. `10.0.0.0/8`
- B. `0.0.0.0/0`
- C. `192.168.0.0/16`
- D. `local`
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The CIDR notation 0.0.0.0/0 is a special route that represents all IP addresses that are not explicitly defined in other, more specific routes in the table. It effectively means "any destination on the internet."
</details>
---

**18. Which service is stateful?**

- A. Network ACL
- B. Route Table
- C. Internet Gateway
- D. NAT Gateway
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** A NAT Gateway is a stateful device. It maintains a mapping of outbound requests to internal private IP addresses so that it can correctly route the return traffic back to the instance that initiated the connection. Network ACLs are stateless.
</details>
---

**19. You have an application running on EC2 instances in a private subnet. These instances need to make API calls to a third-party service on the internet. How do you configure the networking?**

- A. Create an Internet Gateway and attach it to the private subnet.
- B. Create a NAT Gateway in a public subnet and route the private subnet's `0.0.0.0/0` traffic to it.
- C. Create a VPC Endpoint for the third-party service.
- D. Assign Elastic IPs to the EC2 instances.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a standard outbound access pattern. The instances are private, but they need to initiate connections to the internet. A NAT Gateway is the managed AWS service designed to provide this outbound-only connectivity.
</details>
---

**20. A user has created a custom VPC and a public subnet with a correctly configured route to an Internet Gateway. They launch an instance into this subnet but do not enable the "Auto-assign public IPv4 address" feature. Can this instance reach the internet?**

- A. Yes, because it is in a public subnet.
- B. Yes, but only after an Elastic IP is attached to it.
- C. No, because it does not have a public IP address.
- D. No, because the Internet Gateway does not have a route back to the instance.
<details>
<summary>View Answer</summary>

**Answer:** B or C

**Explanation:** Both B and C express the same underlying truth. The instance cannot reach the internet because it lacks a public IP address for the IGW to perform NAT with. To fix this, you would need to associate an Elastic IP address with the instance (B). So, the current state is that it cannot reach the internet (C). Both answers are correct facets of the same problem. For an exam, the most direct reason for the failure is the lack of a public IP (C).
</details>
---

**21. What is the maximum number of Internet Gateways you can attach to a VPC?**

- A. 1
- B. 2
- C. 5
- D. 1 per AZ
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** A VPC can have only one Internet Gateway. This single gateway handles all internet traffic for the entire VPC.
</details>
---

**22. If you delete a NAT Gateway, what happens to its associated Elastic IP address?**

- A. The Elastic IP is also deleted.
- B. The Elastic IP is disassociated from the NAT Gateway and remains in your account.
- C. The Elastic IP is returned to the Amazon pool of addresses.
- D. The Elastic IP is attached to the Internet Gateway as a backup.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** When you delete a NAT Gateway, the Elastic IP that was associated with it is simply disassociated. It remains allocated to your account, and you will begin being charged for it as an unattached EIP until you either use it for another resource or release it.
</details>