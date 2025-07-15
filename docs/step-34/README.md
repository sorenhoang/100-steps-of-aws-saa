# Step 34: Route Tables, NACL Rules, Security Groups

## 🎯 Objective

- [x] Master: **Route Tables, NACL Rules, Security Groups**

## 📘 Notes

### **Deep Dive: Route Tables, NACL Rules, & Security Groups**

When a network packet enters or leaves your VPC, it passes through a series of checks and decision points. Mastering the order and function of these components—Route Tables (the director), Network ACLs (the subnet guard), and Security Groups (the instance bodyguard)—is essential for building secure and functional network architectures.

---

### **The Order of Operations**

Think of traffic flow like a package delivery to an apartment building.

**For Inbound Traffic (from the internet to your instance):**

1. **Route Table:** The package first arrives at the VPC's "post office" (the virtual router). The Route Table associated with the public subnet looks at the destination (your instance's private IP) and confirms it's a local delivery, directing it to the correct "floor" (the subnet).
2. **Network ACL (NACL):** The package arrives at the "security desk for the floor" (the subnet). The NACL checks its **inbound** rules in numerical order. If the source IP and port are allowed, the package is let onto the floor. If denied, it's rejected.
3. **Security Group:** The package finally arrives at the "door of the specific apartment" (the instance). The Security Group checks its **inbound** rules. If the source and port are allowed, the door opens and the package is delivered.

**For Outbound Traffic (from your instance to the internet):**

1. **Security Group:** The instance sends a package out its "apartment door." The Security Group checks its **outbound** rules. If allowed, the package leaves the instance.
2. **Network ACL (NACL):** The package reaches the "security desk for the floor." The NACL checks its **outbound** rules in numerical order. If allowed, the package leaves the subnet.
3. **Route Table:** The package is now at the VPC's "post office" (the router). The Route Table looks at the destination (`0.0.0.0/0`) and directs the package to the correct exit (e.g., the Internet Gateway).

---

### **Recap of Key Characteristics**

- **Route Table:**
    - **Function:** Directs traffic (Routing). It doesn't allow or deny, it just points the way.
    - **Scope:** Associated with a subnet.
    - **Logic:** Uses the most specific route that matches the destination address. `10.0.0.0/16` is more specific than `0.0.0.0/0`.
- **Network ACL (NACL):**
    - **Function:** Filters traffic (Firewall).
    - **Scope:** Associated with a subnet (first/last line of defense for the subnet).
    - **State:** **Stateless**. You need explicit rules for both inbound and outbound return traffic.
    - **Rules:** Supports both `Allow` and `Deny` rules.
    - **Evaluation:** Rules are evaluated in **numerical order** (lowest first). The first matching rule is applied.
- **Security Group (SG):**
    - **Function:** Filters traffic (Firewall).
    - **Scope:** Associated with an instance's ENI (the last line of defense for the instance).
    - **State:** **Stateful**. If inbound traffic is allowed, the outbound response is automatically allowed.
    - **Rules:** Supports **`Allow` rules only**.
    - **Evaluation:** All rules are evaluated together. If any rule allows the traffic, it is permitted.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A packet is sent from the internet to an EC2 instance in a public subnet. What is the correct order of evaluation for the VPC components?**

- A. Security Group -> Network ACL -> Route Table
- B. Network ACL -> Security Group -> Route Table
- C. Route Table -> Network ACL -> Security Group
- D. Route Table -> Security Group -> Network ACL
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** For inbound traffic, the VPC's router first uses the Route Table to determine which subnet to send the packet to. The packet then hits the subnet boundary and is evaluated by the Network ACL. If allowed, it proceeds to the instance's ENI, where it is finally evaluated by the Security Group.
</details>
---

**2. An administrator wants to explicitly block a known malicious IP address from accessing any resources in an entire subnet. Which component should be used?**

- A. The VPC's main Route Table.
- B. A Security Group with a `Deny` rule.
- C. A Network ACL with a numbered `Deny` rule.
- D. An IAM policy.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Only Network ACLs support explicit Deny rules, making them the correct tool for blacklisting specific IP addresses at the subnet level. Security Groups do not have Deny rules.
</details>
---

**3. An EC2 instance successfully receives an HTTP request on port 80. The security group has an inbound rule allowing port 80. The outbound security group rules deny all traffic. Will the instance be able to send a response back to the client?**

- A. No, because the outbound security group rules will block the response.
- B. Yes, because Security Groups are stateful, and return traffic is automatically allowed.
- C. No, because the Network ACL will block the response.
- D. Yes, because HTTP is a stateless protocol.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This question tests the core concept of statefulness. Because Security Groups are stateful, they track connections. Since the inbound request was allowed, the security group automatically permits the corresponding outbound response traffic, regardless of what the outbound rules say.
</details>
---

**4. A Network ACL has the following inbound rules: Rule #100 `ALLOW from 10.0.2.5/32 on port 22` and Rule #110 `DENY from 10.0.2.0/24 on port 22`. An EC2 instance at `10.0.2.5` attempts to SSH into an instance in the subnet. What is the result?**

