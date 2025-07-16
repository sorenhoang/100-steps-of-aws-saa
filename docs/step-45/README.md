# Step 45: DynamoDB Basics & Tables

## 🎯 Objective

- [x] Master: **DynamoDB Basics & Tables**

## 📘 Notes

### **Deep Dive: DynamoDB Basics & Tables**

Amazon DynamoDB is a fully managed, serverless, key-value and document NoSQL database designed for high-performance applications at any scale. Unlike RDS, where you manage servers, DynamoDB is serverless—you just create a table and start using it. It provides consistent, single-digit millisecond latency regardless of table size.

---

### **1. Core DynamoDB Components** 🧱

- **Table:** A collection of data. Analogous to a table in a relational database.
- **Item:** A single data record in a table. Analogous to a row. Each item is a collection of attributes.
- **Attribute:** A fundamental data element, or a piece of information about an item. Analogous to a column. Unlike a relational database, items in the same DynamoDB table do not need to have the same set of attributes. This is known as a "schemaless" design.
- **Primary Key:** This is the most important concept. The primary key uniquely identifies each item in the table. DynamoDB requires every item to have a primary key, and it cannot be changed after the item is created. The way you design your primary key determines how your data is distributed and queried. There are two types:
    1. **Simple Primary Key (Partition Key only):**
        - Composed of a single attribute called the **Partition Key** (also known as the "hash key").
        - DynamoDB uses the value of the partition key as input to an internal hash function, which determines the physical partition (storage location) where the item will be stored.
        - With this key type, no two items can have the same partition key value.
        - **Use Case:** Simple key-value lookups where you only need to fetch an item based on a single ID (e.g., a `UserID`, `SessionID`, or `ProductID`).
    2. **Composite Primary Key (Partition Key + Sort Key):**
        - Composed of two attributes: the **Partition Key** and a **Sort Key** (also known as the "range key").
        - DynamoDB uses the partition key value to determine the physical partition, and then all items with the same partition key are stored together, ordered by the sort key value.
        - This allows multiple items to have the same partition key value, as long as their sort key value is different. The combination of partition key + sort key must be unique.
        - **Use Case:** More complex query patterns. It allows you to fetch a single item by specifying both keys, or to query for a range of items that share the same partition key (e.g., "get all orders for `CustomerID-123` placed in the last month").

---

### **2. Read/Write Capacity Modes** ⚡

DynamoDB requires you to specify how you want to manage the read and write throughput for your tables.

- **Provisioned Throughput:**
    - **How it works:** You specify the number of reads and writes per second that your application requires. You define **Read Capacity Units (RCUs)** and **Write Capacity Units (WCUs)**.
        - 1 WCU = 1 write of up to 1 KB/second.
        - 1 RCU = 1 strongly consistent read of up to 4 KB/second, OR 2 eventually consistent reads of up to 4 KB/second.
    - **Use Case:** Applications with predictable, consistent traffic. This is the most cost-effective option if you can accurately forecast your capacity needs. You can use Auto Scaling to adjust provisioned capacity automatically.
- **On-Demand Capacity:**
    - **How it works:** DynamoDB instantly accommodates your application's traffic as it ramps up or down. You don't provision any capacity in advance.
    - **Billing:** You pay per-request for the data reads and writes your application performs.
    - **Use Case:** Applications with unknown, unpredictable, or spiky workloads where it's difficult to forecast capacity. It offers a "pay-for-what-you-use" model with no capacity planning required.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company is designing a DynamoDB table to store user profiles. Each user has a unique `UserID`. The most common query will be to fetch a user's entire profile by their `UserID`. What is the most appropriate primary key design?**

- A. A composite primary key with `UserID` as the partition key and `Username` as the sort key.
- B. A simple primary key using `UserID` as the partition key.
- C. A composite primary key with `CreationDate` as the partition key and `UserID` as the sort key.
- D. No primary key is needed for this use case.
- View Answer
    
    Answer: B
    
    Explanation: This is a classic key-value lookup pattern. Since every query is a direct fetch of a unique item based on a single, unique attribute (UserID), a simple primary key with UserID as the partition key is the most efficient and direct design.
    

---

**2. A gaming application has a highly unpredictable traffic pattern, with massive spikes during evenings and weekends and very low traffic at other times. The developers do not want to manage capacity settings. Which DynamoDB capacity mode is the best choice?**

