# Step 19: Load Balancers – CLB vs ALB vs NLB

## 🎯 Objective

- [x] Master: **Load Balancers – CLB vs ALB vs NLB**

## 📘 Notes

### **Deep Dive: Load Balancers – CLB vs. ALB vs. NLB**

Elastic Load Balancing (ELB) automatically distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses. It increases the fault tolerance and availability of your application. AWS provides three main types of load balancers, each operating at a different layer of the OSI model and designed for different use cases.

---

### **1. Classic Load Balancer (CLB) - *Legacy*** 👴

The Classic Load Balancer is the previous generation of load balancer. It can operate at both **Layer 4 (TCP)** and **Layer 7 (HTTP/HTTPS)**, but with limited functionality compared to the newer types.

- **Key Features:**
    - Provides basic load balancing across multiple EC2 instances.
    - Supports health checks.
    - **Legacy:** You should know what it is for the exam, but it is **not recommended for new applications**. You should always choose an Application or Network Load Balancer instead.
- **Use Case:** Primarily for applications built within the EC2-Classic network (an older, legacy network that is being phased out).

---

### **2. Application Load Balancer (ALB) - *Layer 7*** 🌐

The Application Load Balancer is the best choice for load balancing HTTP and HTTPS traffic. It operates at the **Application Layer (Layer 7)** of the OSI model, which means it is "application-aware" and can make intelligent routing decisions based on the content of the request.

- **Key Features:**
    - **Content-Based Routing:** This is its most powerful feature. An ALB can inspect the request and route it to different target groups based on:
        - **Path-based routing:** `example.com/users` goes to one target group, while `example.com/orders` goes to another.
        - **Host-based routing:** `users.example.com` goes to one group, while `orders.example.com` goes to another.
        - HTTP headers, query string parameters, and source IP addresses.
    - **Target Groups:** An ALB routes traffic to *target groups*. A target group can contain EC2 instances, IP addresses (for on-premises servers), or Lambda functions.
    - **Protocols:** HTTP, HTTPS, and gRPC.
    - **Other Features:** Supports sticky sessions, redirects, fixed-response actions, and integrates with AWS WAF for web application security. It also supports multiple SSL certificates using Server Name Indication (SNI).
- **Use Case:** Perfect for modern microservices-based architectures, containerized applications (ECS/EKS), and any standard web application that needs flexible routing rules.

---

### **3. Network Load Balancer (NLB) - *Layer 4*** ⚡

The Network Load Balancer is designed for extreme performance and operates at the **Transport Layer (Layer 4)**. It can handle millions of requests per second with ultra-low latency.

- **Key Features:**
    - **Extreme Performance:** It is optimized for handling volatile traffic patterns and high-throughput TCP/UDP traffic.
    - **Static IP Address:** An NLB can have a **static Elastic IP address** assigned to it per Availability Zone. This is a key differentiator and is useful when clients or firewalls need to whitelist a fixed IP address.
    - **Source IP Preservation:** It preserves the client-side source IP address, which is passed through to your instances. ALBs, by contrast, typically replace the source IP with their own.
    - **Protocols:** TCP, UDP, and TLS. It does not understand HTTP headers or content. It simply forwards the connection.
- **Use Case:** Ideal for TCP/UDP traffic where extreme performance is required. Common examples include online gaming, IoT protocols, and high-performance API endpoints that don't require HTTP-level inspection. Also used when a static IP address is a requirement for the load balancer endpoint.

---

### **Summary of Key Differences**

| Feature | Application Load Balancer (ALB) | Network Load Balancer (NLB) | Classic Load Balancer (CLB) |
| --- | --- | --- | --- |
| **OSI Layer** | **Layer 7** (Application) | **Layer 4** (Transport) | Layer 4/7 |
| **Protocols** | HTTP, HTTPS, gRPC | **TCP, UDP, TLS** | TCP, SSL/TLS, HTTP, HTTPS |
| **Routing** | **Content-based** (Path, Host, etc.) | Forwards connection directly | Basic round-robin |
| **Static IP** | No | **Yes (one per AZ)** | No |
| **Use Case** | Web applications, microservices, containers | **Extreme performance, gaming, static IPs** | Legacy EC2-Classic apps |
| **IP Preservation** | No (uses X-Forwarded-For header) | **Yes** (preserves source IP) | No (uses X-Forwarded-For header) |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company is building a microservices-based application. They need a load balancer that can route traffic to different services based on the URL path. For example, requests to `/api/users` should go to the user service, and requests to `/api/orders` should go to the order service. Which load balancer should they choose?**

