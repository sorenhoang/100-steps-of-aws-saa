# Step 46: DynamoDB TTL, Streams, GSIs

## 🎯 Objective

- [x] Master: **DynamoDB TTL, Streams, GSIs**

## 📘 Notes

### **Deep Dive: DynamoDB TTL, Streams, & GSIs**

Beyond its basic key-value capabilities, DynamoDB offers advanced features that solve common application development challenges. Time to Live (TTL) handles ephemeral data, Streams enable event-driven architectures, and Global Secondary Indexes (GSIs) provide flexible query patterns.



### **1. DynamoDB Time to Live (TTL)** ⏳

TTL is a cost-free feature that allows you to define a per-item timestamp to determine when that item is no longer needed. DynamoDB automatically deletes items from your table once their TTL timestamp has expired, without consuming any write capacity.

- **How it works:**
    1. You enable TTL on a table and specify which attribute will hold the TTL value.
    2. For each item you want to expire, you add an attribute with that name. The value must be a **timestamp in Unix epoch time format** (the number of seconds since January 1, 1970).
    3. DynamoDB's background process periodically scans the table and deletes items that have passed their expiration timestamp. This process can take up to 48 hours after the expiration time, so it's not for mission-critical, instantaneous deletions.
- **Use Case:** Perfect for managing data with a limited lifecycle, such as:
    - User session data
    - Temporary application logs
    - Caching data
    - Event notifications that are only relevant for a short time



### **2. DynamoDB Streams** 🌊

DynamoDB Streams captures a time-ordered sequence of item-level modifications in any DynamoDB table. It's the "change data capture" (CDC) mechanism for DynamoDB.

- **How it works:** When you enable Streams on a table, every time an item is created, updated, or deleted, a "stream record" is created. This stream record contains information about the modification.
- **Stream Record Information:** You can configure how much information is written to the stream:1
    - **KEYS_ONLY:** Only the key attributes of the modified item.2
    - **NEW_IMAGE:** The entire item as it appeared *after* it was modified.3
    - **OLD_IMAGE:** The entire item as it appeared *before* it was modified.4
    - **NEW_AND_OLD_IMAGES:** Both the new and old images of the item.
- **Integration with AWS Lambda:** This is the most powerful and common pattern. You can configure a **Lambda function to be triggered** by events in the DynamoDB Stream. The stream records are sent to the Lambda function in batches for processing.
- **Use Cases:**
    - **Cross-Region Replication:** A Lambda function reads from a stream in one region and replicates the changes to a table in another region. (Note: DynamoDB Global Tables now automate this).
    - **Triggering Notifications:** When a new order is placed (a new item is created), trigger a Lambda function to send a confirmation email.
    - **Real-time Analytics:** Stream data modifications to services like Amazon OpenSearch or Redshift.



### **3. Global Secondary Indexes (GSI)** 🔍

A Global Secondary Index (GSI) allows you to perform queries on attributes that are not part of your table's primary key. It is essentially a copy of a subset of your table's data, organized by a different primary key.

- **How it works:**
    - You create a GSI on an existing table and specify a **different primary key** (which can be a simple or composite key) for the index.
    - You also specify which attributes from the main table should be **projected** (copied) into the index.
    - DynamoDB automatically maintains this index. When you write to the main table, DynamoDB asynchronously updates the GSI.
- **Key Characteristics:**
    - **Alternate Query Patterns:** Its sole purpose is to provide an efficient way to query your data using a different key structure. For example, your main table might be partitioned by `UserID`, but you could create a GSI partitioned by `EmailAddress` to allow for efficient lookups by email.
    - **Provisioned Throughput:** A GSI has its **own read and write capacity settings** that are completely separate from the base table's capacity. You must provision capacity for the index itself.
    - **Eventual Consistency:** Writes from the base table are replicated to the GSI **asynchronously**. This means there is a small delay. A read from a GSI might return data that is slightly stale or may not find an item that was just written to the base table.
    - **Flexibility:** You can create or delete GSIs on a table at any time without impacting the table itself.

*Note: There is also a **Local Secondary Index (LSI)**, which you must create at the same time as the table. It uses the same partition key as the base table but a different sort key. GSIs are more flexible and more commonly discussed.*

---

### **AWS SAA-C03 Style Questions & Explanations**


**1. A web application stores user session data in a DynamoDB table. To manage costs and keep the table clean, session data should be automatically deleted 48 hours after it is created. Which DynamoDB feature should be used?**

