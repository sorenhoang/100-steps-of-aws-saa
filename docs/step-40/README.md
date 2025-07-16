# Step 40: Route 53 Routing Policies & DNS TTL

## 🎯 Objective

- [x] Master: **Route 53 Routing Policies & DNS TTL**

## 📘 Notes

### **Deep Dive: Route 53 Routing Policies & DNS TTL**

Amazon Route 53 translates human-friendly domain names (like `www.amazon.com`) into the IP addresses that computers use to connect to each other. Beyond simple translation, Route 53 provides a set of powerful routing policies that let you control how it responds to DNS queries, enabling a wide range of architectures for high availability, performance, and traffic management.

---

### **1. DNS Time to Live (TTL)** ⏰

**Time to Live (TTL)** is a value, in seconds, that tells DNS resolvers (like your ISP's DNS server or your own computer) how long they should cache a DNS record.

- **How it works:** When a resolver gets a response from Route 53, it stores that IP address in its cache for the duration of the TTL. Any subsequent requests for the same domain from users behind that resolver will be answered from the cache, not by asking Route 53 again.
- **Impact:**
  - **Long TTL (e.g., 86400 seconds / 24 hours):** Reduces the number of queries to Route 53, which can lower costs and make responses faster for repeat lookups. However, it means that if you change the IP address (e.g., during a failover), it can take up to 24 hours for the change to propagate globally as caches expire.
  - **Short TTL (e.g., 60 seconds):** Increases the number of queries to Route 53 but allows for much faster changes and failovers, as resolvers will ask for the latest record more frequently. This is common for records used in failover configurations.

---

### **2. Route 53 Routing Policies**

This is how you configure Route 53 to respond to DNS queries.

- **Simple Routing:**
  - **Use Case:** The most basic form. You have one record that maps to one or more IP addresses (e.g., your website's server).
  - **How it works:** If you provide multiple IP addresses, Route 53 returns all of them to the client in a random order. It does **not** perform health checks. If one of your servers is down, Route 53 will still hand out its IP address.
- **Failover Routing:**
  - **Use Case:** Designing an active-passive disaster recovery setup.
  - **How it works:** You configure a **primary** record and a **secondary** record. Route 53 monitors the health of the primary endpoint using **Route 53 health checks**. If the primary endpoint becomes unhealthy, Route 53 will stop responding with the primary record and start responding with the secondary record instead.
- **Geolocation Routing:**
  - **Use Case:** Restricting content distribution to specific geographic locations (geoblocking) or localizing content (e.g., sending users in France to a French-language version of your site).
  - **How it works:** You create records for specific geographic locations (e.g., continent, country, or state in the US). Route 53 identifies the geographic location of the user's DNS resolver and responds with the record for the most specific location match. You must also create a "Default" record for users who don't match any specific location.
- **Geoproximity Routing:**
  - **Use Case:** Shifting traffic from resources in one location to resources in another based on a defined "bias."
  - **How it works:** This is a more advanced traffic flow model (often used with Route 53 Flow). You can define a "bias" to expand or shrink the size of the geographic region from which traffic is routed to a resource. For example, you can increase the bias for a region to send more traffic there, even from users who are geographically closer to another resource.
- **Latency-Based Routing:**
  - **Use Case:** Improving global application performance by routing users to the AWS region that provides the **lowest network latency**.
  - **How it works:** You create records for your resources in multiple AWS regions. Route 53 has a database of latency measurements between different parts of the internet and AWS regions. When a query is received, it responds with the record for the region that will give that user the fastest response time.
- **Multivalue Answer Routing:**
  - **Use Case:** A simple, client-side approach to load balancing and high availability that is similar to Simple Routing but with the addition of **health checks**.
  - **How it works:** You can configure Route 53 to return up to eight healthy records in response to a DNS query. Route 53 will perform health checks on each endpoint and will only return the IP addresses of the healthy ones. The client then randomly chooses one of the returned IPs.
- **Weighted Routing:**
  - **Use Case:** Distributing traffic across multiple resources in proportions that you specify. Common for A/B testing or blue-green deployments.
  - **How it works:** You assign a "weight" (a number from 0 to 255) to each record. Route 53 sends traffic to a record based on its weight relative to the sum of all weights. For example, if Record A has a weight of 80 and Record B has a weight of 20, 80% of traffic will be sent to A and 20% will be sent to B.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company wants to run a global web application hosted on EC2 instances in both the `us-east-1` and `eu-west-1` regions. They want to ensure that users are automatically directed to the region that will provide them with the lowest network latency. Which Route 53 routing policy should be used?**

- A. Geolocation Routing
- B. Failover Routing
- C. Latency-Based Routing
- D. Weighted Routing

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Latency-Based Routing is specifically designed for this purpose. Route 53 maintains a database of network latency from different parts of the world to each AWS Region. It uses this data to respond to DNS queries with the IP address of the resource in the region that will provide the fastest response time for that specific user.

</details>

---

**2. A company is performing a blue-green deployment for a new version of their website. They want to send 10% of their traffic to the new version (the "blue" environment) and keep 90% of the traffic on the current version (the "green" environment) to test for issues. Which routing policy allows for this traffic distribution?**

- A. Simple Routing
- B. Failover Routing
- C. Multivalue Answer Routing
- D. Weighted Routing

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Weighted Routing is the perfect tool for this. You can create two record sets for the same domain name: one for the green environment with a weight of 90, and one for the blue environment with a weight of 10. Route 53 will then distribute the traffic according to this 90/10 split.

</details>

---

**3. What is the effect of setting a very low Time to Live (TTL) value (e.g., 60 seconds) for a DNS record?**

- A. It reduces the number of queries to Route 53, lowering costs.
- B. It allows DNS changes to propagate more quickly because resolvers must re-query for the record more often.
- C. It improves the security of the DNS record by encrypting it.
- D. It ensures that DNS queries are always served from a cache.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A low TTL means that DNS resolvers are not allowed to cache the record for very long. This forces them to frequently ask Route 53 for the latest version of the record. While this increases queries, it is essential for scenarios like a DNS failover, where you need clients to see the updated IP address as quickly as possible.

</details>

---

**4. A company needs to configure an active-passive disaster recovery setup for their primary website in `us-east-1` and a standby site in `us-west-2`. If the primary site becomes unavailable, traffic should automatically be redirected to the standby site. Which routing policy should be used?**

- A. Latency-Based Routing
- B. Failover Routing
- C. Simple Routing
- D. Geolocation Routing

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the classic use case for Failover Routing. You configure a primary record and a secondary record. Route 53 uses health checks to monitor the primary endpoint. If the health check fails, Route 53 automatically stops responding with the primary record and starts responding with the secondary record.

</details>

---

**5. A media company needs to serve different versions of its website to users in Germany and Spain, with a default English version for all other users. Which routing policy can achieve this content localization?**

- A. Simple Routing
- B. Failover Routing
- C. Latency-Based Routing
- D. Geolocation Routing

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Geolocation Routing is designed to route traffic based on the geographic location of the user's DNS query. You can create specific records for Germany and Spain pointing to the localized content, and a "Default" record pointing to the English version for everyone else.

</details>

---

**6. Which Route 53 routing policy is similar to Simple Routing but has the added benefit of integrating with health checks to return only the IP addresses of healthy endpoints?**

- A. Weighted Routing
- B. Failover Routing
- C. Multivalue Answer Routing
- D. Geolocation Routing

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Multivalue Answer Routing improves upon Simple Routing by adding health checks. You can have up to eight records, and Route 53 will only return the values for the ones that are currently passing their health checks, providing a simple, client-side form of load balancing and fault tolerance.

</details>

---

**7. What is required for a Failover Routing policy to work?**

- A. The TTL must be set to 0.
- B. The primary and secondary resources must be in the same AWS Region.
- C. A Route 53 health check must be configured for the primary endpoint.
- D. Both endpoints must be EC2 instances.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The failover mechanism is triggered by the status of a Route 53 health check. Route 53 continuously monitors the primary endpoint. The moment the health check fails, Route 53 initiates the DNS failover to the secondary record. Without a health check, Route 53 has no way of knowing the primary endpoint is down.

</details>

---

**8. Which routing policy is NOT suitable if you need to associate a health check with the record?**

- A. Failover Routing
- B. Simple Routing
- C. Multivalue Answer Routing
- D. Weighted Routing

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Simple Routing is the most basic policy and does not support health checks. If an endpoint associated with a simple record becomes unavailable, Route 53 will continue to return its IP address, potentially sending users to a dead server. All the other listed policies can use health checks.

</details>

---

**9. When using Geolocation routing, what happens if a user's query comes from a location for which you have not configured a specific record?**

- A. The user receives a DNS error.
- B. The query is routed based on latency instead.
- C. The query is routed to the endpoint specified in the "Default" record.
- D. Route 53 does not respond to the query.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A "Default" record is mandatory when using Geolocation routing. It acts as a catch-all for any user whose location does not match one of your more specific geographic records, ensuring that all users get a valid response.

</details>

---

**10. What is Time to Live (TTL) in the context of DNS?**

- A. The total time a DNS record is allowed to exist before it is automatically deleted.
- B. The time in seconds that a DNS resolver is allowed to cache a DNS record.
- C. The latency, in milliseconds, for a DNS query.
- D. The time it takes for a new DNS record to be created.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** TTL is a setting on a DNS record that tells caching resolvers how long they should hold on to that record's information before they need to ask the authoritative DNS server (like Route 53) for it again.

</details>

---

**11. A company wants to gradually move traffic from an old set of servers to a new set. They plan to start by sending 5% of traffic to the new servers and 95% to the old ones, gradually increasing the percentage for the new servers. Which policy is ideal for this?**

- A. Latency-Based Routing
- B. Geolocation Routing
- C. Weighted Routing
- D. Failover Routing

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Weighted Routing provides precise control over traffic distribution. The administrator can set the weights to 95 and 5, and then over time, gradually adjust the weights (e.g., to 90/10, then 80/20, etc.) to smoothly transition traffic to the new environment.

</details>

---

**12. Which statement about Simple Routing is TRUE?**

- A. It routes traffic based on the geographic location of the user.
- B. It supports health checks to remove unhealthy endpoints.
- C. It can be used for a single resource or multiple resources, but it returns all values to the client in a random order if multiple exist.
- D. It distributes traffic based on a predefined weight.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Simple Routing policy is for a single record with one or more values (e.g., IP addresses). It does not perform health checks and simply returns the list of values to the resolver.

</details>

---

**13. When would you choose Multivalue Answer routing over a Simple Routing policy with multiple IP addresses?**

- A. When you need to route traffic to different AWS regions.
- B. When you want Route 53 to perform health checks and only return the IPs of healthy endpoints.
- C. When you need to specify the percentage of traffic sent to each IP.
- D. When you want the lowest possible DNS query cost.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The key advantage of Multivalue Answer over Simple routing is the integration with health checks. This provides a simple, DNS-based method of fault tolerance by ensuring clients are not given the IP addresses of servers that are known to be down.

</details>

---

**14. A higher TTL value for a DNS record leads to:**

- A. Faster propagation of DNS changes.
- B. Lower latency for initial DNS lookups.
- C. Less traffic to your authoritative DNS servers (e.g., Route 53).
- D. More accurate geolocation routing.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A higher TTL means that resolvers will cache the record for a longer time. This means they won't need to ask Route 53 for the record as often, which reduces the number of DNS queries you serve and are billed for. The downside is that changes take longer to propagate.

</details>

---

**15. A company is using a Failover routing policy. The primary endpoint passes its health check. Which record will Route 53 return to users?**

- A. The secondary record.
- B. Both the primary and secondary records.
- C. The primary record.
- D. Whichever record has lower latency.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** As long as the primary endpoint is healthy, the Failover policy will exclusively return the primary record. It only switches to the secondary record when the primary health check fails.

</details>

---

**16. Which routing policy is most concerned with the network performance between the end-user and an AWS Region?**

- A. Geolocation
- B. Weighted
- C. Latency-Based
- D. Multivalue Answer

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Latency-Based Routing's entire purpose is to minimize network latency. It makes routing decisions based on real-world performance measurements to send users to the region that will respond the fastest.

</details>

---

**17. What is the maximum number of healthy records that a Multivalue Answer query can return?**

- A. 2
- B. 8
- C. 16
- D. There is no limit.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** You can configure up to eight records for a Multivalue Answer routing policy. Route 53 will return the IP addresses for all of them, assuming they are all healthy.

</details>

---

**18. You want to set up a record for `test.example.com` that points to an Application Load Balancer. What type of Route 53 record should you create?**

- A. An A record pointing to the ALB's IP address.
- B. An AAAA record.
- C. An Alias record pointing to the ALB's DNS name.
- D. A PTR record.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The best practice for pointing a domain to an AWS resource like a Load Balancer or CloudFront distribution is to use an Alias record. Alias records are a Route 53-specific feature that acts like a CNAME but can be used at the zone apex (e.g., example.com). They point to the resource's DNS name, and Route 53 automatically resolves this to the correct IP addresses. You should not use an A record (A) because the ALB's IP addresses can change.

</details>

---

**19. You have just updated the IP address for `www.example.com`. The record's TTL is set to 24 hours (86400 seconds). Some users are still being sent to the old IP address. Why?**

- A. The Route 53 health check has not updated yet.
- B. The user's local DNS resolver has cached the old record and will not ask for the new one until the 24-hour TTL expires.
- C. The new IP address is in a different Availability Zone.
- D. You must use a Weighted routing policy for changes to take effect.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This demonstrates the effect of a long TTL. A DNS resolver that looked up the record before the change will have cached the old IP address. It is programmed to not ask Route 53 for an update until the TTL of 86400 seconds has passed. This is why low TTLs are critical for resources that might change.

</details>

---

**20. Geoproximity routing is different from Geolocation routing because it allows you to:**

- A. Route users based on their country.
- B. Influence the size of the geographic area from which traffic is routed to a resource by using a "bias".
- C. Route users based on network latency.
- D. Create a default record for un-matched locations.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Geoproximity routing is a more advanced, traffic-flow-oriented policy. Its unique feature is the "bias," which allows you to expand or shrink a geographic map. For example, you can apply a positive bias to an endpoint to have it draw in traffic from a larger geographic area, even if those users are physically closer to a different endpoint.

</details>

---

**21. A company wants to use Route 53 to perform simple client-side load balancing for a set of 4 web servers. They also need to ensure that if a server goes down, its IP is no longer returned by DNS queries. Which routing policy is the best fit?**

- A. Simple Routing
- B. Weighted Routing with equal weights.
- C. Failover Routing
- D. Multivalue Answer Routing

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Multivalue Answer Routing is designed for exactly this. It allows you to register multiple IP addresses for the same record and, crucially, associates health checks with each one. It will return the set of healthy IPs, and the client (like a web browser) will typically pick one at random, providing simple load balancing and fault tolerance.

</details>

---

**22. Which two routing policies would you consider for an active-active multi-region architecture? (Select TWO)**

- A. Simple Routing
- B. Weighted Routing
- C. Latency-Based Routing
- D. Failover Routing
- E. Geolocation Routing

<details>
<summary>View Answer</summary>

**Answers: B and C**

**Explanation:** In an active-active setup, both regions are serving live traffic. You could use Latency-Based Routing (C) to send users to the region with the best performance for them. Alternatively, you could use Weighted Routing (B) to distribute traffic between the regions in a specific ratio (e.g., 50/50). Failover Routing (D) is for active-passive setups.

</details>
