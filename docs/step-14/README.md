# Step 14: Security Groups vs NACL

## 🎯 Objective

- [x] Master: **Security Groups vs NACL**

## 📘 Notes

### **Deep Dive: Security Groups vs. Network ACLs (NACLs)**

In AWS, both Security Groups and Network Access Control Lists (NACLs) act as virtual firewalls to control traffic to and from your resources. However, they operate at different levels of your VPC and have fundamentally different characteristics. A solutions architect must know which to use and how they interact.

### **1. Security Groups (SGs)** 🚪

A Security Group acts as a firewall for your **EC2 instances**. It controls inbound and outbound traffic at the instance level.

- **Level:** Operates at the **instance level** (or more specifically, the Elastic Network Interface - ENI level).
- **Statefulness:** **Stateful**. This is the most important concept. If you allow an inbound traffic request, the corresponding outbound response traffic is **automatically allowed**, regardless of any outbound rules. You don't need to create a separate outbound rule for the response. For example, if you allow inbound TCP port 80 for a web server, the return traffic on a high-numbered ephemeral port is automatically permitted.
- **Rules:** You can only create **`Allow` rules**. You cannot create `Deny` rules. By default, all traffic is denied until you add an `Allow` rule.
- **Evaluation:** **All rules are evaluated** before a decision is made. For example, if one rule allows SSH from IP A and another rule allows SSH from IP B, a request from IP B will be allowed.
- **Scope:** You can assign up to 5 security groups to an instance. The source or destination of a rule can be an IP range, another Security Group, or a prefix list.

### **2. Network ACLs (NACLs)** 🧱

A Network ACL acts as a firewall for a **subnet**. It controls inbound and outbound traffic at the subnet level, acting as the first line of defense for anything entering or leaving the subnet.

- **Level:** Operates at the **subnet level**. It applies to all instances within the associated subnet.
- **Statefulness:** **Stateless**. This is the other critical concept. Return traffic must be **explicitly allowed** by a corresponding outbound or inbound rule. If you allow inbound traffic on TCP port 80, you must also create a separate **outbound rule** to allow traffic on the ephemeral ports (1024-65535) for the response to get out.
- **Rules:** You can create both **`Allow` and `Deny` rules**. This allows you to explicitly block a known malicious IP address.
- **Evaluation:** Rules are evaluated in **numerical order**, from the lowest number to the highest. The first rule that matches the traffic is immediately applied, and no further rules are processed. For example, if rule #90 denies traffic from an IP and rule #100 allows it, the traffic will be **denied** because rule #90 is processed first.
- **Default NACL:** The default NACL that comes with your VPC **allows all inbound and outbound traffic**. A custom NACL, when first created, **denies all inbound and outbound traffic** until you add rules.

### **Summary of Key Differences**

| Feature             | Security Group (SG)                                      | Network ACL (NACL)                                                   |
| ------------------- | -------------------------------------------------------- | -------------------------------------------------------------------- |
| **State**           | **Stateful**                                             | **Stateless**                                                        |
| **Level**           | Instance / ENI level                                     | Subnet level                                                         |
| **Allowed Rules**   | `Allow` rules only                                       | `Allow` and `Deny` rules                                             |
| **Rule Evaluation** | All rules are processed                                  | Processed in number order                                            |
| **Default**         | Denies all inbound traffic. Allows all outbound traffic. | Allows all traffic (default NACL). Denies all traffic (custom NACL). |
| **Application**     | Can be assigned to an instance.                          | Automatically applies to all instances in its associated subnet.     |

### **How They Work Together**

When traffic flows into your VPC, it passes through these firewalls in a specific order:

1. **Inbound Traffic:**
   - **NACL:** Traffic arriving at the subnet boundary is evaluated against the inbound rules of the NACL associated with the subnet. If allowed, it proceeds.
   - **Security Group:** Traffic arriving at the instance's ENI is then evaluated against the inbound rules of the instance's security group. If allowed, it reaches the instance.
2. **Outbound Traffic:**
   - **Security Group:** Traffic leaving the instance is first evaluated against the outbound rules of its security group.
   - **NACL:** If allowed by the SG, the traffic then reaches the subnet boundary and is evaluated against the outbound NACL rules.

For traffic to get through, it must be allowed at **both** levels.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A web server instance is not responding to HTTP requests from the internet. You have confirmed that its security group allows inbound traffic on port 80 from `0.0.0.0/0`. The instance itself is running and the web server process is active. What is the most likely cause of the issue?**