- A. Classic Load Balancer
- B. Network Load Balancer
- C. Application Load Balancer
- D. AWS Global Accelerator

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is the primary use case for an Application Load Balancer (ALB). Because it operates at Layer 7 (the application layer), it can inspect the content of the request, including the URL path. This allows for advanced, content-based routing rules, making it perfect for microservices architectures.
</details>
    

---

**2. A high-performance online gaming application uses a custom UDP-based protocol for communication between clients and servers. The company needs a load balancer that can handle millions of requests per second with ultra-low latency. Which load balancer is the most appropriate choice?**

- A. Classic Load Balancer
- B. Application Load Balancer
- C. Network Load Balancer
- D. An Auto Scaling group without a load balancer.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A Network Load Balancer (NLB) is the correct choice for high-throughput, low-latency UDP traffic. It operates at Layer 4 and is optimized for extreme performance. An Application Load Balancer (B) does not support the UDP protocol.
</details>
    

---

**3. A company has a legacy application running on EC2 that requires its clients to whitelist a single, unchanging public IP address for the load balancer. Which load balancer type can provide a static Elastic IP address?**

- A. Application Load Balancer
- B. Classic Load Balancer
- C. Network Load Balancer
- D. None of the above.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A key feature and major differentiator of the Network Load Balancer (NLB) is its ability to have a static Elastic IP address assigned to it in each Availability Zone it covers. This provides a predictable, fixed IP address that is essential for scenarios involving firewall rules or legacy clients.
</details>
    

---

**4. What is a key feature of an Application Load Balancer (ALB) that is not available on a Network Load Balancer (NLB)?**

- A. The ability to handle TCP traffic.
- B. The ability to route traffic based on the HTTP host header.
- C. The ability to be placed in multiple Availability Zones.
- D. The ability to use health checks.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Content-based routing is exclusive to the Application Load Balancer. An ALB can inspect Layer 7 data like HTTP host headers, paths, and query strings to make intelligent routing decisions. An NLB operates at Layer 4 and simply forwards packets based on IP and port, without inspecting the application content.
</details>
    

---

**5. At which layer of the OSI model does a Network Load Balancer operate?**

- A. Layer 3 (Network)
- B. Layer 4 (Transport)
- C. Layer 5 (Session)
- D. Layer 7 (Application)

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Network Load Balancer operates at Layer 4 (Transport). This means it works with TCP and UDP protocols and makes decisions based on information like IP addresses and port numbers.
</details>
    

---

**6. An Application Load Balancer is placed in front of a fleet of web servers. The servers need to know the original IP address of the client making the request for logging and analytics. How can the web servers get this information?**

- A. The source IP of the incoming packet will be the client's original IP address.
- B. The ALB automatically adds the client's IP to the `X-Forwarded-For` HTTP header.
- C. This is not possible; the client's IP is lost when using an ALB.
- D. By querying the Network Load Balancer that sits in front of the ALB.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** An ALB terminates the client's connection and initiates a new one to the backend target. In this process, the source IP of the packet arriving at the web server is the IP of the ALB itself. To preserve the original client IP, the ALB adds it to the X-Forwarded-For header in the HTTP request. The application must be configured to read this header to get the client's IP.
</details>
    

---

**7. Which load balancer is considered a legacy product and is no longer recommended for new application deployments?**

- A. Application Load Balancer
- B. Network Load Balancer
- C. Gateway Load Balancer
- D. Classic Load Balancer

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** The Classic Load Balancer (CLB) is the first-generation load balancer. While still supported for existing applications, AWS strongly recommends using Application Load Balancers or Network Load Balancers for all new architectures due to their enhanced features, performance, and flexibility.
</details>
    

---

**8. An Application Load Balancer needs to serve traffic for `https://api.example.com` and `https://www.example.com` from the same listener on port 443. How can this be achieved?**

