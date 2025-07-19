# Step 77: Architecture Use‑Case: Scalable Web App

## 🎯 Objective
- [x] Master: **Architecture Use‑Case: Scalable Web App**

## 📘 Notes

### **Architectural Deep Dive: Scalable Web App**

The goal of this architecture is to build a web application that is **highly available**, **scalable**, **resilient**, and **performant**. It follows the principles of the Well-Architected Framework by eliminating single points of failure, decoupling components, and designing for elasticity. This is the standard "multi-tier" architecture on AWS.

---

### **The Architectural Layers**

Let's break down the architecture from the user to the database.

1. **DNS (The Address Book):**
    - **Service:** **Amazon Route 53**
    - **Function:** Resolves the user-friendly domain name (e.g., `www.myapp.com`) to an IP address. For this architecture, we use a Route 53 **Alias Record** to point the domain name to our load balancer.
2. **Content Delivery Network (CDN) & Security Edge:**
    - **Service:** **Amazon CloudFront**
    - **Function:** CloudFront serves two key purposes:
        - **Performance:** It caches the website's **static content** (images, CSS, JavaScript) at edge locations around the world, delivering it to users with very low latency.
        - **Security:** It acts as the front door, absorbing DDoS attacks (**AWS Shield Standard** is built-in) and filtering malicious web traffic if integrated with **AWS WAF**.
    - **Dynamic Content:** Requests for dynamic content (e.g., API calls) are not cached but are still routed through CloudFront's optimized network path to the origin.
3. **Load Balancing (The Traffic Cop):**
    - **Service:** **Application Load Balancer (ALB)**
    - **Function:** The ALB is the origin for the CloudFront distribution. It receives all dynamic requests and distributes them across the fleet of web servers. It is deployed across **multiple Availability Zones** for high availability and can terminate SSL/TLS traffic, offloading that work from the web servers.
4. **Compute Layer (The Web Servers):**
    - **Service:** **Auto Scaling Group (ASG)** of **EC2 instances**.
    - **Function:** The ASG manages the fleet of web servers.
        - **Scalability:** It uses dynamic scaling policies (e.g., Target Tracking on CPU utilization) to automatically add or remove instances to match traffic demand.
        - **High Availability:** The ASG is configured to launch instances across **multiple Availability Zones**. If one AZ fails, the ASG will continue to run instances in the healthy AZs.
        - **Self-Healing:** If an instance fails an ELB or EC2 health check, the ASG will automatically terminate it and launch a replacement.
5. **Session State (The Short-Term Memory):**
    - **Principle:** The EC2 instances in the compute layer must be **stateless**. They should not store user session data locally.
    - **Service:** **Amazon ElastiCache** (Redis or Memcached) or **Amazon DynamoDB**.
    - **Function:** User session data (e.g., items in a shopping cart, login status) is offloaded to this external, shared data store. This allows any web server to handle any user's request, which is critical for Auto Scaling.
6. **Data Layer (The Long-Term Memory):**
    - **Service:** **Amazon RDS** in a **Multi-AZ** configuration.
    - **Function:** This is the relational database that stores the application's persistent data (e.g., user profiles, product catalogs).
        - **High Availability:** The **Multi-AZ** deployment ensures the database is fault-tolerant and can survive an AZ failure by automatically failing over to a synchronous standby replica.
        - **Read Scaling (Optional):** If the database is read-heavy, you can add one or more **Read Replicas** to offload read queries.

---

### **SAA-C03 Style Scenario Questions**

**1. A company is deploying a popular web application on AWS and needs to ensure it can handle unpredictable traffic spikes and remain available even if a single EC2 instance fails. Which combination of services provides both scalability and high availability for the web tier?**

- A. A single, large EC2 instance with an Elastic IP address.
- B. An Application Load Balancer and an Auto Scaling Group.
- C. Amazon Route 53 with a Failover routing policy.
- D. AWS Lambda and Amazon S3.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the core of the scalable web architecture. The Application Load Balancer distributes traffic, and the Auto Scaling Group adds/removes instances (scalability) and replaces failed instances (high availability).
</details>
    