- A. A Global Secondary Index (GSI) on the creation date.
- B. A DynamoDB Stream triggering a Lambda function to delete old items.
- C. A lifecycle policy.
- D. Time to Live (TTL) on a timestamp attribute.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** This is the primary use case for Time to Live (TTL). You can add an attribute to each session item containing its expiration timestamp (current time + 48 hours). DynamoDB will then automatically delete the item after that timestamp has passed, at no additional cost. While a Lambda function (B) could work, TTL is the simpler, native, and cost-free solution.
</details>



**2. A company has a DynamoDB table of customer orders with `OrderID` as the partition key. They now have a new requirement to efficiently query for all orders placed by a specific customer using their `CustomerID`. How can they enable this new query pattern without changing the base table?**

- A. Perform a Scan operation on the table with a filter on `CustomerID`.
- B. Create a Global Secondary Index (GSI) with `CustomerID` as the partition key.
- C. Create a new table with `CustomerID` as the partition key and migrate the data.
- D. Enable DynamoDB Streams.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A Global Secondary Index (GSI) is designed for exactly this purpose: to provide alternate query patterns on an existing table. By creating a GSI with CustomerID as its partition key, you create a new queryable index that allows for efficient lookups by customer, while the main table remains structured by OrderID. A Scan (A) would be extremely inefficient and costly on a large table.
</details>


**3. An application needs to perform an action every time a new item is added to a DynamoDB table. What is the most efficient, event-driven way to trigger this action?**

- A. Write a script that scans the table every minute for new items.
- B. Enable DynamoDB Streams on the table and use it as a trigger for an AWS Lambda function.
- C. Use Amazon EventBridge to monitor the table.
- D. Enable TTL on the table.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** DynamoDB Streams provide a "change data capture" log for the table. The most common and powerful pattern is to configure the stream as an event source for a Lambda function. This creates a serverless, event-driven architecture where the function is automatically invoked in near real-time whenever an item is created, updated, or deleted.
</details>



**4. When you enable DynamoDB Streams for a table, which of the following is NOT a valid configuration for the data that is written to the stream?**

- A. `KEYS_ONLY` - Only the key attributes of the modified item.
- B. `NEW_IMAGE` - The item as it appears after the modification.
- C. `OLD_IMAGE` - The item as it appeared before the modification.
- D. `METADATA_ONLY` - Only the metadata about the modification, like the timestamp.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** The four valid StreamView types are KEYS_ONLY, NEW_IMAGE, OLD_IMAGE, and NEW_AND_OLD_IMAGES. There is no METADATA_ONLY option. The stream record itself contains metadata, but you must choose one of the four options to specify what item data is included.
</details>



**5. What is a key characteristic of a Global Secondary Index (GSI)?**

- A. It has the same primary key as the base table.
- B. It is always strongly consistent with the base table.
- C. It has its own provisioned read and write capacity that is separate from the base table.
- D. It must be created at the same time as the base table.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A GSI is effectively a separate table under the hood. Therefore, it requires its own provisioned throughput (RCUs and WCUs) to handle the writes that are replicated to it and the read queries that it serves. You must plan capacity for both your base table and each of its GSIs.
</details>



**6. A developer has implemented Time to Live (TTL) to delete old log entries. They notice that items are sometimes still present in the table up to 48 hours after their specified expiration timestamp. Is this expected behavior?**

- A. No, TTL deletions should be instantaneous.
- B. Yes, DynamoDB TTL is a background process, and deletion can take up to 48 hours after the expiration time.
- C. No, this indicates that the TTL attribute is not in the correct epoch time format.
- D. Yes, but only if the table is using On-Demand capacity.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Yes, this is expected. TTL deletion is not a real-time process. A background scanner periodically sweeps partitions for expired items. AWS states that deletion typically occurs within 48 hours. Applications should be designed to filter out items that have passed their TTL time, even if they haven't been deleted yet.
</details>



**7. An application writes a new item to a DynamoDB table that has a Global Secondary Index. The application immediately performs a read against the GSI to find that new item. Is the read guaranteed to find the item?**

- A. Yes, because all writes are synchronously replicated to the GSI.
- B. No, because GSIs are eventually consistent, and there might be a small replication lag.
- C. Yes, but only if the GSI uses the same partition key as the base table.
- D. No, because you cannot read from a GSI immediately after a write.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Replication from a base table to a GSI is asynchronous, which means the GSI is eventually consistent. There is a brief moment after a write to the base table before that write is reflected in the index. An immediate read from the GSI might not find the newly written item.
</details>