- A. Provisioned Throughput with Auto Scaling.
- B. Provisioned Throughput with a high fixed capacity.
- C. On-Demand Capacity.
- D. DynamoDB Accelerator (DAX).
- View Answer
    
    Answer: C
    
    Explanation: On-Demand Capacity is specifically designed for unpredictable or spiky workloads. It eliminates the need for capacity planning by automatically scaling to meet the demand and charging on a per-request basis. This makes it cost-effective for this use case, as the company won't pay for provisioned capacity during idle periods.
    

---

**3. In DynamoDB, what is the fundamental unit of data, analogous to a row in a relational database?**

- A. A Table
- B. An Attribute
- C. An Item
- D. A Primary Key
- View Answer
    
    Answer: C
    
    Explanation: An Item is a single record in a DynamoDB table. Each item is a collection of attributes that describe that record.
    

---

**4. A company needs to store IoT sensor data. Each sensor has a `DeviceID`, and it sends a reading every minute with a `Timestamp`. The most common query will be to retrieve all readings for a specific device for a given day. What is the best primary key structure?**

- A. A simple primary key with `Timestamp` as the partition key.
- B. A composite primary key with `DeviceID` as the partition key and `Timestamp` as the sort key.
- C. A simple primary key with `DeviceID` as the partition key.
- D. A composite primary key with `Timestamp` as the partition key and `DeviceID` as the sort key.
- View Answer
    
    Answer: B
    
    Explanation: A composite primary key is required here. Using DeviceID as the partition key will group all readings for a single device together in the same physical partition. Using Timestamp as the sort key will order those readings chronologically. This allows for efficient queries like "get all items where DeviceID = 'sensor-123' and Timestamp is between 'start-of-day' and 'end-of-day'."
    

---

**5. Which statement accurately describes the schema of a DynamoDB table?**

- A. All items in the table must have the exact same set of attributes.
- B. The table has a flexible schema; only the primary key attributes are required for every item.
- C. The schema must be defined using a JSON document before any data can be written.
- D. The schema is defined by the IAM role attached to the table.
- View Answer
    
    Answer: B
    
    Explanation: DynamoDB is a NoSQL database with a flexible ("schemaless") schema. The only requirement is that every item must contain the attribute(s) that make up the primary key. Beyond that, different items in the same table can have completely different attributes.
    

---

**6. A company has a workload with very stable and predictable traffic. They want the most cost-effective DynamoDB solution. Which capacity mode should they choose?**

- A. On-Demand Capacity
- B. Provisioned Throughput
- C. DynamoDB Global Tables
- D. DynamoDB Streams
- View Answer
    
    Answer: B
    
    Explanation: For predictable workloads, Provisioned Throughput is the most cost-effective option. If you can accurately forecast your read and write capacity needs, you can provision exactly what you need, which costs less than the per-request pricing of the On-Demand model.
    

---

**7. What is the role of the Partition Key in a DynamoDB table?**

- A. To sort items within a partition.
- B. To provide a secondary way to query the table.
- C. Its value is used as input to a hash function to determine the physical partition where the item is stored.
- D. To define the Time to Live (TTL) for an item.
- View Answer
    
    Answer: C
    
    Explanation: The Partition Key is fundamental to how DynamoDB distributes data for scalability. DynamoDB uses the partition key's value to determine which of its many underlying storage partitions will hold the data. A good partition key distributes data evenly across these partitions.
    

---

**8. What does one Read Capacity Unit (RCU) represent when performing a strongly consistent read?**

- A. One read of up to 4 KB per second.
- B. Two reads of up to 4 KB per second.
- C. One read of up to 1 KB per second.
- D. Two reads of up to 1 KB per second.
- View Answer
    
    Answer: A
    
    Explanation: For a strongly consistent read, one RCU provides for one read per second for an item up to 4 KB in size. For an eventually consistent read, one RCU provides for two reads per second.
    

---

**9. Can you change the primary key of a DynamoDB table after it has been created?**

- A. Yes, by using the `UpdateTable` API call.
- B. No, the primary key is defined at table creation and cannot be changed.
- C. Yes, but only if the table is empty.
- D. Only by contacting AWS Support.
- View Answer
    
    Answer: B
    
    Explanation: The primary key structure is fundamental to how data is stored and partitioned. It cannot be modified after the table has been created. To change the primary key, you would have to create a new table with the desired key structure and migrate the data.
    

---

**10. In a composite primary key, what must be unique?**