- A. The security group's outbound rules are blocking the response traffic.
- B. The Network ACL associated with the server's subnet does not have a rule to allow inbound traffic on port 80.
- C. The instance does not have an Elastic IP address.
- D. A Service Control Policy (SCP) is blocking HTTP traffic.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For traffic to reach an instance, it must be allowed by both the Network ACL (at the subnet level) and the Security Group (at the instance level). Since the Security Group is correctly configured, the next logical place to check is the NACL. If the NACL is blocking inbound port 80, the traffic will never reach the instance's security group. Security groups are stateful, so outbound rules (A) are not needed for response traffic.

</details>

---

**2. A security team needs to explicitly block a list of known malicious IP addresses from accessing any resource in a specific subnet. What is the most effective way to do this?**

- A. Create a security group rule with a `Deny` action for each IP address.
- B. Create a Network ACL with numbered `Deny` rules for each malicious IP address.
- C. Use AWS WAF to block the IP addresses.
- D. Use an IAM policy to deny access from the malicious IPs.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Only Network ACLs support explicit Deny rules. This is their primary use case for blacklisting. You would create numbered rules (e.g., starting at 100) with a Deny effect for each IP address. Security groups (A) do not support Deny rules. AWS WAF (C) operates at a higher level (for services like Application Load Balancer or CloudFront) and is not the primary tool for subnet-level IP blocking.

</details>

---

**3. What does it mean for a Security Group to be "stateful"?**

- A. It keeps a log of all accepted and rejected traffic.
- B. If you allow inbound traffic, the corresponding outbound return traffic is automatically allowed.
- C. It requires you to create explicit rules for both request and response traffic.
- D. Its rules are processed in a specific numerical order.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** "Stateful" means the firewall maintains a connection tracking table. When it sees an incoming request that matches an Allow rule, it remembers that connection. When the instance sends a response back to the original source, the security group recognizes it as part of an established connection and allows it out, regardless of the outbound rules.

</details>

---

**4. A Network ACL has the following two rules: Rule #100 `ALLOW TCP port 22 from 0.0.0.0/0` and Rule #200 `DENY TCP port 22 from 0.0.0.0/0`. What is the result for an incoming SSH connection?**

- A. The connection is allowed because the `ALLOW` rule has a lower number.
- B. The connection is denied because a `DENY` rule always overrides an `ALLOW` rule.
- C. The connection is allowed because all rules are evaluated.
- D. The connection is denied because the `DENY` rule has a higher number.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Network ACLs process rules in numerical order, from lowest to highest. The first rule that matches the traffic is applied, and all further processing stops. In this case, the incoming SSH traffic matches rule #100 first. The action is ALLOW, so the traffic is permitted to enter the subnet, and rule #200 is never evaluated.

</details>

---

**5. Which statement accurately describes the default configuration of security groups and NACLs in a default VPC?**

- A. The default security group denies all traffic, and the default NACL denies all traffic.
- B. The default security group allows all traffic from other members of the same group, and the default NACL allows all traffic.
- C. The default security group allows all inbound traffic, and the default NACL allows all inbound traffic.
- D. The default security group denies all inbound traffic, and the default NACL denies all inbound traffic.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The default security group has a rule that allows all inbound traffic from other resources assigned to the same security group. It also allows all outbound traffic. The default Network ACL that is created with a VPC is configured to ALLOW all inbound and all outbound traffic. This makes the initial environment permissive for ease of use.

</details>

---

**6. An EC2 instance in a public subnet can successfully connect to the internet, but it cannot receive any inbound SSH traffic from your office IP. The security group attached to the instance allows inbound SSH on port 22 from your office IP. What is the most likely problem?**

- A. The subnet's NACL is blocking inbound traffic on port 22.
- B. The subnet's NACL is blocking outbound traffic on ephemeral ports (1024-65535).
- C. The security group is stateless.
- D. The Internet Gateway's route table is misconfigured.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** If the instance can reach the internet, its outbound path (SG outbound -> NACL outbound -> IGW) is working. The problem is with the inbound path. Since the SG is correctly configured to allow the traffic, the next layer of defense is the NACL. It is most likely that the NACL is blocking inbound TCP port 22. Because NACLs are stateless, you might also have a problem with the outbound ephemeral port rule (B), but the inbound block is the first thing to check.

</details>

---

**7. To which AWS resource is a Security Group directly applied?**