- A. This is not possible; a separate ALB is needed for each domain.
- B. Use multiple SSL certificates on the ALB and Server Name Indication (SNI).
- C. Use a Network Load Balancer instead.
- D. Configure two separate listeners, one for each domain.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** An Application Load Balancer supports Server Name Indication (SNI), which allows you to associate multiple SSL certificates with a single secure listener. The ALB will automatically inspect the hostname requested by the client and present the correct certificate, allowing you to host multiple secure sites behind a single ALB listener.
</details>
    

---

**9. A company wants to load balance traffic to a fleet of EC2 instances and also to an on-premises server connected via AWS Direct Connect. Which load balancer target type would allow this?**

- A. You can only load balance to EC2 instances.
- B. A target group that uses IP addresses as targets.
- C. A target group that uses Lambda functions as targets.
- D. A target group that uses instance IDs as targets.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Both Application Load Balancers and Network Load Balancers support using IP addresses as targets. This powerful feature allows you to register any IP address, including those of servers running in your on-premises data center, as targets in a target group, effectively extending the load balancer's reach beyond the VPC.
</details>
    

---

**10. Which load balancer natively preserves the client's source IP address without needing to use a proxy protocol or special HTTP headers?**

- A. Application Load Balancer
- B. Classic Load Balancer
- C. Network Load Balancer
- D. All load balancers preserve the source IP address.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Because the Network Load Balancer operates at Layer 4 and forwards the connection, the original source IP address of the client is preserved in the TCP/UDP packet header that arrives at the backend target instance. This simplifies network analysis and source IP-based filtering.
</details>
    

---

**11. A web application uses WebSockets for real-time communication. Which load balancer type provides full support for the WebSocket protocol?**

- A. Both Application Load Balancers and Network Load Balancers.
- B. Only Network Load Balancers.
- C. Only Classic Load Balancers.
- D. WebSockets are not supported by any ELB.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** Both the Application Load Balancer and the Network Load Balancer provide support for WebSockets. The ALB understands WebSockets as an upgrade to an HTTP connection, while the NLB supports them as a long-lived TCP connection.
</details>
    

---

**12. An Application Load Balancer is configured with a feature that directs all requests from a specific user to the same target instance for the duration of their session. What is this feature called?**

- A. Session Draining
- B. Sticky Sessions (Session Affinity)
- C. Cross-Zone Load Balancing
- D. Health Checks

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Sticky Sessions, also known as session affinity, is a feature that uses a cookie to ensure that subsequent requests from the same client are sent to the same backend target. This is useful for applications that store user session information locally on the instance.
</details>
    

---

**13. A solutions architect is choosing between an ALB and an NLB. The primary requirement is to route traffic based on a query string parameter in the URL (e.g., `?productID=123`). Which load balancer must be used?**

- A. Network Load Balancer, as it's faster.
- B. Either one can be used.
- C. Application Load Balancer, as it operates at Layer 7.
- D. Classic Load Balancer is the only one with this feature.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Routing based on a query string parameter is a form of content-based routing. This requires the load balancer to inspect the application-level data (Layer 7). Only the Application Load Balancer has this capability.
</details>
    

---

**14. What is the function of a "target group" in the context of an Application Load Balancer?**

- A. It defines the rules for routing traffic, such as path-based routing.
- B. It is a logical grouping of registered targets (like EC2 instances) to which the ALB routes traffic.
- C. It specifies the listener port and protocol for the load balancer.
- D. It defines the health check settings for the load balancer itself.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A target group is simply a collection of destinations for traffic. You register your EC2 instances, IP addresses, or Lambda functions into a target group. The ALB then sends traffic to the healthy targets within that group. A single ALB can have multiple listener rules that route to different target groups.
</details>
    

---

**15. By default, is cross-zone load balancing enabled for Application Load Balancers and Network Load Balancers?**

- A. It is enabled by default for both.
- B. It is disabled by default for both.
- C. It is enabled by default for ALBs but disabled by default for NLBs.
- D. It is enabled by default for NLBs but disabled by default for ALBs.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a key operational difference. For an Application Load Balancer, cross-zone load balancing is enabled by default, meaning it distributes traffic evenly across all registered instances in all enabled Availability Zones. For a Network Load Balancer, it is disabled by default, meaning it only distributes traffic to targets within its own AZ. Enabling it for an NLB can incur inter-AZ data transfer costs.
</details>
    

