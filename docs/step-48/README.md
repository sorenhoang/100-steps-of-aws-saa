# Step 48: ElastiCache – Redis vs Memcached

## 🎯 Objective
- [x] Master: **ElastiCache – Redis vs Memcached**

## 📘 Notes

### **Deep Dive: ElastiCache – Redis vs. Memcached**

Amazon ElastiCache is a fully managed, in-memory data store and caching service. Its primary purpose is to improve the performance of web applications by allowing you to retrieve information from a fast, managed, in-memory system instead of relying entirely on slower, disk-based databases. It supports two popular open-source caching engines: Redis and Memcached. Choosing the right engine is critical as they offer very different feature sets.

---

### **1. Amazon ElastiCache for Memcached**

Memcached is a simple, high-performance, multithreaded, in-memory key-value store. Think of it as a basic, distributed memory object caching system.

- **Data Model:** Simple **key-value** store only. Values are treated as opaque strings or blobs.
- **Performance:** Extremely fast and simple. It is **multithreaded**, meaning it can use multiple processing cores on a node to handle a very high volume of requests, especially for large clusters.
- **Scalability:** Scales horizontally by adding more nodes to the cluster. The client application is responsible for sharding data across the available nodes.
- **Persistence:** **No persistence**. It is an in-memory-only cache. If a node fails or is rebooted, all data on that node is lost.
- **High Availability:** **No built-in high availability**. A node failure results in data loss for that node's cache.
- **Use Case:** The best choice for the **simplest caching needs**. Ideal for caching relatively static data, such as the results of database queries or rendered HTML pages, where losing the cache is not critical and it can be easily repopulated from the source.

---

### **2. Amazon ElastiCache for Redis**

Redis (Remote Dictionary Server) is a much more feature-rich in-memory data store. It goes far beyond simple key-value caching.

- **Data Model:** Supports complex **data structures**, including Strings, Lists, Sets, Sorted Sets, Hashes, and Bitmaps. This allows for more advanced application logic.
- **Performance:** Very fast, but it is primarily **single-threaded**. This means a single node can only use one CPU core.
- **Persistence:** **Offers persistence**. You can configure Redis to take snapshots (`.rdb` files) or use an Append Only File (`.aof`) to persist the cache data to disk, allowing for recovery after a reboot.
- **High Availability:** Supports **Multi-AZ with automatic failover**. You can create a primary node and one or more read replicas in different Availability Zones. If the primary node fails, ElastiCache will automatically promote a read replica to become the new primary.
- **Advanced Features:**
    - **Pub/Sub:** Supports publish/subscribe messaging capabilities for real-time communication.
    - **Replication:** Supports read replicas to scale read traffic.
    - **Transactions:** Supports Multi/Exec transactions for atomic operations.
- **Use Case:** A more versatile tool suitable for:
    - Advanced caching.
    - Session storage for user login data.
    - Real-time analytics.
    - Gaming leaderboards (using Sorted Sets).
    - Messaging queues (using Lists or Pub/Sub).

---

### **Summary of Key Differences**

| Feature               | ElastiCache for Redis                         | ElastiCache for Memcached    |
| --------------------- | --------------------------------------------- | ---------------------------- |
| **Data Types**        | **Complex (Lists, Sets, Hashes, etc.)**       | Simple Key-Value only        |
| **High Availability** | **Yes (Multi-AZ with auto failover)**         | No                           |
| **Persistence**       | **Yes (Snapshots, AOF)**                      | No (In-memory only)          |
| **Threading Model**   | Single-threaded                               | **Multithreaded**            |
| **Pub/Sub**           | **Yes**                                       | No                           |
| **Use Case**          | Advanced caching, Leaderboards, Session Store | **Simplest form of caching** |

---

### **AWS SAA-C03 Style Questions & Explanations**


**1. A company needs to implement a real-time leaderboard for a mobile game. The leaderboard must rank players by score and be updated with very low latency. Which ElastiCache engine and data type are best suited for this?**

- A. Memcached using a simple key-value pair.
- B. Redis using the Sorted Set data type.
- C. Redis using the List data type.
- D. Memcached, as it is multithreaded and faster.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a classic use case for Redis Sorted Sets. A sorted set allows you to store members (player IDs) with an associated score. Redis automatically keeps the set sorted by the score, making it extremely efficient to query for the top N players for the leaderboard. Memcached does not have a data structure that can handle this natively.
</details>

---


**2. An application needs a simple caching layer to store the results of expensive database queries. The data is not critical, and if the cache is lost, it can be easily repopulated from the database. The primary goal is raw speed and simplicity. Which ElastiCache engine should be chosen?**

- A. Redis, because it offers persistence.
- B. Memcached, because it is simple, multithreaded, and designed for this use case.
- C. Redis, because it has Multi-AZ failover.
- D. Memcached, because it supports complex data types.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** For the simplest form of caching where durability is not a concern, Memcached is the ideal choice. Its simplicity and multithreaded nature allow it to handle a high volume of simple get/set requests very efficiently. The advanced features of Redis (persistence, HA) are not needed here and would add unnecessary complexity and cost.
</details>

