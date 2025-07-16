# Step 47: DynamoDB DAX & Performance

## 🎯 Objective

- [x] Master: **DynamoDB DAX & Performance**

## 📘 Notes

### **Deep Dive: DynamoDB DAX & Performance**

While DynamoDB is incredibly fast, offering single-digit millisecond latency, some applications require even faster, microsecond-level performance, especially for read-heavy workloads. **Amazon DynamoDB Accelerator (DAX)** is a fully managed, highly available, in-memory cache for DynamoDB that delivers up to a 10x performance improvement—from milliseconds to microseconds.

---

### **1. What is DynamoDB Accelerator (DAX)?** ⚡

DAX is a write-through caching service that sits in front of your DynamoDB tables. It is designed to be seamless to integrate; you point your application to the DAX cluster endpoint instead of the DynamoDB endpoint, and the DAX client handles the rest.

- **Architecture:** A DAX cluster is composed of one or more nodes.
    - **Primary Node:** Handles all writes to the cluster.
    - **Read Replicas:** The cluster can have up to 10 read replicas to serve a massive volume of read requests.
- **How it Works (The Caching Logic):**
    1. Your application makes a read request (e.g., `GetItem`) to the DAX endpoint.
    2. DAX checks its **Item Cache** to see if it has the requested data.
        - **Cache Hit:** If the data is in the cache, DAX returns it to the application with **microsecond** latency. It never has to contact DynamoDB.
        - **Cache Miss:** If the data is not in the cache, DAX forwards the request to the underlying DynamoDB table (an eventually consistent read). DynamoDB returns the item to DAX, which then stores it in its cache and passes it back to the application. Subsequent requests for the same item will now be a cache hit.
    3. For write requests (`PutItem`, `UpdateItem`, `DeleteItem`), DAX acts as a **write-through** cache. The data is written to the DAX primary node and *synchronously* to the DynamoDB table. The write is only considered successful after the data is durable in both the DAX cache and the DynamoDB table.
- **Key Characteristics:**
    - **API Compatible:** The DAX client is designed to be a drop-in replacement for existing DynamoDB SDK calls. Minimal application code changes are required.
    - **Fully Managed:** AWS handles all the patching, failure detection, and management of the DAX cluster nodes.
    - **In-VPC Service:** A DAX cluster runs within your VPC and is accessed by resources within that VPC.

---

### **2. When to Use DAX (And When Not To)**

DAX provides a massive performance boost, but it isn't necessary for every application.

- **Ideal Use Cases:**
    - **Read-Heavy Applications:** Applications that read the same items over and over again will see the most benefit (e.g., the home page of a popular e-commerce site, a social media timeline, a lookup table for product metadata).
    - **Applications Requiring Microsecond Latency:** Real-time bidding, social gaming, or any application where response time is absolutely critical.
- **Poor Use Cases:**
    - **Write-Heavy Applications:** Since DAX is a write-through cache, it doesn't improve write performance. In fact, it adds a tiny bit of latency. If your application primarily writes data, DAX provides no benefit.
    - **Applications Needing Strongly Consistent Reads:** By default, DAX provides eventually consistent reads. While you can request a strongly consistent read, this forces DAX to bypass the cache and read directly from DynamoDB, negating the performance benefit of the cache.
    - **Applications with Infrequent Reads:** If your application rarely reads the same item twice, the cache hit ratio will be very low, and you'll be paying for a cache that isn't being used effectively.

---

### **AWS SAA-C03 Style Questions & Explanations**


**1. A real-time bidding application requires read latency in the microsecond range for accessing product information stored in a DynamoDB table. The same products are read thousands of times per second. Which service should be used to achieve this performance?**

- A. A DynamoDB Global Secondary Index (GSI).
- B. Provisioning a very high number of Read Capacity Units (RCUs) on the table.
- C. DynamoDB Accelerator (DAX).
- D. An RDS Read Replica.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The key requirement is microsecond latency. Standard DynamoDB provides single-digit millisecond latency. DAX is the purpose-built, in-memory caching service for DynamoDB that reduces this latency to microseconds for frequently accessed items, making it the perfect fit for this read-heavy, low-latency application.
</details>

---


**2. How does DAX handle write operations like `PutItem` or `UpdateItem`?**

