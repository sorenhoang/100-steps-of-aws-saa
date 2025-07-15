# Step 35: VPC Peering & Transit Gateway

## 🎯 Objective
- [x] Master: **VPC Peering & Transit Gateway**

## 📘 Notes
### **Deep Dive: VPC Peering & Transit Gateway**

As your AWS footprint grows, you will inevitably have resources in different VPCs that need to communicate with each other. AWS provides two main solutions for this: VPC Peering for simple, direct connections, and AWS Transit Gateway for scalable, hub-and-spoke networking.

---

### **1. VPC Peering** 🤝

A VPC Peering connection is a networking connection between **two VPCs** that enables you to route traffic between them using private IPv4 or IPv6 addresses. It makes the two VPCs behave as if they are on the same network.

- **Architecture:** A direct, 1-to-1 connection. If you have three VPCs (A, B, C) and you want them all to communicate, you need to create three separate peering connections (A-B, B-C, and A-C). This creates a "full mesh" topology.
- **Key Characteristics:**
    - **Non-Transitive:** This is the most important concept. If VPC A is peered with VPC B, and VPC B is peered with VPC C, **VPC A cannot communicate with VPC C** through VPC B. A direct peering connection must be established between A and C.
    - **No Overlapping CIDRs:** The CIDR blocks of the two VPCs in a peering connection cannot overlap.
    - **Regional & Inter-Region:** You can peer VPCs in the same region or in different AWS regions.
    - **Routing:** You must manually update the **route tables** in each VPC to direct traffic destined for the other VPC's CIDR block to the peering connection.
- **Use Case:** Simple scenarios involving a small number of VPCs that need to communicate directly. It's easy to set up for two or three VPCs.
- **Limitation:** Becomes very complex and difficult to manage as the number of VPCs grows. The number of connections required for a full mesh is `n * (n-1) / 2`, where `n` is the number of VPCs.

---

### **2. AWS Transit Gateway (TGW)** HUB

An AWS Transit Gateway is a managed service that acts as a **central cloud router** or a **network transit hub**. It simplifies your network architecture by allowing you to connect thousands of VPCs and on-premises networks through a single, central gateway.

- **Architecture:** A **hub-and-spoke** model. The Transit Gateway is the "hub," and your VPCs, VPN connections, and Direct Connect gateways are the "spokes."
- **How it Works:**
    1. You create a single Transit Gateway in a region.
    2. You "attach" your VPCs and on-premises connections (VPN, Direct Connect) to the TGW.
    3. The TGW has its own route tables that control how traffic is routed between the attachments. By default, all attached spokes can communicate with each other.
- **Key Characteristics:**
    - **Transitive Routing:** This is its primary advantage. By default, any resource attached to the TGW can communicate with any other resource attached to the TGW. If VPC A and VPC C are both attached to the TGW, they can communicate with each other without a direct peering connection.
    - **Scalability:** Drastically simplifies network management at scale. To connect a new VPC, you just create one attachment to the TGW instead of creating multiple peering connections.
    - **Centralization:** Acts as a central point for network connectivity and monitoring. You can also connect multiple TGWs together across different regions (inter-region peering).
- **Use Case:** The standard solution for connecting a large number of VPCs and on-premises networks. It is the modern, scalable way to manage cloud networking.

---

### **Summary of Key Differences**

| Feature | VPC Peering | AWS Transit Gateway |
| --- | --- | --- |
| **Architecture** | 1-to-1 Mesh | Hub-and-Spoke |
| **Transitivity** | **Non-Transitive** | **Transitive** |
| **Scalability** | Poor (complex to manage at scale) | Excellent (scales to thousands of VPCs) |
| **On-Premises** | Does not directly connect to VPN/DX | Connects to VPN/DX as attachments |
| **Management** | Manage many individual connections | Manage one central gateway & attachments |
| **Use Case** | Simple, few-VPC connections | Complex, many-VPC, hybrid networks |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company has 3 VPCs (A, B, and C). VPC A is peered with VPC B, and VPC B is peered with VPC C. Can an EC2 instance in VPC A communicate with an EC2 instance in VPC C?**

