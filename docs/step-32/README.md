# Step 32: Subnets – Public, Private, Isolated

## 🎯 Objective

- [x] Master: **Subnets – Public, Private, Isolated**

## 📘 Notes

### **Deep Dive: Subnets – Public, Private, Isolated**

Within a VPC, a subnet is a subdivision of your IP address range where you can place groups of isolated resources. The *only* thing that determines whether a subnet is public, private, or isolated is its **routing configuration**. It has nothing to do with the name of the subnet or any checkbox you select.

---

### **1. Public Subnet** 🌐

A public subnet is one that has a direct route to the internet. It's the "front door" of your VPC.

- **Defining Characteristic:** The subnet's associated route table contains a route for `0.0.0.0/0` (all internet-bound traffic) that points to an **Internet Gateway (IGW)**.
- **Instance Access:** For an EC2 instance inside a public subnet to be directly reachable from the internet, it must have a **public IPv4 address** or an **Elastic IP address**.
- **Common Resources:**
    - Public-facing web servers.
    - Bastion hosts (jump servers) for secure administrative access.
    - Public-facing Elastic Load Balancers.
    - **NAT Gateways** (which must reside in a public subnet to function).

---

### **2. Private Subnet** 🔒

A private subnet does not have a direct route to the internet. However, resources within it might still need to *initiate* outbound connections to the internet (e.g., to download software patches or call external APIs).

- **Defining Characteristic:** The subnet's route table does **NOT** have a route to an Internet Gateway.
- **Outbound Internet Access:** To get outbound-only internet access, a private subnet must have a route for `0.0.0.0/0` that points to a **NAT (Network Address Translation) Gateway** or a NAT Instance. The NAT Gateway itself must be located in a public subnet.
- **How NAT Works:** The instance in the private subnet sends its outbound traffic to the NAT Gateway. The NAT Gateway, which has a public IP, forwards the traffic to the internet on behalf of the private instance. When a response comes back, the NAT Gateway knows which private instance to send it to. This allows outbound connections but prevents the internet from initiating connections *to* the private instance.
- **Common Resources:**
    - Application servers that process requests from web servers but shouldn't be publicly exposed.
    - Database servers.
    - Backend processing workloads.

---

### **3. Isolated Subnet (Private Subnet with No Internet Access)** 🚫

This is a specific type of private subnet that has no route to an IGW *and* no route to a NAT Gateway. It is completely isolated from the internet.

- **Defining Characteristic:** The subnet's route table only contains the default `local` route for the VPC. There is no `0.0.0.0/0` route at all.
- **Internet Access:** Resources in an isolated subnet have **no internet connectivity**, either inbound or outbound. They can only communicate with other resources within the same VPC.
- **Common Resources:**
    - The most sensitive resources, such as database servers that should *never* need to download patches or connect to external services.
    - Internal directory services or other core infrastructure that only needs to talk to other resources within the VPC.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. An EC2 instance in a private subnet needs to download a critical security patch from the internet. The instance must not be directly reachable from the internet. What component is required to allow this outbound-only connectivity?**

- A. An Internet Gateway attached to the private subnet's route table.
- B. A NAT Gateway located in a public subnet, with a route from the private subnet's route table to it.
- C. An Elastic IP address attached directly to the instance in the private subnet.
- D. A VPC Peering connection.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is the primary use case for a NAT Gateway. By placing the NAT Gateway in a public subnet and adding a route (0.0.0.0/0 -> NAT Gateway) to the private subnet's route table, instances in the private subnet can initiate outbound connections to the internet. The NAT Gateway handles the translation, and external services cannot initiate connections back to the private instances.
</details>
---

**2. A solutions architect is designing a standard three-tier web application. Where should the public-facing web servers be placed?**

- A. In a private subnet, behind a NAT Gateway.
- B. In an isolated subnet with no internet route.
- C. In a public subnet.
- D. Directly attached to the Internet Gateway.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Public-facing web servers need to be accessible from the internet to serve user traffic. Therefore, they must be placed in a public subnet, which has a route to the Internet Gateway, and they must be assigned public or Elastic IP addresses (or be placed behind a public-facing load balancer).
</details>
---

**3. What is the single defining characteristic that makes a subnet "public"?**

- A. The "Auto-assign public IPv4 address" setting is enabled for the subnet.
- B. The subnet's route table has a route to an Internet Gateway.
- C. The subnet is associated with a security group that allows inbound HTTP traffic.
- D. The subnet's name contains the word "public".
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A subnet's classification as public or private depends entirely on its routing. If there is a route in its associated route table that directs internet-bound traffic (0.0.0.0/0) to an Internet Gateway, it is a public subnet. All other factors are irrelevant to this definition.
</details>
---

**4. A database server containing sensitive customer data is hosted on an EC2 instance. According to security best practices, this server should have no internet connectivity whatsoever. In which type of subnet should it be deployed?**