- A. It is a write-around cache, meaning writes go directly to DynamoDB, bypassing the DAX cache.
- B. It is a write-through cache, meaning data is written to both the DAX cache and the underlying DynamoDB table synchronously.
- C. It is a write-back cache, meaning data is written to the DAX cache first and then asynchronously to DynamoDB later.
- D. DAX does not support write operations.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** DAX uses a write-through caching strategy. This ensures that the cache is always consistent with the database for any data that is written through it. The write operation is only confirmed as successful after the data has been durably stored in both DAX and DynamoDB.
</details>

---


**3. An application makes a `GetItem` request to a DAX cluster, but the item is not currently in the DAX item cache. What happens next?**

- A. DAX returns a "cache miss" error to the application.
- B. DAX performs an eventually consistent read from the underlying DynamoDB table, caches the result, and returns it to the application.
- C. DAX performs a strongly consistent read from the underlying DynamoDB table.
- D. The application must then make a separate request directly to DynamoDB.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This describes the cache miss workflow. DAX handles the miss transparently. It fetches the data from DynamoDB (using a low-cost, eventually consistent read by default), populates its own cache with the item, and then returns the item to the client. Subsequent requests for that same item will then be a cache hit.
</details>

---


**4. For which of the following workloads would DAX be LEAST effective?**

- A. A popular news website's homepage, which displays the same top 10 articles to all users.
- B. An application that primarily ingests and writes large volumes of streaming data with very few reads.
- C. A product catalog for an e-commerce site where popular items are viewed frequently.
- D. A lookup table for country codes that is read by many different microservices.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** DAX is a read cache. For a write-intensive application, it provides no performance benefit for the write operations. Since it is a write-through cache, it actually adds a small amount of latency to each write. Therefore, it would not be a cost-effective or useful addition to this architecture.
</details>

---


**5. How does an application use a DAX cluster?**

- A. The application continues to point to the DynamoDB endpoint, and DAX intercepts the traffic automatically.
- B. The application code must be updated to use the DAX SDK and point to the DAX cluster's endpoint.
- C. The application must query DAX first, and if it's a miss, query DynamoDB second.
- D. The application connects to DAX using a JDBC driver.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** To use DAX, you must integrate the DAX client into your application. This client is API-compatible with the standard DynamoDB client, so code changes are minimal. The primary change is updating the service endpoint in your configuration to point to the DAX cluster's discovery endpoint instead of the standard DynamoDB endpoint.
</details>

---


**6. Which type of read consistency does a standard `GetItem` request to a DAX cluster provide?**

- A. Strongly consistent.
- B. Eventually consistent.
- C. Transactionally consistent.
- D. The consistency level is configurable per request.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** By default, DAX prioritizes speed. Reads that are served from the cache are eventually consistent because there could be a brief moment where the underlying table was updated directly (bypassing DAX), and the cache hasn't been invalidated yet. Even on a cache miss, DAX performs an eventually consistent read from DynamoDB to populate its cache.
</details>

---


**7. An application requires the absolute latest, most up-to-date version of an item from DynamoDB and cannot tolerate even millisecond-level stale data. How should it interact with a DAX cluster?**

- A. It should perform a standard `GetItem` request, as DAX is always up-to-date.
- B. It should perform a `GetItem` request but set the `ConsistentRead` parameter to `true`.
- C. It should perform a `Scan` operation instead of a `GetItem`.
- D. It should bypass DAX and query the DynamoDB table directly.
<details>
<summary>View Answer</summary>

**Answer:** B or D

**Explanation:** Both B and D are valid ways to get a strongly consistent read.

- **Option B:** The DAX client allows you to specify `ConsistentRead=true`. When this flag is set, DAX will **bypass its cache** and send the request directly to DynamoDB as a strongly consistent read. You get the correct data but lose the microsecond performance benefit.
- **Option D:** For applications with mixed consistency needs, a common pattern is to have two clients: a DAX client for eventually consistent reads and a standard DynamoDB client for strongly consistent reads, and use the appropriate one as needed.
</details>

---


**8. What is the primary benefit of adding read replicas to a DAX cluster?**