**8. You need to create an index that has the same partition key as the base table but a different sort key to provide a different sorting order for your items. This index must be created at the same time as the table. What type of index is this?**

- A. A Global Secondary Index (GSI)
- B. A Clustered Index
- C. A Local Secondary Index (LSI)
- D. A Composite Index
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** This describes a Local Secondary Index (LSI). Its defining characteristics are that it shares the same partition key as the base table and must be created when the table is created. It provides a different "local" sort order for the items within each partition. GSIs are more flexible and common.
</details>


**9. A DynamoDB Stream is configured to trigger a Lambda function. If there is a massive spike in writes to the table, what happens?**

- A. The Lambda function will be invoked once for every single item that is written.
- B. The stream records are batched together, and the Lambda function will be invoked with a batch of records.
- C. The stream will be throttled, and some change records will be lost.
- D. The Lambda function will fail because it cannot handle the volume.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The DynamoDB Streams Lambda integration is designed for efficiency. Instead of invoking the function for every single change (which would be inefficient and costly), the service batches the stream records together (e.g., up to 10,000 records per batch) and sends the entire batch to a single Lambda invocation for processing.
</details>


**10. What is a "projection" in the context of a Global Secondary Index?**

- A. The primary key of the index.
- B. The set of attributes from the base table that are copied into the index.
- C. A forecast of the index's future storage size.
- D. The read and write capacity units of the index.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** When you create a GSI, you must specify which attributes to project (copy) from the base table into the index. You can choose KEYS_ONLY, INCLUDE (specify certain attributes), or ALL. This choice is a trade-off between index storage size and query flexibility.
</details>




**11. What happens to items deleted by the TTL feature?**

- A. They are moved to the S3 Glacier storage class.
- B. They are captured in DynamoDB Streams just like a manual delete operation.
- C. They are permanently deleted but are not captured by DynamoDB Streams.
- D. They are stored in a hidden backup for 30 days.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A TTL-driven deletion is treated like any other deletion. It creates a stream record in the DynamoDB Stream (if enabled). This allows you to perform downstream actions on expired items, such as moving them to a long-term data warehouse before they are permanently deleted.
</details>




**12. Can a Global Secondary Index have a primary key type that is different from the base table's primary key?**

- A. No, it must use the same key attributes.
- B. Yes, for example, the base table could have a simple primary key while the GSI has a composite primary key.
- C. No, it must have the same partition key but can have a different sort key.
- D. Yes, but only if the base table is also a GSI.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Yes, this flexibility is a key feature of GSIs. The primary key of a GSI is completely independent of the base table's primary key. You can use any top-level String, Number, or Binary attributes from the base table to form a new simple or composite key for the GSI.
</details>




**13. Which feature would you use to create a "materialized view" or a different representation of your DynamoDB table data in Amazon OpenSearch for complex searching and analytics?**

- A. Time to Live (TTL)
- B. A Global Secondary Index
- C. DynamoDB Streams with a Lambda trigger.
- D. A database trigger inside DynamoDB.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The standard pattern for this is to use DynamoDB Streams and AWS Lambda. The Lambda function is triggered by changes in the DynamoDB table, and its code is responsible for transforming and writing that data into the Amazon OpenSearch cluster, keeping the search index up-to-date in near real-time.
</details>




**14. When should you choose to project `ALL` attributes into a GSI?**

- A. When you want to minimize the storage cost of the GSI.
- B. When your queries against the GSI require access to many different attributes that are not part of the index key.
- C. Always, as this is the recommended best practice.
- D. When you want to achieve strong consistency on the GSI.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** If you only project the keys (KEYS_ONLY), any query that needs other attributes would require a second lookup against the base table, which is slow and costly. By projecting ALL attributes, your GSI becomes a self-contained copy of the data for your queries, eliminating the need for this second lookup. The trade-off is higher storage cost and write capacity consumption for the index.
</details>



**15. A DynamoDB table is used to store game scores. The primary key is `UserID` (partition key) and `GameID` (sort key). You need a new query to find the top 10 highest scores across all users for a specific game (`GameID`). How would you achieve this efficiently?**