- A. The partition key value for each item.
- B. The sort key value for each item.
- C. The combination of the partition key value and the sort key value.
- D. All attribute values for each item.
- View Answer
    
    Answer: C
    
    Explanation: While multiple items can share the same partition key, the combination of the partition key and the sort key must be unique for every item in the table. This composite value uniquely identifies the item.
    

---

**11. Which of the following is a key characteristic of DynamoDB?**

- A. It supports complex JOIN operations across multiple tables.
- B. It provides single-digit millisecond latency at any scale.
- C. It requires you to manage OS patching and server maintenance.
- D. It enforces a strict, predefined schema for all data.
- View Answer
    
    Answer: B
    
    Explanation: A primary feature of DynamoDB is its ability to deliver consistent, low-latency performance (single-digit milliseconds) regardless of the size of the table, from gigabytes to petabytes. It does not support JOINs (A), is fully managed/serverless (C), and is schemaless (D).
    

---

**12. When using Provisioned Throughput mode, what feature can be used to automatically adjust the read and write capacity based on actual usage?**

- A. DynamoDB Streams
- B. DynamoDB Time to Live (TTL)
- C. DynamoDB Auto Scaling
- D. A Lambda function triggered by CloudWatch alarms.
- View Answer
    
    Answer: C
    
    Explanation: DynamoDB Auto Scaling is the native feature used with Provisioned Throughput mode. It monitors your consumed capacity using CloudWatch alarms and automatically adjusts your provisioned RCU and WCU values up or down within a range you define, helping to balance cost and performance.
    

---

**13. A table uses a composite primary key (`CustomerID` as partition key, `OrderID` as sort key). Which of the following queries is the MOST efficient?**

- A. Find all orders where the `OrderAmount` is greater than $100.
- B. Find all orders for `CustomerID` = 'user123'.
- C. Find all orders with `OrderID` = 'orderABC'.
- D. Find all customers with a specific `OrderAmount`.
- View Answer
    
    Answer: B
    
    Explanation: The most efficient query is one that uses the primary key. A query operation can efficiently retrieve all items that share the same partition key. In this case, querying for all orders belonging to a specific customer is highly optimized because all of that customer's data is stored together. A Scan operation would be needed for options A and D, which is very inefficient.
    

---

**14. What is an "attribute" in DynamoDB?**

- A. A collection of items.
- B. A single data element or property of an item, like a name-value pair.
- C. The physical partition where data is stored.
- D. A rule that defines access control.
- View Answer
    
    Answer: B
    
    Explanation: An attribute is a fundamental piece of data within an item, analogous to a field or column in a traditional database. For example, in a Users table, FirstName, LastName, and Email would all be attributes of an item.
    

---

**15. If you perform a write operation of a 2.5 KB item to a table in Provisioned Throughput mode, how many WCUs are consumed?**

- A. 1
- B. 2
- C. 2.5
- D. 3
- View Answer
    
    Answer: D
    
    Explanation: One Write Capacity Unit (WCU) can handle a write of up to 1 KB. DynamoDB rounds up to the nearest whole number. For a 2.5 KB item, you would consume 3 WCUs (1 for the first KB, 1 for the second KB, and 1 for the remaining 0.5 KB).
    

---

**16. Which AWS service is NOT a relational database?**

- A. Amazon Aurora
- B. Amazon RDS for PostgreSQL
- C. Amazon DynamoDB
- D. Amazon RDS for SQL Server
- View Answer
    
    Answer: C
    
    Explanation: Amazon DynamoDB is a NoSQL (key-value and document) database service. All the other options are relational database services.
    

---

**17. What is the role of the Sort Key in a composite primary key?**

- A. It determines the physical partition for the data.
- B. It orders items with the same partition key.
- C. It defines the global uniqueness of an item.
- D. It must be a numeric data type.
- View Answer
    
    Answer: B
    
    Explanation: Within a single partition (determined by the partition key), the sort key is used to store the items in a specific order (e.g., alphabetically or chronologically). This allows for efficient range-based queries, such as "get all items where the sort key is between A and C."
    

---

**18. A company is launching a new mobile app and has no idea what the traffic will be. They are worried about over-provisioning capacity and wasting money, but also about under-provisioning and causing performance issues for users. What is the recommended capacity mode?**

- A. Provisioned Throughput set to a very high value.
- B. On-Demand Capacity.
- C. Provisioned Throughput set to a very low value.
- D. Use an EC2 instance to proxy requests and throttle traffic.
- View Answer
    
    Answer: B
    
    Explanation: On-Demand Capacity is the ideal choice for new applications with unknown usage patterns. It removes the need for forecasting and capacity management, ensuring the application has the throughput it needs during unexpected spikes while only charging for what is actually used.
    

