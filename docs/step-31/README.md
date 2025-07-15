# Step 31: VPC Fundamentals & CIDR Planning

## 🎯 Objective

- [x] Master: **VPC Fundamentals & CIDR Planning**

## 📘 Notes

### **Deep Dive: VPC Fundamentals & CIDR Planning**

Amazon Virtual Private Cloud (VPC) is your own logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. It gives you complete control over your virtual networking environment, including selection of yo1ur own IP address range, creation of subnets, and configuration of route tables and network gateways.

---

### **1. Core Concepts**

- **VPC (Virtual Private Cloud):** Think of this as your own private data center in the cloud. It's a container for all your networking components and resources for a specific region. A VPC is a regional service.
- **Subnet:** A subnet is a range of IP addresses within your VPC. You launch your AWS resources, like EC2 instances, into subnets.
    - **Public Subnet:** A subnet is "public" if its associated route table has a route to an **Internet Gateway (IGW)**. This allows resources in the subnet to have direct outbound access to the internet and be reachable from the internet (if they have a public IP).
    - **Private Subnet:** A subnet is "private" if its route table does **not** have a route to an Internet Gateway. Resources in a private subnet cannot be reached directly from the internet. To access the internet, they typically need to route traffic through a NAT Gateway located in a public subnet.
- **Availability Zones (AZs):** A VPC spans all Availability Zones within a region. However, a specific subnet can only exist in **one** AZ. This is a critical concept for designing highly available architectures. You create subnets in different AZs and distribute your resources among them.

### **2. CIDR (Classless Inter-Domain Routing) Planning**

CIDR is the notation used to define the IP address range for your VPC and its subnets. It's represented by an IP address and a slash followed by a number (e.g., `10.0.0.0/16`).

- **How CIDR Works:**
    - The number after the slash (`/16`) represents the number of bits in the "prefix." The lower the number, the more available IP addresses there are.
    - `/16` = 65,536 total IPs
    - `/24` = 256 total IPs
    - `/28` = 16 total IPs
- **VPC and Subnet Sizing:**
    1. You first define a large CIDR block for your entire VPC (e.g., `10.0.0.0/16`). This CIDR block must come from the private IP address ranges defined in RFC 1918:
        - `10.0.0.0` to `10.255.255.255` (`10.0.0.0/8`)
        - `172.16.0.0` to `172.31.255.255` (`172.16.0.0/12`)
        - `192.168.0.0` to `192.168.255.255` (`192.168.0.0/16`)
    2. You then carve that large VPC block into smaller CIDR blocks for your individual subnets (e.g., `10.0.1.0/24`, `10.0.2.0/24`, etc.). A subnet's CIDR block cannot be larger than the VPC's CIDR block.
- **AWS Reserved IPs:** In any subnet you create, AWS reserves **5 IP addresses** that you cannot use.
    - `.0`: Network address.
    - `.1`: Reserved by AWS for the VPC router.
    - `.2`: Reserved by AWS for DNS.
    - `.3`: Reserved by AWS for future use.
    - `.255`: Network broadcast address.
    - **Rule of Thumb:** If a CIDR block has `X` total addresses, you can only use `X - 5` of them for your resources. A `/24` (256 addresses) has 251 usable addresses.

---

### **3. Key Networking Components**

- **Route Table:** A set of rules, called routes, that determine where network traffic from your subnet is directed. Each subnet must be associated with a route table.
    - **Default Route:** Every route table contains a default, local route (e.g., `10.0.0.0/16 -> local`) that allows all resources within the VPC to communicate with each other. This route cannot be deleted.
    - **Custom Routes:** To provide internet access, you add a route: `0.0.0.0/0 -> igw-xxxxxxxx` (for a public subnet) or `0.0.0.0/0 -> nat-xxxxxxxx` (for a private subnet).
- **Internet Gateway (IGW):** A horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. It provides a target in your route tables for internet-routable traffic. You can only have **one IGW per VPC**.
- **Default VPC:** When you create an AWS account, a "default VPC" is created for you in each region. It comes pre-configured with a public subnet in each AZ, an internet gateway, and a main route table, allowing you to launch public instances immediately.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A solutions architect needs to design a VPC with a private IP address range that can support approximately 65,000 IP addresses. Which of the following CIDR blocks should be chosen for the VPC?**

- A. `10.0.0.0/8`
- B. `10.0.0.0/16`
- C. `10.0.0.0/24`
- D. `10.0.0.0/28`

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The number after the slash indicates the size of the network. A smaller number means a larger network. A /16 CIDR block provides 2^(32-16) = 65,536 total IP addresses, which matches the requirement. A /8 would be too large, and a /24 (256 IPs) or /28 (16 IPs) would be too small.
</details>

---

**2. What makes a subnet a "public subnet"?**