- A. Scan the entire table and sort the results.
- B. Create a GSI with `GameID` as the partition key and `Score` as the sort key.
- C. Create a Local Secondary Index with `Score` as the sort key.
- D. This query is not possible in DynamoDB.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The current key structure is inefficient for this query. By creating a GSI with GameID as the partition key and Score as the sort key, you create an index where all scores for a specific game are grouped together and sorted from highest to lowest. You can then query this GSI with GameID='game-abc', ScanIndexForward=false (to sort descending), and Limit=10 to get the top 10 scores very efficiently.
</details>



**16. How long are records retained in a DynamoDB Stream?**

- A. 1 hour
- B. 24 hours
- C. 7 days
- D. 35 days
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** DynamoDB Stream records are accessible for 24 hours. After that, they are automatically and permanently deleted. This means your processing application (like a Lambda function) must be able to keep up with the stream of changes.
</details>



**17. What is the cost associated with using the DynamoDB TTL feature?**

- A. A per-GB fee for the items being deleted.
- B. A small hourly fee for enabling TTL on the table.
- C. It consumes Write Capacity Units (WCUs) from the table.
- D. There is no additional cost; the deletion of expired items is free.
<details>
<summary>View Answer</summary>

**Answer:** D

**Explanation:** TTL is a free feature. The background deletion process does not consume any of your table's provisioned or on-demand write capacity.
</details>



**18. You are designing a table to store email messages, partitioned by `UserID` and sorted by `ReceiveDate`. You also need to be able to query for all emails that have `IsFlagged = true`. What should you do?**

- A. Create a GSI with `IsFlagged` as the partition key.
- B. Create a GSI with `UserID` as the partition key and `IsFlagged` as the sort key.
- C. Create a "sparse" GSI with `IsFlagged` as the partition key.
- D. Scan the table with a filter.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** IsFlagged is a low-cardinality attribute (it only has two values, true or false), so making it a partition key (A) is a bad design (a "hot" partition). The best solution is to create a sparse GSI. You would only write the IsFlagged attribute to an item when it is true. Then you create a GSI with IsFlagged as its partition key. Because only flagged items will even have the attribute, the index will be very small and efficient to query, containing only the items you care about.
</details>



**19. Which AWS service is most similar in function to DynamoDB Streams?**

- A. Amazon SQS
- B. Amazon Kinesis Data Streams
- C. Amazon SNS
- D. Amazon EventBridge
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Both DynamoDB Streams and Amazon Kinesis Data Streams are services for capturing a time-ordered sequence of records that can be processed by multiple consumers. They are both based on the concept of shards and ordered records. In fact, you can use Kinesis Data Streams to capture DynamoDB changes as an alternative to native DynamoDB Streams.
</details>



**20. A GSI on a table has very little provisioned Write Capacity. The base table is experiencing a huge spike in write traffic. What is a likely consequence?**

- A. Writes to the base table will be throttled to match the GSI's capacity.
- B. The GSI will automatically scale its write capacity.
- C. The replication lag between the base table and the GSI will increase significantly.
- D. Writes to the base table will fail.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** If the GSI cannot keep up with the rate of changes from the base table, writes to the GSI will be throttled. This does NOT block writes to the base table. Instead, the replication lag will grow, meaning the GSI will become increasingly stale until it has enough capacity to catch up.
</details>



**21. When setting up a TTL attribute on a DynamoDB item, what format must the value be in?**

- A. An ISO 8601 date string (e.g., "2025-07-16T14:30:00Z").
- B. A Number representing the Unix epoch time in seconds.
- C. A Number representing the Unix epoch time in milliseconds.
- D. A String representing the date (e.g., "July 16, 2025").
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The TTL attribute value must be a Number data type, and it must represent the time as a Unix epoch timestamp in seconds. Any other format will be ignored by the TTL deletion process.
</details>



**22. You need to provide a secondary query pattern for a table. You require the results from this secondary query to be strongly consistent. What type of index should you use?**

- A. A Global Secondary Index (GSI).
- B. A Local Secondary Index (LSI).
- C. This is not possible; all secondary indexes are eventually consistent.
- D. A sparse GSI.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Local Secondary Indexes (LSIs) are the only type of index in DynamoDB that can support strongly consistent reads. This is because an LSI shares the same partition key as its base table, meaning the index data is stored in the same physical partition as the base data. A GSI, being in a different partition (and possibly a different machine), can only be eventually consistent.
</details>