- A. Yes, because traffic will route through VPC B.
- B. No, because VPC Peering is non-transitive.
- C. Yes, but only if all VPCs are in the same Availability Zone.
- D. No, because the CIDR blocks will overlap.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This question tests the most critical concept of VPC Peering: it is non-transitive. The peering connection between A and B does not extend to the connection between B and C. For A and C to communicate, a direct peering connection must be established between them.
</details>
---

**2. A large enterprise has over 50 VPCs spread across multiple AWS accounts. They also have several on-premises data centers connected via AWS Direct Connect. They need a scalable solution to allow any VPC to communicate with any other VPC and with the on-premises networks. What is the most appropriate service?**

- A. A full mesh of VPC Peering connections.
- B. An Internet Gateway for each VPC.
- C. AWS Transit Gateway.
- D. AWS Storage Gateway.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This is the ideal use case for AWS Transit Gateway. Managing connectivity for 50+ VPCs and on-premises networks with VPC Peering would be incredibly complex (over 1200 peering connections). A Transit Gateway acts as a central cloud router, simplifying this into a manageable hub-and-spoke architecture where each VPC and on-premises connection just needs one attachment to the central hub.
</details>
---

**3. What is a primary limitation of VPC Peering that is solved by AWS Transit Gateway?**

- A. VPC Peering does not support cross-region connections.
- B. VPC Peering does not allow the use of security groups.
- C. VPC Peering becomes complex to manage at scale due to its non-transitive, mesh architecture.
- D. VPC Peering does not encrypt traffic.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The main drawback of VPC Peering is its lack of scalability. As the number of VPCs increases, the number of required peering connections grows exponentially, creating a complex "mesh" that is difficult to manage and troubleshoot. Transit Gateway's hub-and-spoke model solves this by centralizing connectivity.
</details>
---

**4. After establishing a VPC Peering connection between VPC-A and VPC-B, what additional step is required to enable traffic flow?**

- A. Nothing, traffic will flow automatically.
- B. Update the security groups in each VPC to allow traffic from the other VPC's CIDR block.
- C. Update the route tables in each VPC to direct traffic destined for the other VPC to the peering connection.
- D. Attach an Internet Gateway to the peering connection.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Creating the peering connection is only the first step. It creates a potential path, but you must tell your VPCs how to use it. This is done by updating the route tables. In VPC-A, you must add a route for VPC-B's CIDR block pointing to the peering connection, and vice-versa in VPC-B.
</details>
---

**5. Which AWS networking service acts as a "cloud router"?**

- A. Internet Gateway
- B. NAT Gateway
- C. VPC Peering Connection
- D. AWS Transit Gateway
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** AWS Transit Gateway is explicitly designed to function as a regional virtual router. It sits at the center of your network and makes intelligent routing decisions to forward traffic between VPCs, VPNs, and Direct Connect connections, much like a physical core router in a traditional data center.
</details>
---

**6. A company has two VPCs, a "Shared Services" VPC (10.1.0.0/16) and a "Production" VPC (10.2.0.0/16). They want to establish a peering connection. What must be true about their CIDR blocks?**

- A. The CIDR blocks must be identical.
- B. The CIDR blocks must not overlap.
- C. The CIDR block for the Production VPC must be smaller.
- D. The CIDR blocks do not matter for VPC Peering.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A fundamental requirement for VPC Peering is that the two VPCs involved must not have overlapping CIDR blocks. If they overlapped, the VPC router would not be able to determine whether to route a packet locally or across the peering connection. The CIDRs 10.1.0.0/16 and 10.2.0.0/16 do not overlap, so a peering connection is possible.
</details>
---

**7. In a hub-and-spoke network topology using AWS Transit Gateway, what represents the "hub"?**

- A. A designated VPC that all other VPCs peer with.
- B. The on-premises data center.
- C. The Transit Gateway itself.
- D. The Internet Gateway.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** In this model, the Transit Gateway is the central hub. All other network components, such as your VPCs and VPN connections, are the "spokes" that attach to this central hub.
</details>
---

**8. Can a security group in a peered VPC be referenced in a security group rule in another VPC?**