- A. The subnet's CIDR block is from a public IP range.
- B. The subnet's associated route table has a route to an Internet Gateway.
- C. All instances in the subnet have an Elastic IP address.
- D. The "Public Subnet" option is checked during creation.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The defining characteristic of a public subnet is its route to the internet. If the subnet's route table contains a route that directs internet-bound traffic (0.0.0.0/0) to an Internet Gateway (IGW), it is considered a public subnet.
</details>

---

**3. An administrator creates a new subnet with a CIDR block of `192.168.10.0/24`. How many usable IP addresses are available for EC2 instances in this subnet?**

- A. 256
- B. 255
- C. 254
- D. 251

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** A /24 CIDR block contains 256 total IP addresses. However, AWS reserves the first four addresses (.0, .1, .2, .3) and the last address (.255) in every subnet for networking purposes. Therefore, the number of usable addresses is 256 - 5 = 251.
</details>

---

**4. A VPC spans across which of the following geographical boundaries?**

- A. A single Availability Zone.
- B. All Availability Zones within a single AWS Region.
- C. Multiple AWS Regions.
- D. A single physical data center.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A VPC is a regional resource. It is designed to span across all Availability Zones within the region where it is created, allowing you to build highly available architectures by placing resources in different AZs within the same VPC.
</details>

---

**5. What is the primary function of an Internet Gateway (IGW)?**

- A. To provide a private connection to on-premises data centers.
- B. To allow communication between two different VPCs.
- C. To allow resources within a VPC to communicate with the internet.
- D. To filter traffic at the subnet level.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** An Internet Gateway is the component that enables internet connectivity for a VPC. It is a highly available and scalable gateway that you attach to your VPC and then use as a target in your route tables for internet-bound traffic.
</details>

---

**6. Every route table in a VPC contains a default entry that cannot be deleted. What is the purpose of this entry?**

- A. To allow all resources within the VPC to communicate with each other.
- B. To provide a default route to the internet.
- C. To block all traffic by default.
- D. To allow access to the AWS management console.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** Every route table has a local route for the VPC's main CIDR block (e.g., 10.0.0.0/16 -> local). This route enables all instances inside the VPC to communicate with each other using their private IP addresses, regardless of which subnet they are in.
</details>

---

**7. How many Internet Gateways can be attached to a single VPC at a time?**

- A. One per Availability Zone.
- B. One per Subnet.
- C. Only one.
- D. As many as needed.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A VPC can have only one Internet Gateway attached to it at any given time. This single IGW serves as the internet connection for the entire VPC.
</details>

---

**8. You are designing a three-tier web application with a web layer, an application layer, and a database layer. For security, the database servers should not be directly accessible from the internet. In which type of subnet should the database servers be placed?**

- A. A public subnet.
- B. A private subnet.
- C. A VPC endpoint subnet.
- D. A hardware VPN subnet.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To prevent direct internet access, resources should be placed in a private subnet. A private subnet is one whose route table does not have a route to the Internet Gateway. The application layer servers (in a public or private subnet) would then communicate with the database servers using their private IP addresses.
</details>

---

**9. When you create a new AWS account, what networking components are automatically created for you in each region?**

- A. An empty VPC with no subnets.
- B. A default VPC with a public subnet in each Availability Zone, an Internet Gateway, and a main route table.
- C. A VPC with a private subnet in each Availability Zone and a NAT Gateway.
- D. No networking components are created; you must create everything manually.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To help new users get started quickly, AWS provides a default VPC in each region. This VPC is pre-configured with everything needed for basic public internet connectivity, including public subnets, an IGW, and routes.
</details>

---

**10. A subnet is associated with a route table. What is the function of the route table?**

- A. To filter inbound and outbound traffic based on port and protocol.
- B. To define the permissions for EC2 instances within the subnet.
- C. To specify the allowed paths for network traffic leaving the subnet.
- D. To assign IP addresses to resources in the subnet.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A route table acts like a signpost for network traffic. It contains a set of rules (routes) that tell the VPC's virtual router where to send packets destined for various IP addresses. For example, a route of 0.0.0.0/0 pointing to an IGW tells the router to send all non-local traffic to the internet.
</details>

---

**11. Which of the following is a valid private IP address range according to RFC 1918?**

- A. `8.8.8.0/24`
- B. `172.20.0.0/16`
- C. `203.0.113.0/24`
- D. `1.1.1.0/24`

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The private IP ranges are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16. The range 172.20.0.0/16 falls within the 172.16.0.0/12 block (which covers 172.16.0.0 to 172.31.255.255). All other options are public IP ranges.
</details>

---

**12. A subnet can only reside in a single Availability Zone.**