- A. It increases the write throughput of the cluster.
- B. It provides cross-region disaster recovery for the cache.
- C. It increases the overall read throughput and availability of the cache cluster.
- D. It makes the DAX cache strongly consistent.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Adding read replicas to a DAX cluster allows it to serve a much higher volume of read requests concurrently. It also improves availability, because if one read node fails, the others can continue to serve cached data.
</details>

---


**9. DAX is a fully managed service that runs inside of what network environment?**

- A. It runs in an AWS-managed service VPC.
- B. It runs inside your own Amazon VPC.
- C. It runs on-premises using AWS Outposts.
- D. It is a global service like S3 or IAM.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A DAX cluster is provisioned within your specified VPC and subnets. Your application resources (like EC2 instances or Lambda functions) must be in the same VPC (or a peered VPC with correct routing) to access the DAX cluster endpoint.
</details>

---


**10. How does DAX improve the performance of DynamoDB queries (`Query` operations)?**

- A. It does not cache `Query` results.
- B. It caches the individual items returned by a query, so subsequent `GetItem` requests for those items will be fast.
- C. It caches the entire result set of a `Query` operation in a "Query Cache."
- D. It sends the `Query` operation to be executed on a read replica.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** In addition to its primary Item Cache, DAX also maintains a Query Cache. When you run a Query or Scan, DAX stores the parameters and the set of results in this cache. If another identical Query or Scan is made while the result is still in the cache, DAX can return the entire result set instantly without hitting DynamoDB.
</details>

---


**11. Is DAX a good solution for improving the performance of an application that performs a large number of DynamoDB `Scan` operations?**

- A. Yes, because the Query Cache will cache the results of the `Scan`.
- B. No, because `Scan` operations are inherently inefficient, and using DAX does not fix the underlying poor access pattern.
- C. No, because DAX does not support the `Scan` operation.
- D. Yes, because DAX makes all DynamoDB operations run faster.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** While DAX can cache the results of a Scan (A), a Scan operation itself is highly inefficient as it reads every item in the table. Using a cache to speed up a bad practice is an anti-pattern. The correct solution is to fix the access pattern by using a more appropriate primary key or a Global Secondary Index (GSI) so that a Query can be used instead of a Scan.
</details>

---


**12. What is the relationship between DAX and DynamoDB capacity modes?**

- A. DAX can only be used with tables in Provisioned Throughput mode.
- B. DAX can only be used with tables in On-Demand mode.
- C. DAX can be used with tables in either Provisioned Throughput or On-Demand mode.
- D. Using DAX removes the need to select a capacity mode for the table.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** DAX works as a caching layer regardless of the underlying table's capacity mode. It is particularly effective at reducing the number of RCUs consumed from a Provisioned Throughput table or reducing the per-request read costs for an On-Demand table.
</details>

---


**13. A DAX cluster has a Time to Live (TTL) setting for its cache. What does this define?**

- A. The time after which an item in the cache is considered stale and will be evicted.
- B. The time after which the entire DAX cluster will be automatically deleted.
- C. The TTL for the underlying DynamoDB table.
- D. The time it takes for a write to be propagated to the cache.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** The DAX cache TTL is the "time-to-live" for the items it stores. For example, with a 5-minute TTL, after an item has been in the cache for 5 minutes, DAX will remove it. The next request for that item will be a cache miss, forcing DAX to fetch the latest version from DynamoDB, which then gets cached for another 5 minutes.
</details>

---


**14. What happens if the primary node in a multi-node DAX cluster fails?**

- A. The entire cluster becomes unavailable until the node is replaced.
- B. The cluster becomes read-only until the node is replaced.
- C. DAX automatically promotes one of the read replicas to be the new primary node.
- D. All writes are sent directly to DynamoDB, bypassing the cache.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** DAX is a highly available service. In a multi-node cluster, if the primary node fails, DAX will automatically detect the failure and promote an existing read replica to take over as the new primary, typically within a few seconds.
</details>

---


**15. What is required in your VPC to allow your EC2 instances to connect to your DAX cluster?**

- A. An Internet Gateway.
- B. A NAT Gateway.
- C. A security group rule that allows TCP traffic on the DAX port (e.g., 8111) from your application's security group.
- D. A VPC Peering connection.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A DAX cluster is accessed via a TCP port within your VPC. Therefore, the security group attached to your application instances (the source) must have an outbound rule allowing traffic on the DAX port to the DAX cluster's security group. Conversely, the DAX cluster's security group must have an inbound rule allowing traffic on that port from the application's security group.
</details>