- A. No, security groups can only be referenced within the same VPC.
- B. Yes, if it is a cross-region peering connection.
- C. Yes, this is a supported feature of VPC Peering.
- D. Only if a Transit Gateway is also used.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Yes, you can reference a security group from a peered VPC in a security group rule. This allows for more granular control than just using CIDR blocks. For example, you can allow traffic from the "web-sg" in VPC-A to the "database-sg" in VPC-B. You must use the security group's ID in the rule.
</details>
---

**9. How does AWS Transit Gateway simplify connecting an on-premises network to multiple VPCs?**

- A. It removes the need for an AWS Direct Connect or VPN connection.
- B. It allows you to establish a single VPN or Direct Connect attachment to the Transit Gateway, which then provides connectivity to all attached VPCs.
- C. It requires a separate VPN connection to be established to each individual VPC.
- D. It automatically encrypts all traffic between on-premises and the VPCs.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Instead of managing a separate VPN or Direct Connect Virtual Interface (VIF) for every VPC you need to connect to, you can establish a single connection to the Transit Gateway. Due to the TGW's transitive nature, that single on-premises connection can then reach all VPCs that are also attached to the TGW, drastically simplifying hybrid connectivity.
</details>
---

**10. When should a company choose VPC Peering over AWS Transit Gateway?**

- A. When they need to connect more than 100 VPCs.
- B. When they have a simple architecture with only two or three VPCs that need to communicate directly.
- C. When they require transitive routing.
- D. When they need to connect to their on-premises network.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** VPC Peering is a perfectly valid and cost-effective solution for simple scenarios. If you only have a handful of VPCs and do not foresee a large growth in complexity, setting up a few direct peering connections is often simpler and cheaper than deploying a Transit Gateway.
</details>
---

**11. What is required for an EC2 instance in VPC-A to resolve a private DNS hostname of an EC2 instance in peered VPC-B?**

- A. This happens automatically when the peering connection is established.
- B. You must enable the "DNS resolution" option on the VPC Peering connection.
- C. You must set up a Route 53 private hosted zone and share it between the VPCs.
- D. This is not possible.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** By default, DNS resolution over a peering connection is disabled. To allow instances in one VPC to resolve the private DNS names of instances in the other VPC, you must explicitly enable the DNS resolution setting for the peering connection.
</details>
---

**12. Which of the following statements about Transit Gateway is TRUE?**

- A. It operates on a 1-to-1 mesh topology.
- B. It is a zonal service and can become a single point of failure.
- C. It simplifies network management by acting as a central hub for routing.
- D. It is free to use.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The primary benefit and true statement is that a Transit Gateway simplifies network management by centralizing connectivity in a hub-and-spoke model. It uses a hub-spoke model (A is false), is a highly available regional service (B is false), and is a paid service with charges per attachment and per GB of data processed (D is false).
</details>
---

**13. You have peered VPC-A with VPC-B. You add a route to VPC-A's route table for VPC-B's CIDR. However, you forget to add a route in VPC-B's route table for VPC-A's CIDR. What is the result?**

- A. Full two-way communication will work.
- B. No communication will work in either direction.
- C. Traffic can flow from VPC-A to VPC-B, but return traffic from B to A will fail.
- D. The peering connection will enter a "failed" state.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Routing must be configured on both sides. An instance in VPC-A can send a packet to VPC-B because its route table knows where to send it. However, when the instance in VPC-B tries to send a response back, its route table has no entry for VPC-A's CIDR, so it doesn't know how to route the packet and the connection fails.
</details>
---

**14. A company wants to connect VPCs in `us-east-1` and `eu-west-1`. Which services can achieve this? (Select TWO)**

- A. Inter-Region VPC Peering
- B. AWS Direct Connect
- C. An Application Load Balancer
- D. AWS Transit Gateway Peering
- E. An Internet Gateway
<details>
<summary>View Answer</summary>

**Answers:** A and D

**Explanation:** To connect VPCs in different regions, you can use Inter-Region VPC Peering for a direct 1-to-1 connection. Alternatively, if you have a Transit Gateway in each region, you can establish a Transit Gateway Peering connection between them, which then connects all the VPCs attached to each TGW.
</details>
---