---

**16. An Application Load Balancer listener rule has a "fixed-response" action configured. What does this do?**

- A. It forwards the request to a fixed, single EC2 instance.
- B. It responds to the client with a static HTTP response code and an optional message, without forwarding the request.
- C. It redirects the client to a different URL.
- D. It forwards the request to a Lambda function.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A fixed-response action is a useful feature where the ALB itself can generate a response. This is often used to respond to requests that don't match any other rule with a custom 404 Not Found page or to implement a maintenance page by responding with a 503 Service Unavailable code without needing any backend infrastructure.
</details>
    

---

**17. Which load balancer would you choose for a simple web application if your only goal is to distribute HTTP traffic evenly across a few EC2 instances with minimal configuration?**

- A. Network Load Balancer
- B. Classic Load Balancer
- C. Application Load Balancer
- D. AWS Global Accelerator

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Even for simple web applications, the Application Load Balancer is the modern standard. It provides more features, better performance, and more security options than the legacy Classic Load Balancer. While a CLB would work, an ALB is the recommended best practice for any new HTTP/HTTPS workload.
</details>
    

---

**18. Which components of an ALB can be targets in a target group? (Select THREE)**

- A. EC2 Instances
- B. Security Groups
- C. IP Addresses
- D. Lambda Functions
- E. S3 Buckets

<details>
<summary>View Answer</summary>
<br>

**Answers: A, C, and D**

**Explanation:** An Application Load Balancer is highly flexible in its target types. It can route traffic to traditional EC2 Instances, to specific IP Addresses (useful for containers or on-premises servers), and it can directly invoke a Lambda Function to run serverless web applications.
</details>
    

---

**19. A Network Load Balancer is configured with a TCP listener on port 8080. One of the target instances has a firewall that only allows traffic on port 80. What will happen to requests sent to this target?**

- A. The NLB will automatically translate the port from 8080 to 80.
- B. The requests will be blocked by the instance's firewall.
- C. The NLB will report the target as healthy but traffic will fail.
- D. This configuration is not possible.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Network Load Balancer forwards the TCP packet with the destination port intact (port 8080). When this packet arrives at the instance, the instance's own firewall (like Security Groups or an OS firewall) will evaluate it. Since the firewall only allows port 80, the packet for port 8080 will be dropped.
</details>
    

---

**20. You want to secure your web application by using AWS WAF. Which load balancer can be integrated with AWS WAF?**

- A. Network Load Balancer
- B. Classic Load Balancer
- C. Application Load Balancer
- D. All of the above.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS WAF (Web Application Firewall) is a Layer 7 security service that protects against common web exploits like SQL injection and cross-site scripting. It can only be integrated with Layer 7 services, making the Application Load Balancer the correct choice.
</details>
    

---

**21. An ALB sits in front of an Auto Scaling group of web servers. The ALB listener is configured to use HTTPS. How is the connection between the ALB and the backend EC2 instances handled?**

- A. The connection between the ALB and the EC2 instances must also be HTTPS.
- B. The connection between the ALB and the EC2 instances can be HTTP, as the traffic is within your secure VPC.
- C. The ALB encrypts the traffic using the EC2 instance's public key.
- D. The connection is handled by a Network Load Balancer.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is a very common architecture. The ALB terminates the SSL/TLS connection from the client, meaning it decrypts the traffic. It can then initiate a new, unencrypted HTTP connection to the backend instances. This offloads the SSL processing from your EC2 instances and simplifies certificate management. While you can re-encrypt the traffic to the backend, it's very common to use HTTP for this internal leg of the journey.
</details>
    

---

**22. Which load balancer type is best suited for achieving the highest possible network throughput for TCP-based applications?**

- A. Application Load Balancer with a performance-mode enabled.
- B. Classic Load Balancer with TCP listeners.
- C. AWS Global Accelerator.
- D. Network Load Balancer.

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** The Network Load Balancer is architected from the ground up for performance. It bypasses much of the connection processing that an ALB does, allowing it to forward TCP packets at extremely high speeds with minimal latency, making it the clear choice for throughput-intensive TCP workloads.
</details>