- A. A Subnet
- B. An Availability Zone
- C. An Elastic Network Interface (ENI)
- D. A VPC

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While we often say a Security Group is applied to an "instance," it is more precisely applied to the instance's Elastic Network Interface (ENI). An instance can have multiple ENIs, and each ENI can have its own set of security groups. NACLs, in contrast, are applied to subnets.

</details>

---

**8. When creating a brand new, custom Network ACL, what is its initial state?**

- A. It allows all inbound and outbound traffic.
- B. It denies all inbound traffic but allows all outbound traffic.
- C. It denies all inbound and all outbound traffic.
- D. It inherits the rules from the default NACL.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Unlike the default NACL, a custom NACL is created in a "deny by default" state. It contains no rules, and its final rule (\*) is an implicit Deny. Therefore, until you add explicit Allow rules, it will block all traffic from entering or leaving any subnet it is associated with.

</details>

---

**9. What does it mean for a Network ACL to be "stateless"?**

- A. It cannot be modified after it is created.
- B. It does not inspect the contents of the packets.
- C. It does not track connections, meaning return traffic must be explicitly allowed by a rule.
- D. It only applies to stateless protocols like UDP.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** "Stateless" means the firewall examines each packet individually without any context of previous packets. When an inbound request is allowed, the NACL does not remember it. Therefore, when the instance sends a response, that outbound packet is evaluated fresh against the outbound rules. You must have an explicit Allow rule for that return traffic.

</details>

---

**10. You have two EC2 instances, Instance-A and Instance-B, in the same subnet and the same security group. The security group has a single inbound rule that allows all traffic where the source is the security group's own ID. What is the result?**

- A. The instances cannot communicate because security groups cannot reference themselves.
- B. Instance-A can communicate with Instance-B, and Instance-B can communicate with Instance-A.
- C. Instance-A can send traffic to Instance-B, but cannot receive traffic back.
- D. The instances can only communicate over the public internet.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a very common and powerful configuration. By setting the source of an Allow rule to the security group's own ID, you are effectively allowing all instances within that security group to communicate freely with each other over any port, without opening them up to external traffic. Because security groups are stateful, two-way communication is automatically enabled.

</details>

---

**11. Which of the following is a valid source or destination for a Security Group rule?**

- A. Another Security Group ID.
- B. A Subnet ID.
- C. An Availability Zone ID.
- D. A User ARN.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Security group rules can reference an IP range (CIDR), a specific prefix list, or, very commonly, another Security Group ID. This allows you to build tiered architectures (e.g., allowing traffic from the "Web-SG" into the "App-SG"). Subnets and AZs are not valid targets.

</details>

---

**12. An EC2 instance needs to make a request to a database server. The database security group allows inbound traffic on port 3306 from the EC2 instance's security group. The NACL for the database subnet allows inbound port 3306 but denies all outbound traffic. What happens?**

- A. The EC2 instance can connect to the database, and the database can send a response.
- B. The EC2 instance cannot connect to the database.
- C. The EC2 instance can connect, but the database's response will be blocked by the NACL.
- D. The database can send a response, but it will be blocked by its security group.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The initial inbound request from the EC2 instance will be allowed by the NACL (inbound 3306) and the SG. However, when the database tries to send a response back, the traffic will be blocked by the outbound NACL rule (DENY all). Because NACLs are stateless, this outbound response is not automatically permitted.

</details>

---

**13. How many subnets can a single NACL be associated with?**

- A. Only one.
- B. Up to 5.
- C. Many; a NACL can be associated with multiple subnets.
- D. It depends on the number of rules in the NACL.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** You can associate a single NACL with multiple subnets. This is useful for applying a consistent set of baseline network rules to several subnets at once. However, a subnet can only be associated with one NACL at a time.

</details>

---

**14. A Network ACL's highest numbered rule is #32767. What is the purpose of the final, un-numbered `*` rule?**

- A. To allow all traffic that was not matched by a previous rule.
- B. To ensure that all traffic is logged.
- C. To deny all traffic that was not explicitly allowed by a preceding rule.
- D. To forward unmatched traffic to the default NACL.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Every NACL has an implicit, final rule represented by an asterisk (\*). This rule ensures that if a packet does not match any of the numbered rules you've created, it will be denied. This enforces a "deny by default" posture for any traffic you haven't explicitly allowed.

</details>

---