---


**3. A company wants to use ElastiCache to store user session data for a busy e-commerce website. The session data must be highly available and survive a cache node failure. Which configuration should be used?**

- A. ElastiCache for Memcached cluster.
- B. ElastiCache for Redis with Multi-AZ enabled.
- C. A single, large Redis node.
- D. A single, large Memcached node.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The key requirement is high availability. Only Redis supports a Multi-AZ configuration with automatic failover. By enabling this feature, ElastiCache will maintain a standby replica in a different AZ. If the primary node fails, ElastiCache will automatically promote the standby, ensuring the session data is not lost. Memcached does not have this capability.
</details>

---


**4. Which of the following is a key difference between Redis and Memcached?**

- A. Redis is multithreaded, while Memcached is single-threaded.
- B. Redis supports data persistence, while Memcached is in-memory only.
- C. Memcached supports complex data structures like Sets and Lists.
- D. Redis is less expensive than Memcached.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** One of the most significant differences is persistence. Redis can be configured to persist its data to disk via snapshots or an append-only file, allowing for data recovery after a reboot or failure. Memcached is purely an in-memory cache, and all of its data is lost if a node goes down.
</details>

---


**5. An application uses ElastiCache for caching. The developers notice that when a node in their Memcached cluster fails, a large number of requests suddenly go to the backend database. What is this phenomenon called?**

- A. A cache hit.
- B. A cold start.
- C. The thundering herd problem.
- D. A failover event.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The thundering herd problem occurs when a cache node fails, and all the clients that were previously getting cache hits from that node suddenly send their requests to the backend database simultaneously. This massive, sudden increase in load can overwhelm the database.
</details>

---


**6. Which ElastiCache engine provides built-in support for Publish/Subscribe (Pub/Sub) messaging?**

- A. Memcached
- B. Both Redis and Memcached
- C. Redis
- D. Neither, you must use Amazon SQS or SNS for messaging.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Redis has built-in support for a lightweight Pub/Sub messaging paradigm. This allows clients to subscribe to channels and receive messages that are published to those channels in real-time, which is useful for chat applications or live notifications.
</details>

---


**7. When would you choose Memcached over Redis?**

- A. When you need high availability and automatic failover.
- B. When your application requires complex data types like sorted sets.
- C. When you need the simplest possible key-value caching model and can tolerate data loss on node failure.
- D. When you need to back up your cache data to S3.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** You choose Memcached when your only requirement is a simple, fast, in-memory key-value cache. If the advanced features of Redis like persistence, HA, and complex data types are not needed, Memcached provides a simpler and often higher-throughput solution for basic caching.
</details>

---


**8. How does ElastiCache for Redis handle high availability?**

- A. It uses a shared storage volume across multiple nodes.
- B. It automatically distributes data across multiple nodes, and if one fails, the data is served from another.
- C. By using a Multi-AZ deployment with a primary node and a synchronous standby replica in a different AZ with automatic failover.
- D. Memcached does not support high availability.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The question asks about Redis, not Memcached. ElastiCache for Redis achieves high availability by using a Multi-AZ configuration. It creates a primary node and replicates data to a standby node in a different Availability Zone. If the primary fails, ElastiCache promotes the standby automatically.
</details>

---


**9. Which engine is multithreaded, allowing it to better utilize multi-core CPU nodes?**

- A. Redis
- B. Memcached
- C. Both are multithreaded.
- D. Both are single-threaded.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Memcached is multithreaded, which means it can handle many concurrent connections and operations by utilizing all available CPU cores on a node. Redis is primarily single-threaded, meaning it handles operations sequentially on a single core (though it uses other threads for background tasks).
</details>

---


**10. You need to implement caching for your application. The cached data must be recoverable even if the entire ElastiCache cluster is rebooted. Which engine must you choose?**

- A. Memcached, because it's faster.
- B. Redis, because it supports data persistence.
- C. Either engine can be used.
- D. This is not possible with ElastiCache.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The key requirement is data recoverability after a reboot, which requires persistence. Only Redis offers persistence features like RDB snapshots and AOF (Append Only File) logging, which save the in-memory data to disk.
</details>

---


**11. A social media application needs to store the timeline for each user. A user's timeline is a collection of post IDs that should be stored in chronological order. Which Redis data type is most appropriate for this?**

- A. A Hash
- B. A Set
- C. A List
- D. A String
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A Redis List is an ordered sequence of strings. It's perfect for modeling a timeline, where you can easily add new post IDs to the beginning or end of the list (LPUSH, RPUSH) and retrieve a range of recent posts (LRANGE).
</details>

---


**12. When scaling a Memcached cluster horizontally, which component is typically responsible for determining which node to store or retrieve a key from?**

- A. The ElastiCache service itself.
- B. The client application's caching library.
- C. A Network Load Balancer.
- D. Amazon Route 53.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Memcached clusters use client-side sharding. The client library (e.g., in your application code) is given a list of all the node endpoints in the cluster. It then uses a hashing algorithm (like consistent hashing) to determine which specific node a given key belongs to.
</details>