- A. The connection is allowed.
- B. The connection is denied.
- C. The connection is sometimes allowed and sometimes denied.
- D. Both rules are applied, and the `DENY` takes precedence.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** NACL rules are processed in numerical order. The traffic from 10.0.2.5 matches Rule #100 first. The action is ALLOW, so the packet is permitted to enter the subnet. Rule #110 is never evaluated for this packet because a match was already found.
</details>
---

**5. Which statement accurately describes the default rules for a newly created, custom Security Group?**

- A. It allows all inbound and all outbound traffic.
- B. It denies all inbound traffic and denies all outbound traffic.
- C. It allows all inbound traffic and denies all outbound traffic.
- D. It denies all inbound traffic and allows all outbound traffic.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** A custom security group you create is secure by default. It has no inbound rules, so it implicitly denies all inbound traffic. It has a single outbound rule that allows all outbound traffic.
</details>
---

**6. Which component determines the destination of a network packet, such as sending it to an Internet Gateway or a NAT Gateway?**

- A. Security Group
- B. Network ACL
- C. Route Table
- D. VPC Endpoint
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The Route Table is responsible for directing traffic. It does not filter traffic but simply looks at the destination IP address of a packet and forwards it to the specified target (like an IGW, NAT Gateway, or another local destination).
</details>
---

**7. An EC2 instance needs to send a response back to a client on the internet. The Security Group allows the traffic. However, the response is being blocked. The subnet's NACL has an inbound rule allowing the client's traffic, but no explicit outbound rules have been added to the custom NACL. Why is the response blocked?**

- A. Security Groups are stateless and need an outbound rule.
- B. A custom NACL denies all traffic by default, and an explicit outbound rule is needed for the return traffic.
- C. The instance does not have an Elastic IP address.
- D. The Route Table does not have a route for the return traffic.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** NACLs are stateless. This means that even if inbound traffic is allowed, the return traffic is treated as a completely new connection that must be evaluated against the outbound rules. Since a custom NACL denies all traffic by default, you must add an outbound Allow rule for the ephemeral port range (1024-65535) to permit the response.
</details>
---

**8. At which level do Security Groups and Network ACLs operate, respectively?**

- A. Subnet level and Instance level.
- B. Instance level and Subnet level.
- C. VPC level and Availability Zone level.
- D. Availability Zone level and VPC level.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Security Groups are applied at the instance (ENI) level. Network ACLs are applied at the subnet level.
</details>
---

**9. The final rule in every Network ACL is an implicit rule with the number `*`. What is the action of this rule?**

- A. `Allow`
- B. `Deny`
- C. `Log`
- D. `Forward`
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The final rule (*) is an implicit Deny. This ensures that any packet that does not match one of your numbered Allow rules is automatically dropped, enforcing a "deny by default" security posture.
</details>
---

**10. You have two instances in the same security group, which has a rule allowing all inbound traffic from itself. The instances are in different subnets. The NACL for Subnet A allows all traffic, but the NACL for Subnet B denies all traffic. Can Instance A successfully send a request to and receive a response from Instance B?**

- A. Yes, the security group rule allows full communication.
- B. No, the request from A to B will be blocked by Subnet B's NACL.
- C. The request from A to B will succeed, but the response from B to A will be blocked by Subnet B's NACL.
- D. Communication is only possible if they are in the same subnet.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The traffic flow is: Instance A -> SG Outbound (Allow) -> NACL A Outbound (Allow) -> Route Table -> NACL B Inbound (DENY). The request from Instance A is blocked by the inbound rules of Subnet B's Network ACL before it can ever reach Instance B's security group.
</details>
---

**11. Which statement about rule evaluation is correct?**

- A. Security Groups evaluate rules in numerical order; NACLs evaluate all rules.
- B. Security Groups only support `Allow` rules; NACLs support `Allow` and `Deny` rules.
- C. NACLs are stateful; Security Groups are stateless.
- D. NACLs apply to ENIs; Security Groups apply to subnets.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a fundamental difference. Security Groups work on a "whitelist" model; you can only add Allow rules. NACLs allow for more complex configurations by supporting both Allow and Deny rules, which is useful for blacklisting.
</details>
---

**12. When you create a new custom VPC, what is the state of its default Network ACL?**

- A. It denies all inbound and outbound traffic.
- B. It allows all inbound traffic but denies all outbound traffic.
- C. It allows all inbound and outbound traffic.
- D. There is no default NACL; you must create one.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The default NACL that is created with a VPC is configured to be permissive to avoid blocking traffic unexpectedly. It contains a rule #100 that allows all traffic and the final * deny rule. Therefore, it effectively allows all traffic in and out. A custom NACL you create yourself, however, will deny all traffic by default.
</details>
---

**13. A packet is leaving an EC2 instance, destined for the internet. What is the correct order of evaluation?**