- A. True
- B. False

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** This is a fundamental rule of VPC design. While a VPC is regional, each subnet you create is tied to a single, specific Availability Zone. To build a highly available application, you must create multiple subnets in different AZs.
</details>

---

**13. Which AWS component is most analogous to a physical router in a traditional data center?**

- A. Security Group
- B. Network ACL
- C. Internet Gateway
- D. Route Table

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** A Route Table contains the rules that determine how traffic is directed, which is the primary function of a router. The VPC has an implicit virtual router that reads the route table to make its forwarding decisions.
</details>

---

**14. A VPC has a CIDR block of `10.10.0.0/16`. A solutions architect wants to create a subnet with 251 usable IP addresses. Which CIDR block should they choose for the subnet?**

- A. `10.10.10.0/16`
- B. `10.10.10.0/20`
- C. `10.10.10.0/24`
- D. `10.10.10.0/28`

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A /24 CIDR block provides 256 total IP addresses. After subtracting the 5 addresses reserved by AWS, this leaves 251 usable IP addresses, which matches the requirement.
</details>

---

**15. If you delete the main route table in a VPC, what happens?**

- A. All traffic in the VPC stops flowing.
- B. A new, default main route table is automatically created.
- C. You cannot delete the main route table.
- D. All subnets become private.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Every VPC must have a main route table. You are not allowed to delete the main route table. You can, however, replace the main route table with a different custom route table that you have created.
</details>

---

**16. An EC2 instance in a public subnet has a public IP address but cannot reach the internet. Its security group allows all outbound traffic, and its subnet's NACL also allows all outbound traffic. What is the most likely issue?**

- A. The instance does not have an IAM role attached.
- B. The subnet's route table is missing a route to the Internet Gateway.
- C. The VPC does not have a NAT Gateway.
- D. The instance needs an Elastic IP address.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** For an instance to reach the internet, its subnet must have a path to get there. This path is defined in the route table. The most common reason for this failure is a missing route for 0.0.0.0/0 that targets the VPC's Internet Gateway.
</details>

---

**17. Which reserved IP address in a subnet is used by the VPC's virtual router?**

- A. The first IP address (`.0`)
- B. The second IP address (`.1`)
- C. The third IP address (`.2`)
- D. The last IP address (`.255`)

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The second IP address in any subnet's CIDR range (the network address + 1) is always reserved by AWS to be the gateway address for the VPC router.
</details>

---

**18. What is the relationship between subnets and route tables?**

- A. A subnet can be associated with multiple route tables.
- B. A route table can be associated with multiple subnets, but a subnet can only be associated with one route table at a time.
- C. Each subnet must have its own unique route table.
- D. Route tables are associated with instances, not subnets.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This describes the many-to-one relationship. You can create a single route table (e.g., a "private" route table) and associate it with several different private subnets. However, any individual subnet can only follow the rules of one route table at any given time.
</details>

---

**19. What is the smallest CIDR block you can assign to a VPC?**

- A. `/16`
- B. `/24`
- C. `/28`
- D. `/32`

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The smallest allowed VPC size is a /28 CIDR block, which provides 16 total IP addresses (11 usable). The largest is a /16.
</details>

---

**20. A company needs to connect their VPC to another VPC in a different AWS account. Which AWS service would they use?**

- A. Internet Gateway
- B. VPC Peering or Transit Gateway
- C. NAT Gateway
- D. Direct Connect

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To connect VPCs together, you use either VPC Peering (for 1-to-1 connections) or AWS Transit Gateway (to connect many VPCs and on-premises networks to a central hub). An Internet Gateway (A) is for connecting to the public internet.
</details>

---

**21. An instance needs to download a software patch from the internet. The instance is in a private subnet. What component is required for the instance to access the internet?**

- A. An Internet Gateway attached directly to the private subnet.
- B. An Elastic IP address attached to the instance.
- C. A route in the private subnet's route table pointing to a NAT Gateway in a public subnet.
- D. A Security Group rule allowing outbound traffic.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Instances in a private subnet have no direct route to the internet. To allow them to initiate outbound connections (like for downloading patches) while still preventing inbound connections from the internet, you use a NAT (Network Address Translation) Gateway. The private subnet routes internet-bound traffic to the NAT Gateway, which resides in a public subnet and has an Elastic IP. The NAT Gateway then forwards the traffic to the internet on behalf of the private instance.
</details>

---

**22. Which IP address is used by AWS for the internal VPC DNS server?**

- A. The first IP of the CIDR block (`.0`)
- B. The second IP of the CIDR block (`.1`)
- C. The third IP of the CIDR block (`.2`)
- D. The fourth IP of the CIDR block (`.3`)

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The address at the base of the VPC CIDR range plus two (e.g., 10.0.0.2 for a 10.0.0.0/16 VPC) is always reserved for the Amazon-provided DNS server.
</details>

---