- A. A public subnet with a restrictive security group.
- B. A private subnet with a route to a NAT Gateway.
- C. An isolated subnet with no route to an IGW or a NAT Gateway.
- D. A subnet with a VPC Endpoint for S3.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The requirement for zero internet connectivity means the subnet should be completely isolated. An isolated subnet has a route table that only contains the default local route for VPC traffic, with no default route (0.0.0.0/0) at all. This ensures the database can communicate with the application tier within the VPC but has no path to or from the internet.
</details>
---

**5. A NAT Gateway must be deployed into which type of subnet to function correctly?**

- A. A private subnet.
- B. An isolated subnet.
- C. A public subnet.
- D. Any subnet, as long as it has an Elastic IP.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A NAT Gateway's purpose is to provide internet access to private resources. To do this, the NAT Gateway itself must be able to reach the internet. Therefore, it must be placed in a public subnet that has a route to an Internet Gateway.
</details>
---

**6. An EC2 instance is launched into a public subnet, but you cannot connect to it from the internet via SSH. The instance's security group allows inbound traffic on port 22 from your IP. The NACL for the subnet allows all traffic. What is the most likely missing component?**

- A. The instance does not have a public IPv4 address or Elastic IP address assigned to it.
- B. The private subnet does not have a route to a NAT Gateway.
- C. The VPC does not have an Internet Gateway attached.
- D. The route table for the public subnet is missing the local route.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** An instance can be in a public subnet, but if it doesn't have a public IP address, it is not reachable from the internet. To be directly accessible, the instance needs a public IP (either auto-assigned at launch or a persistent Elastic IP).
</details>
---

**7. How do instances in a private subnet typically receive software updates from online repositories?**

- A. They connect directly to the internet via the IGW.
- B. An administrator manually downloads patches and transfers them to the instances.
- C. They route their outbound traffic through a NAT Gateway located in a public subnet.
- D. They cannot receive updates.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This is a primary function of a NAT Gateway. It allows instances in a private subnet to initiate outbound connections to internet-based repositories to download updates, while still preventing the internet from initiating connections back to those instances.
</details>
---

**8. Which statement accurately describes the difference between an Internet Gateway (IGW) and a NAT Gateway?**

- A. An IGW provides two-way communication with the internet, while a NAT Gateway provides outbound-only communication for private instances.
- B. A NAT Gateway is for public subnets, and an IGW is for private subnets.
- C. An IGW is a stateful device, while a NAT Gateway is stateless.
- D. You can have multiple IGWs in a VPC, but only one NAT Gateway.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** This is the core distinction. An Internet Gateway enables full, bidirectional communication for public resources. A NAT Gateway enables a one-way street: it allows private instances to start a conversation with the internet, but it does not allow the internet to start a conversation with the private instances.
</details>
---

**9. A bastion host (or jump server) is used to provide secure administrative access to instances in private subnets. In which subnet should the bastion host itself be placed?**

- A. In the same private subnet as the instances it manages.
- B. In an isolated subnet.
- C. In a public subnet.
- D. In a different VPC.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A bastion host acts as the single, hardened entry point from the internet into your VPC. Therefore, the bastion host must be reachable from the internet and should be placed in a public subnet. Administrators then SSH into the bastion host, and from there, "jump" to the instances in the private subnets using their private IP addresses.
</details>
---

**10. You have configured a public subnet with CIDR `10.0.1.0/24` and a private subnet with CIDR `10.0.2.0/24`. How do instances in these two subnets communicate with each other?**

- A. Via the Internet Gateway.
- B. Via the NAT Gateway.
- C. Via the VPC's implicit router using the local route in their route tables.
- D. They cannot communicate directly.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** All route tables contain a local route for the VPC's entire CIDR block (e.g., 10.0.0.0/16). The VPC's implicit router uses this route to enable direct communication between all subnets within the VPC using their private IP addresses. This traffic never leaves the VPC and does not go through an IGW or NAT Gateway.
</details>
---

**11. Which resource is responsible for performing Network Address Translation for instances in a private subnet?**

- A. The Internet Gateway.
- B. The EC2 instance's ENI.
- C. The Route Table.
- D. The NAT Gateway.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** The NAT Gateway's entire purpose is to perform Network Address Translation. It takes traffic from instances with private IPs, translates the source IP to its own public IP, and forwards the traffic to the internet.
</details>
---

**12. A company is creating a new VPC. For high availability, they need at least two public and two private subnets. What is the minimum number of Availability Zones they must use?**

- A. 1
- B. 2
- C. 3
- D. 4
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** To have a highly available architecture, you must deploy resources across multiple Availability Zones. To have two public and two private subnets available for a resilient deployment, you would typically create one public and one private subnet in AZ-A, and one public and one private subnet in AZ-B, requiring a minimum of two AZs.
</details>
---

**13. An EC2 instance in a private subnet needs to connect to an Amazon S3 bucket. What is the most secure way to enable this connection without allowing general internet access?**