- A. Route Table -> NACL -> Security Group
- B. Security Group -> NACL -> Route Table
- C. NACL -> Security Group -> Route Table
- D. Security Group -> Route Table -> NACL
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** For outbound traffic, the packet must first be allowed out of the instance's own firewall, the Security Group. Then it reaches the subnet boundary, where it is evaluated by the Network ACL. If allowed, it is passed to the VPC router, which uses the Route Table to direct it to the Internet Gateway.
</details>
---

**14. A security group rule specifies another security group (`sg-abc123`) as its source. What does this mean?**

- A. It allows traffic from any instance in any account that uses a security group with that ID.
- B. It allows traffic only from EC2 instances that have the `sg-abc123` security group attached and are in the same VPC.
- C. It allows traffic from the subnet associated with `sg-abc123`.
- D. The rule is invalid; sources must be CIDR blocks.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Referencing another security group as a source is a powerful way to build tiered application security. It means that any traffic originating from a network interface (ENI) that has sg-abc123 attached will be permitted, making it easy to allow communication between application layers (e.g., web tier to app tier).
</details>
---

**15. To allow an instance in a private subnet to receive a response from a web server on the internet it has just contacted, what NACL rule is necessary?**

- A. An inbound rule allowing traffic from the web server's IP on any port.
- B. An inbound rule allowing traffic on the ephemeral port range (1024-65535).
- C. An outbound rule allowing traffic on port 80/443.
- D. No rule is necessary as NACLs are stateful.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** NACLs are stateless. The initial outbound request (C) must be allowed. The web server will send its response back to the instance on a high-numbered, temporary "ephemeral" port. Therefore, you must have an inbound NACL rule that allows traffic on the ephemeral port range (1024-65535) for the response to get back into the subnet.
</details>
---

**16. Which component is MOST analogous to a traditional stateless firewall appliance that filters traffic for an entire network segment?**

- A. Security Group
- B. Route Table
- C. Internet Gateway
- D. Network ACL
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** A Network ACL is the component that most closely resembles a traditional firewall ACL. It operates at the network (subnet) level, is stateless, and processes rules in a numbered order to allow or deny traffic based on IP and port.
</details>
---

**17. If a subnet is not explicitly associated with a route table, what happens?**

- A. It has no routing and cannot communicate with anything.
- B. It is automatically associated with the VPC's main route table.
- C. The launch of any instance into that subnet will fail.
- D. It uses the route table of the default VPC.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Every VPC has one route table designated as the "main" route table. Any subnet that you create that is not explicitly associated with another custom route table will automatically inherit the rules from the main route table.
</details>
---

**18. What is the purpose of using different numbered rules in a Network ACL (e.g., 100, 200, 300)?**

- A. To logically group rules for easier reading.
- B. To define the order of precedence in which the rules are evaluated.
- C. To allow for more than 100 rules to be created.
- D. To assign a cost to each rule for billing purposes.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The number assigned to a NACL rule dictates its priority. Rules are processed strictly in ascending order, from the lowest number to the highest. This allows you to create specific Allow or Deny rules that take precedence over more general rules with higher numbers.
</details>
---

**19. An instance has two security groups attached. SG-A allows inbound port 22. SG-B allows inbound port 80. What traffic is allowed to the instance?**

- A. Only port 22.
- B. Only port 80.
- C. Both port 22 and port 80.
- D. Neither, because the rules conflict.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** When multiple security groups are attached to an instance, their rules are aggregated. The effect is a union of all the Allow rules. Since SG-A allows port 22 and SG-B allows port 80, the instance will permit traffic on both ports.
</details>
---

**20. A request to an EC2 instance is being denied. You have confirmed the Security Group allows the traffic. What is the next logical component to check?**

- A. The instance's IAM role.
- B. The subnet's Network ACL.
- C. The VPC's route table.
- D. The Internet Gateway's configuration.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Following the inbound traffic flow (Route Table -> NACL -> SG), if you have verified that the Security Group (the last step) is correct, the next logical place to look for the block is the step immediately before it: the Network ACL.
</details>
---

**21. Which component does not filter traffic but simply directs it?**

- A. Security Group
- B. Network ACL
- C. AWS WAF
- D. Route Table
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** A Route Table's function is purely for routing and directing traffic based on its destination. It does not perform any filtering or inspection. Security Groups, NACLs, and WAF are all types of firewalls that filter traffic.
</details>
---

**22. If you want to create a DMZ (demilitarized zone), you would typically use a combination of:**

- A. A public subnet, private subnet, security groups, and NACLs.
- B. Two VPCs connected with a peering connection.
- C. A single security group with complex rules.
- D. An AWS Direct Connect link.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** A DMZ is a classic network architecture pattern. In AWS, this is implemented by using a public subnet for bastion hosts or public-facing proxies, and a private subnet for the backend application servers. You then use a combination of NACLs and Security Groups to strictly control the flow of traffic between the internet, the DMZ (public subnet), and the internal trusted zone (private subnet).
</details>