**15. You need to allow an EC2 instance to receive HTTP traffic (port 80) and send responses back to clients on the internet. Your subnet's NACL allows all traffic. What is the minimum configuration for the instance's security group?**

- A. Allow inbound port 80. Allow outbound port 80.
- B. Allow inbound port 80. No outbound rule is needed.
- C. Allow inbound port 80. Allow outbound on all ports.
- D. Allow inbound on all ports. Allow outbound on all ports.

<details>
<summary>View Answer</summary>

**Answer: B or C**

**Explanation:** Because Security Groups are stateful, you only need to create the inbound rule (Allow port 80). The return traffic will be automatically permitted. By default, a new security group already has an "Allow all" outbound rule, so C is also correct and represents the default state. The absolute minimum change you need to make is just adding the inbound rule, making B the most precise answer in spirit, but C is the state of a default SG after adding the inbound rule. Both are valid configurations that will work.

</details>

---

**16. What is the primary function of a Security Group?**

- A. To act as a firewall at the subnet level.
- B. To define IAM permissions for an EC2 instance.
- C. To act as a stateful firewall at the instance/ENI level.
- D. To filter traffic based on numbered allow/deny rules.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the core definition. A Security Group is a stateful firewall that controls access directly to an instance's network interface. Subnet-level firewalls (A) and numbered allow/deny rules (D) describe NACLs.

</details>

---

**17. If you associate a new subnet with a custom NACL that has no rules defined, what traffic is allowed?**

- A. All traffic is allowed.
- B. No traffic is allowed in or out.
- C. Only traffic from within the same VPC is allowed.
- D. Only outbound traffic is allowed.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A newly created custom NACL is empty, meaning all traffic will fall through to the final implicit Deny rule (\*). Therefore, no traffic will be able to enter or leave the subnet until you add explicit Allow rules.

</details>

---

**18. Which statement about stateful and stateless firewalls is correct?**

- A. Security Groups are stateless, and NACLs are stateful.
- B. Security Groups are stateful, and NACLs are stateless.
- C. Both Security Groups and NACLs are stateful.
- D. Both Security Groups and NACLs are stateless.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the most fundamental distinction between the two services and a concept you must memorize. Security Groups track connections (stateful), while NACLs inspect each packet independently (stateless).

</details>

---

**19. A developer can SSH into an instance in a private subnet from a bastion host in a public subnet. However, the instance in the private subnet cannot connect to the internet to download software updates. What is the most likely cause?**

- A. The security group for the private instance is blocking outbound internet traffic.
- B. The private subnet does not have a route to a NAT Gateway or NAT Instance.
- C. The NACL for the private subnet is blocking outbound internet traffic.
- D. The security group for the bastion host is blocking traffic.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For an instance in a private subnet to access the internet, it needs a route to a device that can perform Network Address Translation (NAT). This is typically a NAT Gateway located in a public subnet. The problem described is a routing issue, not a firewall issue, since the developer can already connect from the bastion.

</details>

---

**20. When is a Network ACL rule evaluated?**

- A. After the traffic has been processed by the security group.
- B. Only for traffic that is destined for the public internet.
- C. As soon as traffic attempts to cross the subnet boundary (in or out).
- D. Only when a new instance is launched in the subnet.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** NACLs are the gatekeepers of the subnet. They are the first line of defense for inbound traffic and the last line of defense for outbound traffic. They evaluate every packet that tries to enter or leave the subnet.

</details>

---

**21. You have an application server and a database server in the same subnet. The application server's security group allows outbound traffic on port 1433 to the database server's security group. The database server's security group allows inbound traffic on port 1433 from the application server's security group. The subnet's NACL allows all traffic. Can they communicate?**

- A. Yes, because the security groups are correctly configured to allow the traffic.
- B. No, instances in the same subnet cannot communicate.
- C. No, the NACL will block the traffic because it's stateless.
- D. Yes, but only if they have public IP addresses.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** The configuration is perfect. By referencing each other's security groups, the instances are granted precise access. Since SGs are stateful, the return traffic is automatically handled. The NACL allows all traffic, so it does not interfere. Therefore, communication will be successful.

</details>

---

**22. How many Security Groups can be attached to an EC2 instance?**

- A. Only one.
- B. Up to 5.
- C. Up to 10.
- D. There is no limit.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** You can associate up to 5 security groups with a single network interface, which in most cases means up to 5 for a single instance. The rules from all attached security groups are aggregated to form a single set of permissions.

</details>