- A. Move the instance to a public subnet.
- B. Create a NAT Gateway for the private subnet.
- C. Create a VPC Gateway Endpoint for S3 and add it to the private subnet's route table.
- D. Assign an Elastic IP to the instance.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A VPC Gateway Endpoint for S3 allows instances in a private subnet to communicate with the S3 service directly over the private AWS network. The traffic never traverses the internet. This is more secure and often more performant than routing the traffic through a NAT Gateway.
</details>
---

**14. What is the key difference between a NAT Gateway and a NAT Instance?**

- A. A NAT Gateway is managed by AWS and is highly available, while a NAT Instance is a standard EC2 instance you must manage yourself.
- B. A NAT Instance is more scalable than a NAT Gateway.
- C. A NAT Gateway is free, while a NAT Instance has an hourly cost.
- D. A NAT Instance supports port forwarding, while a NAT Gateway does not.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** A NAT Gateway is a fully managed AWS service. AWS handles the patching, scaling, and high availability for you. A NAT Instance is just an EC2 instance that you configure with special routing rules to perform NAT. With a NAT Instance, you are responsible for its availability, patching, and security, making the managed NAT Gateway the preferred choice for most use cases.
</details>
---

**15. If an EC2 instance is in a subnet with no route to an IGW or a NAT Gateway, it is in a(n) __________ subnet.**

- A. public
- B. protected
- C. global
- D. isolated
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** A subnet with no route to the outside world (0.0.0.0/0) is considered an isolated subnet. Resources within it can communicate with other resources in the VPC but have no connectivity to or from the internet.
</details>
---

**16. The "Auto-assign public IPv4 address" setting is enabled for a subnet. What does this do?**

- A. It makes the subnet a public subnet.
- B. It ensures that any EC2 instance launched into that subnet is automatically given a public IP address.
- C. It attaches an Internet Gateway to the subnet.
- D. It converts all private IPs to public IPs.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a convenience setting. When enabled, it tells EC2 to automatically assign a transient public IP from Amazon's pool to any instance launched into that subnet. It does not, by itself, make the subnet public; the route to an IGW is still required for that.
</details>
---

**17. You have a public subnet and a private subnet. An Elastic Load Balancer should be placed in the _____ subnet, while the database servers it connects to should be in the _____ subnet.**

- A. private, public
- B. public, private
- C. private, private
- D. public, public
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The load balancer is the public entry point for your application, so it must be placed in the public subnet(s) to be reachable from the internet. The database servers contain sensitive data and should be protected in the private subnet(s), accessible only from the application tier, not directly from the internet.
</details>
---

**18. A NAT Gateway is a zonal resource, meaning it resides in a single Availability Zone. For a highly available architecture, what is the recommended practice?**

- A. Create a single, large NAT Gateway in one AZ.
- B. Use a NAT Instance instead, as it is regionally redundant.
- C. Create a NAT Gateway in each Availability Zone and configure route tables accordingly for each private subnet.
- D. Use an Internet Gateway as a backup for the NAT Gateway.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** To prevent the NAT Gateway from becoming a single point of failure, the best practice is to provision a NAT Gateway in each AZ that contains private subnets needing internet access. You then configure the route tables for the private subnets in each AZ to point to the NAT Gateway in their same AZ.
</details>
---

**19. An instance in a public subnet has a private IP of `10.0.1.50` and an Elastic IP of `54.10.20.30`. When it makes an outbound request to the internet, what source IP address will the destination server see?**

- A. `10.0.1.50`
- B. The private IP of the Internet Gateway.
- C. `54.10.20.30`
- D. A random IP from Amazon's pool.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The Internet Gateway performs a 1-to-1 network address translation for instances with a public or Elastic IP. It translates the instance's private IP (10.0.1.50) to its public-facing Elastic IP (54.10.20.30) for any traffic going to the internet.
</details>
---

**20. What is required for an instance in a private subnet to communicate with an instance in another private subnet within the same VPC?**

- A. A NAT Gateway.
- B. Nothing extra; this is enabled by the default local route in the VPC's route tables.
- C. An Internet Gateway.
- D. A VPC peering connection.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** All communication within a VPC is handled by the VPC's implicit router and the local route present in all route tables. Instances can communicate using their private IP addresses regardless of which subnet they are in (public or private).
</details>
---

**21. Can you associate a security group with a subnet?**

- A. Yes, to filter all traffic entering the subnet.
- B. No, security groups are applied at the instance (ENI) level, while Network ACLs are applied at the subnet level.
- C. Yes, but only for private subnets.
- D. No, security groups can only be associated with VPCs.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a fundamental distinction. Security Groups act as instance-level firewalls. The firewall that operates at the subnet level is the Network ACL (NACL).
</details>
---

**22. Which component is stateful?**

- A. A NAT Gateway
- B. A Network ACL
- C. A Route Table
- D. None of the above
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** A NAT Gateway is a stateful device. It maintains a connection tracking table to remember which private instance initiated an outbound request. When the response traffic comes back from the internet, the NAT Gateway uses this table to know which private instance to forward the response to. Network ACLs are stateless.
</details>