---


**16. Does using DAX reduce the cost of a read-heavy application running on a DynamoDB table with Provisioned Throughput?**

- A. No, it adds to the cost because you have to pay for the DAX cluster.
- B. Yes, because by serving reads from its cache, it reduces the number of Read Capacity Units (RCUs) consumed from the table.
- C. Both A and B are true; it's a trade-off.
- D. No, DAX costs are always higher than the RCU savings.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This is a classic cost-optimization trade-off. You pay for the DAX cluster itself (an added cost, A). However, if your application is very read-heavy with a high cache hit ratio, DAX can dramatically reduce the number of RCUs you need to provision on your DynamoDB table, leading to significant savings on that side (B). The overall financial benefit depends on whether the RCU savings outweigh the cost of the DAX cluster.
</details>

---


**17. Which of the following is NOT a benefit of using DAX?**

- A. Improved read performance, with latency reduced to microseconds.
- B. Reduced read load on the underlying DynamoDB table.
- C. Improved write performance for `PutItem` operations.
- D. Minimal code changes required for integration due to an API-compatible client.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** DAX is a read cache and a write-through cache. It does not improve write performance. It is designed solely to accelerate read operations.
</details>

---


**18. You are deciding whether to use DAX or a Global Secondary Index (GSI) to improve performance. The application needs to query data based on a non-key attribute. What should you use?**

- A. DAX, because it makes all queries faster.
- B. A GSI, because it provides an efficient query path for the non-key attribute.
- C. Both, use a GSI and put DAX in front of it.
- D. Neither, you should scan the table and cache the results in ElastiCache.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a problem of access patterns, not just read speed. If your application needs to query by an attribute that isn't the primary key, DAX will not help; a query on a non-key attribute would require an inefficient Scan operation. The correct solution is to create a GSI to provide an efficient index for the new query pattern. You could then optionally put DAX in front of the table to cache results from the GSI query (C), but the GSI itself is the essential first step.
</details>

---

**19. What does the "write-through" cache strategy mean for DAX?**

- A. Data is written to the cache and acknowledged, then written to DynamoDB later.
- B. The DAX client writes to DynamoDB first, then updates the DAX cache.
- C. Data is written to the DAX cache and to DynamoDB at the same time, and the operation is only successful if both writes succeed.
- D. The cache is bypassed entirely for write operations.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Write-through means the cache and the backing store are updated in a single, synchronous operation. This ensures that the cache and the database remain consistent for any data written through the cache.
</details>
---

**20. A DAX cluster is placed in front of a DynamoDB table. An administrator updates an item directly in the DynamoDB table using the AWS Management Console. What is the immediate effect on the DAX cache?**

- A. DAX is automatically notified and updates its cache with the new item.
- B. The item in the DAX cache is now stale and will remain so until its TTL expires or it is evicted.
- C. The update to the DynamoDB table will fail because DAX is present.
- D. The DAX cache for that item is immediately invalidated.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** DAX is not aware of writes that bypass it and go directly to the table. If you update the table directly, the old version of the item in the DAX cache becomes stale. DAX will continue to serve this stale data until the item's cache TTL expires, at which point it will fetch the new version from DynamoDB on the next cache miss. This is why it's a best practice to direct all reads and writes through the DAX endpoint.
</details>
---

**21. A DAX cluster is configured with 1 primary node and 2 read replicas. How many nodes are there in total?**

- A. 1
- B. 2
- C. 3
- D. 4
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The total number of nodes in a DAX cluster is the primary node plus the number of read replicas. In this case, 1 primary + 2 replicas = 3 nodes in total.
</details>
---

**22. Which of the following is the MOST important factor when deciding to use DAX?**

- A. The number of tables in your AWS account.
- B. The geographic location of your users.
- C. The need for strongly consistent reads.
- D. A high volume of read requests for a small set of popular items.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** The business case for DAX is strongest when your application exhibits a read-heavy access pattern with a high cache-hit ratio. This means you are frequently reading the same "hot" items over and over again. This is the scenario where an in-memory cache provides the most significant performance improvement and potential for cost savings on RCUs.
</details>