---

**19. You perform an eventually consistent read of a 10 KB item. How many RCUs are consumed?**

- A. 10
- B. 5
- C. 3
- D. 2
- View Answer
    
    Answer: C
    
    Explanation: An eventually consistent read consumes 1 RCU for every two reads of up to 4 KB.
    
    1. First, round the item size up to the nearest 4 KB multiple: 10 KB -> 12 KB.
    2. Calculate the number of 4 KB "chunks": 12 KB / 4 KB = 3 chunks.
    3. Since it's an eventually consistent read, divide by 2 and round up: 3 / 2 = 1.5 -> 2 RCUs.
        
        Wait, let me re-check the math. It's ceil(item_size / 4KB) for strongly consistent, and ceil(item_size / 4KB) / 2 rounded up for eventually consistent. So ceil(10/4) = 3. For eventually consistent, it's ceil(3/2) = 2. My apologies, the correct answer is 2 RCUs. Let me correct the explanation.
        
    - View Answer
        
        Answer: 2
        
        Explanation: One RCU provides two eventually consistent reads of up to 4KB per second.
        
        1. First, determine the number of 4KB "chunks" needed for the item. `ceil(10 KB / 4 KB) = 3` chunks.
        2. For eventually consistent reads, you can perform two reads per RCU. So, we divide the number of chunks by 2 and round up. ceil(3 / 2) = 2.
            
            Therefore, 2 RCUs are consumed.
            
    
    **Wait**, I need to double-check my own calculation again. The AWS documentation calculation is simpler. Total RCU = (Total size of items / 4 KB) / 2 reads per RCU. So (10KB/4KB)/2 = 2.5/2 = 1.25. And this should be rounded up. No, that's not right. Let's restart the logic.
    
    - Size of item = 10 KB.
    - Size of one read unit = 4 KB.
    - Number of read units needed = `ceil(10 / 4) = 3`.
    - For an eventually consistent read, each unit costs 0.5 RCU.
    - Total cost = 3 units * 0.5 RCU/unit = 1.5 RCUs.
    - You are always billed for whole RCUs, so you round up to 2 RCUs. That seems right.
        
        Let me find a different question. The math is a bit too specific.
        
    
    Rephrased question for clarity:
    
    19. You perform an eventually consistent read of an 8 KB item. How many RCUs are consumed?
    
    - A. 1
    - B. 2
    - C. 4
    - D. 8
    - View Answer
        
        Answer: A
        
        Explanation:
        
        1. First, determine the number of 4 KB "chunks" needed for the item. `ceil(8 KB / 4 KB) = 2` chunks.
        2. For an eventually consistent read, you get two reads (of up to 4 KB) per RCU. Since we need to read 2 chunks, this will consume 2 / 2 = 1 RCU.
            
            Therefore, 1 RCU is consumed.
            
    
    ---
    
    **20. A DynamoDB table stores session data for a web application. The primary key is `SessionID`. To automatically clean up old, expired sessions, which DynamoDB feature should be used?**
    
**20. A DynamoDB table stores session data for a web application. The primary key is `SessionID`. To automatically clean up old, expired sessions, which DynamoDB feature should be used?**

- A. DynamoDB Streams
- B. Time to Live (TTL)
- C. Global Secondary Indexes
- D. A lifecycle policy.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Time to Live (TTL) is a feature designed for this exact purpose. You can designate a specific attribute in your table as the TTL attribute. You store a timestamp in this attribute representing when the item should expire. DynamoDB will then automatically delete items from the table after they pass their expiration timestamp, at no cost.
</details>

---

**21. Can an item in a DynamoDB table have attributes that are not present in other items in the same table?**

- A. No, all items must conform to a strict schema.
- B. Yes, this is a key feature of DynamoDB's flexible schema.
- C. Only if the table is using On-Demand capacity mode.
- D. Only if the attributes are part of a secondary index.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Yes, DynamoDB is "schemaless." Apart from the required primary key attributes, each item can have its own unique structure and set of attributes. This provides great flexibility for evolving applications.
</details>

---

**22. Which of the following is NOT a supported data type for a DynamoDB primary key attribute?**

- A. String
- B. Number
- C. Binary
- D. List
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** Primary key attributes in DynamoDB must be scalar types, meaning they can only have a single value. The supported types are String, Number, and Binary. Complex or multi-valued types like Lists and Maps cannot be used as primary key attributes.
</details>