**15. A packet is sent from a VPC attached to a Transit Gateway to an on-premises network also attached to the TGW. What component makes the final routing decision?**

- A. The VPC's route table.
- B. The Transit Gateway's route table.
- C. The on-premises firewall.
- D. The Direct Connect Gateway.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Once traffic from a spoke (the VPC) arrives at the hub (the TGW), the Transit Gateway's route table is consulted to determine where to send the packet next. In this case, it would have a route for the on-premises network's CIDR that points to the Direct Connect or VPN attachment.
</details>
---

**16. What is the architecture of AWS Transit Gateway?**

- A. Point-to-Point
- B. Full Mesh
- C. Hub-and-Spoke
- D. Linear Bus
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The Transit Gateway functions as a central hub, and the VPCs and on-premises connections act as the spokes. This is its defining architectural model.
</details>
---

**17. Can you create a peering connection between two VPCs that have the same CIDR block `10.0.0.0/16`?**

- A. Yes, but only if they are in different regions.
- B. Yes, but you must use a Transit Gateway to resolve the conflict.
- C. No, VPC Peering requires the VPCs to have non-overlapping CIDR blocks.
- D. Yes, as long as the subnets have different CIDR blocks.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This is a hard rule for VPC Peering. The IP address spaces of the two VPCs cannot overlap. The VPC router would be unable to determine if an IP address was local or in the remote peered VPC.
</details>
---

**18. Which service would you use if you needed to create isolated routing domains within a central networking hub, for example, to keep production and development traffic separate even though they are attached to the same hub?**

- A. VPC Peering with route table filtering.
- B. AWS Transit Gateway with multiple route tables.
- C. Security Groups with references to peered VPCs.
- D. Network ACLs.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A powerful feature of AWS Transit Gateway is the ability to create multiple route tables within the TGW itself. You can associate different attachments (like your dev VPCs) with one route table and other attachments (like your prod VPCs) with a different route table. This allows you to create isolated routing domains and control which spokes can communicate with each other, all from a central point.
</details>
---

**19. When establishing a VPC peering connection between two VPCs in different AWS accounts, what is required?**

- A. Both accounts must be part of the same AWS Organization.
- B. The requester account sends an invitation, which must be accepted by the acceptor account.
- C. A support ticket must be opened with AWS to authorize the connection.
- D. You must use a Transit Gateway for cross-account connections.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Cross-account peering follows a request/accept model. The owner of one VPC initiates a peering request to the VPC in the other account. The owner of the second VPC must then explicitly accept this request before the connection becomes active.
</details>
---

**20. A company has 10 VPCs. How many VPC Peering connections are needed to create a full mesh where every VPC can talk to every other VPC?**

- A. 10
- B. 20
- C. 45
- D. 100
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The formula for the number of connections in a full mesh is n * (n-1) / 2. With n=10, the calculation is 10 * 9 / 2 = 10 * 9 / 2 = 90 / 2 = 45. This demonstrates how quickly peering becomes unmanageable.
</details>
---

**21. A VPC needs to connect to an on-premises data center. Which service is the "spoke" that attaches this connection to a Transit Gateway?**

- A. A VPN attachment or a Direct Connect attachment.
- B. An Internet Gateway attachment.
- C. A VPC Peering attachment.
- D. A NAT Gateway attachment.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** A Transit Gateway connects to your on-premises network via either a VPN attachment (for an IPsec VPN connection) or a Direct Connect attachment (for a Direct Connect Gateway). These attachments act as the spokes for your hybrid connections.
</details>
---

**22. Which service acts as a single point for managing and monitoring network connectivity between VPCs and on-premises networks at scale?**

- A. Amazon CloudWatch
- B. VPC Flow Logs
- C. AWS Transit Gateway Network Manager
- D. AWS Organizations
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** AWS Transit Gateway Network Manager is a feature associated with Transit Gateway that provides a centralized management dashboard. It allows you to visualize your global network (both AWS and on-premises), monitor its health, and view connectivity metrics from a single pane of glass.
</details>