---


**13. Which statement accurately describes ElastiCache security?**

- A. ElastiCache clusters are public by default.
- B. ElastiCache clusters are deployed into a VPC and access is controlled by security groups.
- C. Access is controlled by IAM policies attached to the cluster.
- D. All clusters must be accessed over the public internet.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** ElastiCache is a VPC-native service. You launch your cluster into subnets within your VPC. Access to the cluster nodes is then controlled by VPC security groups. You would create a security group for your application servers and configure the ElastiCache security group to only allow inbound traffic from that specific application security group.
</details>

---


**14. What is a primary use case for ElastiCache?**

- A. To provide long-term, durable object storage.
- B. To run a relational database.
- C. To improve application performance by reducing the load on backend databases.
- D. To host a static website.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The main purpose of a cache like ElastiCache is to store frequently accessed data in memory. This allows applications to retrieve data much faster than going to a disk-based database, which reduces the latency for the user and lessens the read load on the primary database.
</details>

---


**15. If your primary requirement is to support transactions that involve multiple keys, which engine should you choose?**

- A. Memcached
- B. Redis
- C. Either engine supports transactions.
- D. Transactions are not supported by in-memory databases.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Redis supports transactions through commands like MULTI and EXEC. This allows you to group multiple commands together that are guaranteed to be executed sequentially as a single, atomic operation. Memcached does not have a comparable feature.
</details>

---


**16. What is a "lazy loading" or "cache-aside" caching strategy?**

- A. The application loads all possible data into the cache before it is needed.
- B. The application writes data to the cache and the database at the same time.
- C. The application first requests data from the cache. If it's a miss, it gets the data from the database and then writes it into the cache.
- D. The cache automatically expires data after a set TTL.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Lazy loading is the most common caching strategy. The logic is: 1) Look for data in the cache. 2) If it's not there (a cache miss), query the database. 3) Write the data received from the database into the cache. 4) Return the data to the application. This ensures that only data that is actually requested gets loaded into the cache.
</details>

---


**17. ElastiCache for Redis supports read replicas. What is the purpose of these replicas?**

- A. To provide high availability with automatic failover.
- B. To scale the read throughput of the Redis cluster.
- C. To provide a writeable copy of the data in another region.
- D. To persist the cache data to disk.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Similar to RDS Read Replicas, ElastiCache for Redis read replicas are used to offload read queries from the primary node. By directing read traffic to one or more replicas, you can significantly increase the overall read capacity of your Redis cluster. High availability is provided by the separate Multi-AZ feature.
</details>

---


**18. You need to implement a simple key-value cache that can handle a very high number of requests per second on a large, multi-core instance. Which engine's architecture is better suited for this?**

- A. Redis, because it is more feature-rich.
- B. Memcached, because its multithreaded architecture can take full advantage of multiple CPU cores.
- C. Redis, because it is single-threaded, which is more efficient.
- D. Both are equally suited for this task.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Because Memcached is multithreaded, it can scale its performance vertically by using all the available cores on a large compute node. For simple get/set operations at a massive scale on a single large node, Memcached's architecture often provides higher throughput than the single-threaded Redis.
</details>

---


**19. Which of the following is NOT a valid data type in Redis?**

- A. Sorted Set
- B. Hash
- C. Table
- D. List
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Redis is a key-value store with advanced data structures. It does not have a "Table" data type, which is a construct from relational databases. Lists, Hashes, and Sorted Sets are all valid Redis data types.
</details>

---


**20. A user wants to ensure that if their primary Redis node fails, a replica in another Availability Zone is promoted automatically. What feature must be enabled?**

- A. Cluster Mode
- B. Persistence
- C. Multi-AZ with Automatic Failover
- D. Read Replicas
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Multi-AZ with Automatic Failover is the specific ElastiCache for Redis feature that provides high availability. It creates a standby replica and manages the entire detection and promotion process if the primary node fails. Read Replicas (D) are for read scaling, not automatic failover.
</details>

---


**21. In which scenario would you choose Redis over Memcached?**

- A. When you need the absolute simplest caching mechanism.
- B. When you require the cache data to be durable and survive reboots.
- C. When you want to maximize throughput on a single, large multi-core node.
- D. When you do not need high availability.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The need for durability and persistence is a primary reason to choose Redis. Its ability to snapshot data to disk means you can recover your cache state after a failure, a feature that Memcached completely lacks.
</details>

---


**22. How do you control network access to an ElastiCache cluster?**

- A. With an IAM policy attached to the cluster.
- B. With a username and password.
- C. With VPC Security Groups.
- D. With Network ACLs only.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** ElastiCache clusters are deployed into your VPC. Network access to the cluster's nodes is controlled by VPC Security Groups. A typical setup is to create one security group for the ElastiCache cluster and another for your application servers, and then create a rule in the cluster's security group that allows inbound traffic only from the application's security group.
</details>