---

**2. In a multi-tier web application, where is the most appropriate place to store user session data, such as items in a shopping cart, to ensure the web tier remains stateless?**

- A. In a cookie on the user's browser.
- B. On the local EBS volume of each EC2 web server.
- C. In a shared, in-memory data store like Amazon ElastiCache for Redis.
- D. In an S3 bucket.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Storing session data on the EC2 instances (B) would make them stateful and prevent effective auto-scaling. The best practice is to externalize this state to a fast, shared data store. ElastiCache for Redis is purpose-built for low-latency session storage. DynamoDB is also a valid option.
</details>
    

---

**3. A website serves a mix of dynamic API content from an ALB and static assets (images, CSS). Users around the world are complaining about slow load times for the images. What is the most effective way to improve performance for the static assets?**

- A. Increase the size of the EC2 instances in the Auto Scaling Group.
- B. Create an RDS Read Replica in each region.
- C. Use Amazon CloudFront to cache the static assets at edge locations.
- D. Use S3 Transfer Acceleration.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The best way to reduce latency for global users accessing static content is to use a Content Delivery Network (CDN). Amazon CloudFront will cache the images and CSS files at edge locations physically closer to the users, resulting in much faster download times.
</details>
    

---

**4. The database for a critical application must be able to withstand the failure of an entire Availability Zone with minimal downtime and no data loss. Which RDS configuration should be used?**

- A. A Single-AZ deployment with frequent snapshots.
- B. A Multi-AZ deployment.
- C. A Read Replica in a different AZ.
- D. A larger instance class.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** RDS Multi-AZ is the feature designed for high availability and fault tolerance. It creates a synchronous, hot standby in a different AZ and will automatically fail over in the event of an AZ outage.
</details>
    

---

**5. Which component of the scalable web architecture is responsible for resolving the application's domain name to the load balancer's endpoint?**

- A. The Auto Scaling Group
- B. Amazon CloudFront
- C. The VPC's internal DNS resolver.
- D. Amazon Route 53
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** Amazon Route 53 is the DNS service. You would create a record set (like an Alias record) in your hosted zone to point your custom domain name to the DNS name of the CloudFront distribution or the Application Load Balancer.
</details>
    

---

### **Key Exam Tips & Tricks**

- **This is THE Pattern:** The multi-tier, multi-AZ, scalable web architecture is the single most important pattern to know for the SAA-C03 exam. A significant percentage of questions will be a variation of this architecture. Burn it into your memory.
- **Stateless is a Prerequisite:** Remember that for horizontal scaling to work, the compute layer (EC2 instances) **must be stateless**. This means any question about scaling a web tier often has a related question about where to put the state (ElastiCache, DynamoDB).
- **Map the Problem to the Layer:** When you read a scenario question, identify which layer of the architecture is being tested.
    - "Global users complain about slowness" -> Think **CloudFront** (CDN Layer).
    - "Need to distribute traffic across instances" -> Think **ELB** (Load Balancing Layer).
    - "Need to automatically add/remove servers" -> Think **ASG** (Compute Layer).
    - "Need to store shopping cart data" -> Think **ElastiCache/DynamoDB** (Session State Layer).
    - "Database must be highly available" -> Think **RDS Multi-AZ** (Data Layer).
- **Differentiate HA and Scaling:**
    - **High Availability (HA):** Using multiple AZs. (ELB across AZs, ASG across AZs, RDS Multi-AZ).
    - **Scalability:** Adding/removing resources based on load. (ASG scaling policies).
    - **Read Scaling:** A separate concept for the database layer. (RDS Read Replicas). A question about a slow database due to reporting queries is solved with a Read Replica, not by making the web tier bigger.
- **Decoupling is Key:** This architecture is inherently decoupled at several layers. The ELB decouples users from specific instances. An optional SQS queue could further decouple the web tier from a backend processing tier. Always look for solutions that reduce interdependencies.
