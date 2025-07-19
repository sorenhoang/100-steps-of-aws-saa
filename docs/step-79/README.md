# Step 79: Architecture Use‑Case: Multi‑Tier App

## 🎯 Objective

- [x] Master: **Architecture Use‑Case: Multi‑Tier App**

## 📘 Notes

### **Architectural Deep Dive: Multi-Tier App**

The multi-tier (often 3-tier) architecture is a standard client-server application model that separates an application into logical and physical tiers. This separation of concerns improves scalability, security, and manageability. On AWS, this pattern is built using a combination of VPC networking and compute services to create a secure and resilient environment.

---

### **The Three Tiers**

1. **Web Tier (Presentation Layer):**
    - **Function:** This is the public-facing layer that handles incoming HTTP/HTTPS requests from users. It's responsible for the user interface and presentation logic.
    - **AWS Services:**
        - **Elastic Load Balancer (ELB):** Typically an Application Load Balancer (ALB) to distribute incoming traffic.
        - **Auto Scaling Group (ASG) of EC2 instances:** Runs the web server software (e.g., Nginx, Apache).
    - **Network Placement:** Resides in **public subnets** across multiple Availability Zones to be accessible from the internet.
    - **Security:** The security group for this tier allows inbound traffic on port 80 (HTTP) and 443 (HTTPS) from anywhere (`0.0.0.0/0`).
2. **Application Tier (Logic Layer):**
    - **Function:** This tier contains the core business logic of the application. It processes the requests received from the web tier, communicates with the database, and performs the main application functions.
    - **AWS Services:**
        - Often, an **internal ELB** to distribute traffic from the web tier.
        - An **Auto Scaling Group (ASG) of EC2 instances** running the application code (e.g., Node.js, Java, Python).
    - **Network Placement:** Resides in **private subnets** across multiple Availability Zones. It should not be directly accessible from the internet.
    - **Security:** The security group for this tier is a critical control point. It should **only** allow inbound traffic from the **security group of the web tier**. This prevents anyone from bypassing the web servers and accessing your application logic directly.
3. **Data Tier (Database Layer):**
    - **Function:** This tier is responsible for storing and managing the application's data.
    - **AWS Services:** Typically a managed database service like **Amazon RDS** or **Aurora**, configured in a **Multi-AZ** deployment for high availability.
    - **Network Placement:** Resides in **private subnets** (often a separate set of private subnets for maximum isolation) across multiple Availability Zones.
    - **Security:** The security group for this tier is the most restrictive. It should **only** allow inbound traffic (e.g., on port 3306 for MySQL) from the **security group of the application tier**.

---

### **SAA-C03 Style Scenario Questions**

**1. A solutions architect is designing a 3-tier web application. For security reasons, the database servers must not be directly accessible from the internet. In which VPC layer should the database instances be placed?**

- A. In a public subnet, but with a restrictive security group.
- B. In a private subnet.
- C. In a different VPC connected via VPC Peering.
- D. In the same public subnet as the web servers.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The data tier contains the most sensitive assets and should have the highest level of protection. Placing the database servers in a private subnet ensures they do not have a direct route to the internet and cannot be accessed from the outside world.
</details>
    

---

**2. How should the security group for the application tier be configured to ensure it only accepts traffic from the web tier?**

- A. Allow inbound traffic from the IP address range of the public subnet.
- B. Allow inbound traffic from the CIDR block of the VPC.
- C. Allow inbound traffic where the source is the security group ID of the web tier.
- D. Allow all inbound traffic (`0.0.0.0/0`).
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a critical security best practice for multi-tier architectures. By setting the source of the inbound rule to the security group ID of the web tier, you create a dynamic and secure rule that allows only the web server instances to communicate with the application server instances. This is more secure and flexible than using IP ranges.
</details>
    

---

**3. The application tier needs to download software updates from the internet, but it resides in a private subnet. What component is required to enable this outbound-only internet access?**

- A. An Internet Gateway attached to the private subnet.
- B. An Elastic IP address on each application server.
- C. A NAT Gateway located in a public subnet, and a route from the private subnet to it.
- D. A VPC Peering connection.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The standard pattern for providing outbound internet access to resources in a private subnet is to use a NAT Gateway. The private subnet's route table is configured to send internet-bound traffic (0.0.0.0/0) to the NAT Gateway, which then forwards the traffic to the internet via an Internet Gateway.
</details>
    

---

**4. What is the primary benefit of separating an application into multiple tiers?**

- A. It decreases the overall cost of the application.
- B. It allows each tier to be scaled and managed independently, improving scalability and security.
- C. It reduces network latency between the components.
- D. It simplifies the deployment process.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The separation of concerns is the key benefit. You can scale the web tier and the application tier independently based on their specific resource needs. It also creates strong security boundaries; a compromise of the public-facing web tier does not automatically mean the more secure data tier is compromised.
</details>
    

---

**5. In a highly available 3-tier architecture, where should the EC2 instances for the web and application tiers be deployed?**

- A. In a single, large subnet for simplicity.
- B. Across multiple Availability Zones within a single region.
- C. Across multiple AWS Regions.
- D. In a placement group for low latency.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To achieve high availability, you must design your system to withstand the failure of an entire data center. This is done by deploying your resources (like the EC2 instances in your Auto Scaling Groups) across multiple Availability Zones. An Elastic Load Balancer will then route traffic only to the instances in the healthy AZs.
</details>
    

---

### **Key Exam Tips & Tricks**

- **Security Group Referencing is Key:** This is one of the most frequently tested concepts related to this architecture. You MUST know that the best way to control traffic between tiers is to have the security group for the inner tier (e.g., the App Tier) reference the security group ID of the outer tier (e.g., the Web Tier) as its source. This is more secure and scalable than using IP addresses or CIDR ranges.
- **Public vs. Private Subnets:** Understand the role of each.
    - **Public Subnet:** For anything that needs to be *directly reached* from the internet (e.g., web servers, public-facing ELBs, bastion hosts) or needs to *directly reach* the internet (NAT Gateways). Requires a route to an Internet Gateway.
    - **Private Subnet:** For backend components that should be protected from the internet (e.g., application servers, database servers). Has no route to an Internet Gateway.
- **Traffic Flow:** Be able to trace a request through the entire architecture.
    - `User -> Route 53 -> ELB (Public Subnet) -> Web Tier EC2 (Public Subnet) -> App Tier EC2 (Private Subnet) -> RDS Database (Private Subnet)`.
    - Remember that communication between the tiers (e.g., Web to App, App to DB) happens over their **private IP addresses** using the VPC's internal router.
- **Think in Tiers:** When you read a question, identify which tier it's talking about.
    - "Users can't access the website" -> Problem is likely at the edge (Route 53, ELB, Web Tier SG).
    - "Web servers can't talk to the application servers" -> Problem is likely the App Tier SG.
    - "Application servers can't talk to the database" -> Problem is likely the Data Tier SG.
    - "Application servers can't download patches" -> Problem is likely a missing NAT Gateway or route.