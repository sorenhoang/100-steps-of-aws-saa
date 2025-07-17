# Step 59: Kinesis Streams & Firehose

## 🎯 Objective
- [x] Master: **Kinesis Streams & Firehose**

## 📘 Notes

### **Deep Dive: Kinesis Streams & Firehose**

The Amazon Kinesis family of services is designed for collecting, processing, and analyzing real-time streaming data at massive scale. The two foundational services are Kinesis Data Streams, which provides maximum flexibility and custom processing, and Kinesis Data Firehose, which provides maximum simplicity for loading data into specific destinations.

---

### **1. Amazon Kinesis Data Streams** 🌊

**What it is:** A massively scalable and durable real-time data streaming service. It is a lower-level, foundational service that gives you full control over how data is processed.

- **Core Concept (Shards):** A Kinesis Data Stream is composed of one or more **shards**. A shard is the base unit of throughput in a stream.
    - Each shard provides an ingest capacity of **1 MB/second or 1,000 records/second**.
    - Each shard provides an egress capacity of **2 MB/second**.
    - The total capacity of your stream is the sum of the capacity of its shards. You scale a Kinesis Data Stream by adding or removing shards.
- **Producers & Consumers:**
    - **Producers** (e.g., IoT devices, web servers, mobile apps) put records into the stream.
    - **Consumers** (e.g., custom applications on EC2, AWS Lambda functions, Kinesis Data Analytics) read and process these records from the shards.
- **Data Retention:** Data records are stored in the stream for a configurable retention period, from **24 hours (default) up to 365 days**. Multiple consumers can read the same data from the stream independently during this period.
- **Management:** **You are responsible for managing capacity** by provisioning the correct number of shards.
- **Ordering:** Data ordering is preserved *within* a single shard. Records sent with the same partition key will go to the same shard and be processed in order.
- **Use Case:** When you need **custom, real-time processing** of streaming data.
    - Real-time analytics dashboards.
    - Complex event processing (e.g., real-time fraud detection).
    - Feeding data to multiple independent applications that need to process the same stream.

---

### **2. Amazon Kinesis Data Firehose** 🔥

**What it is:** A fully managed service for **delivering** real-time streaming data to specific destinations. It is the simplest way to load streaming data into data stores and analytics tools.

- **Core Concept (Delivery Stream):** You create a "delivery stream." Producers send data to it, and Firehose handles everything else automatically.
- **Fully Managed / Serverless:** There are **no shards to manage**. Firehose automatically scales to match the throughput of your data source. You don't provision any capacity.
- **How it works:**
    1. Producers send data to the Firehose delivery stream.
    2. Firehose **buffers** the incoming data based on time (e.g., every 60 seconds) or size (e.g., every 5 MB).
    3. **(Optional)** Firehose can perform in-flight transformations on the data, such as converting JSON to Parquet or compressing it. This is often done using a Lambda function.
    4. Firehose delivers the batched, transformed data to a configured destination.
- **Destinations:**
    - Amazon S3 (most common)
    - Amazon Redshift (data warehouse)
    - Amazon OpenSearch Service (formerly Elasticsearch Service)
    - Splunk and other third-party services.
- **Data Retention:** Firehose **does not have data retention**. It is a direct delivery service. Once the data is delivered to the destination, it is no longer stored in Firehose.
- **Use Case:** When you need a simple, automated way to **load streaming data** into a destination with minimal processing.
    - Archiving log data from thousands of servers directly to S3.
    - Ingesting IoT sensor data into a data lake in S3.
    - Loading clickstream data into Amazon Redshift for analysis.

---

### **Summary of Key Differences**

| Feature            | Kinesis Data Streams            | Kinesis Data Firehose                            |
| ------------------ | ------------------------------- | ------------------------------------------------ |
| **Management**     | **Manual (You manage shards)**  | **Fully Managed / Serverless**                   |
| **Data Retention** | **Yes (24 hours to 365 days)**  | No (Direct delivery)                             |
| **Consumers**      | Custom Apps (EC2, Lambda)       | Pre-configured destinations (S3, Redshift, etc.) |
| **Real-time?**     | **Yes (sub-second latency)**    | Near real-time (lowest buffer is 60s)            |
| **Primary Use**    | **Custom real-time processing** | **Simple data loading/ingestion**                |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to ingest streaming clickstream data from its website and load it into an Amazon S3 data lake for archival and future analysis. The company wants the simplest possible solution with no capacity management overhead. Which AWS service should they use?**

- A. Amazon Kinesis Data Streams with a Lambda function consumer to write to S3.
- B. Amazon Kinesis Data Firehose with S3 as the destination.
- C. Amazon SQS to queue the data and an EC2 instance to write to S3.
- D. Amazon aMSK (Managed Streaming for Kafka).
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the primary use case for Kinesis Data Firehose. It is a fully managed, serverless service designed to easily load streaming data into destinations like S3. You simply create a delivery stream, point it to your S3 bucket, and Firehose handles all the scaling, buffering, and delivery automatically.
</details>
    

---

**2. A financial services application needs to process a stream of stock market data in real-time (with sub-second latency) to detect anomalies. The processing logic is complex and will be handled by a custom application running on EC2 instances. Which service provides the necessary capabilities?**

- A. Kinesis Data Firehose
- B. Kinesis Data Streams
- C. Amazon SQS FIFO Queue
- D. AWS Step Functions
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The key requirements are real-time processing and custom consumers. Kinesis Data Streams is designed for this. It allows custom applications (on EC2, Lambda, etc.) to pull data from the stream with very low latency and perform complex processing. Firehose (A) is for simple loading, not custom real-time processing.
</details>
    

---

**3. What is the fundamental unit of throughput that you must provision and manage in a Kinesis Data Stream?**

- A. A Consumer
- B. A Shard
- C. A Partition Key
- D. A Delivery Stream
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Shard is the base unit of capacity for a Kinesis Data Stream. Each shard provides a fixed amount of ingest and egress bandwidth. To scale the stream up or down, you must increase or decrease the number of shards.
</details>
    

---

**4. Which Kinesis service is fully managed and does NOT require you to provision or manage any shards?**

- A. Kinesis Data Streams
- B. Kinesis Video Streams
- C. Kinesis Data Analytics
- D. Kinesis Data Firehose
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** Kinesis Data Firehose is a serverless offering. It automatically scales its capacity up and down to match the throughput of the incoming data. You do not need to manage shards or any other capacity units.
</details>
    

---

**5. A company wants to use Kinesis Data Firehose to deliver log files to Amazon S3. They need to compress the data and convert its format from JSON to Apache Parquet before it is stored in S3 to optimize costs and query performance. How can this be achieved?**

- A. This is not possible; Firehose can only deliver raw data.
- B. By configuring an in-flight data transformation using a specified AWS Lambda function.
- C. By writing a custom consumer application for the Firehose stream.
- D. By enabling the "Convert to Parquet" option directly in the Firehose console.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Kinesis Data Firehose has a built-in feature for data format conversion and transformation. It can invoke an AWS Lambda function to process each batch of records before delivering them to the destination. This Lambda function can perform tasks like data enrichment, filtering, or format conversion (like JSON to Parquet).
</details>
    

---

**6. What is the default data retention period for a Kinesis Data Stream?**

- A. 1 hour
- B. 24 hours
- C. 7 days
- D. 14 days
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** By default, data records are retained in a Kinesis Data Stream for 24 hours. You can configure this retention period to be extended up to 365 days.
</details>
    

---

**7. A company has a Kinesis Data Stream with two different consumer applications that need to process the same stream of data independently and at their own pace. Is this possible?**

- A. No, a stream can only have one consumer application.
- B. No, the first application to read a record will consume it, making it unavailable to the second.
- C. Yes, multiple consumer applications can read from the same stream independently.
- D. Yes, but only if they are both Lambda functions.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A key feature of Kinesis Data Streams is that data is retained in the stream for the configured retention period. This allows multiple consumer applications to read and process the entire stream of data in parallel without interfering with each other. Each consumer maintains its own pointer to its position in the stream.
</details>
    

---

**8. Which Kinesis service buffers data and delivers it to destinations in batches based on time or size?**

- A. Kinesis Data Streams
- B. Kinesis Data Firehose
- C. Kinesis Video Streams
- D. Amazon SQS
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Kinesis Data Firehose does not deliver records one by one. It buffers incoming records and then delivers them as a single batch (or object, in the case of S3) once either a time limit (e.g., 60 seconds) or a size limit (e.g., 5 MB) is reached.
</details>
    

---

**9. What happens to data in a Kinesis Data Firehose stream after it has been successfully delivered to its destination?**

- A. It is retained for 24 hours.
- B. It is archived in S3 Glacier.
- C. It is deleted from the Firehose stream.
- D. It is sent to a backup Kinesis Data Stream.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Kinesis Data Firehose is a delivery service, not a storage service. It does not have a data retention feature. Once it successfully delivers a batch of data to the configured destination, that data is no longer stored within Firehose.
</details>
    

---

**10. When sending data to a Kinesis Data Stream, what is the purpose of the "Partition Key"?**

- A. To encrypt the data record.
- B. To determine which shard the data record will be sent to.
- C. To define the consumer of the record.
- D. To act as a unique ID for the record itself.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Kinesis Data Streams uses the Partition Key to distribute data among the available shards. It feeds the partition key into a hash function, and the output of the hash determines the shard. All records with the same partition key will always go to the same shard, which is how ordering is guaranteed for a given key.
</details>
    

---

**11. An application is experiencing throttling errors when writing to a Kinesis Data Stream. What is the most likely cause?**

- A. The IAM role for the producer is missing permissions.
- B. The stream has an insufficient number of shards to handle the incoming data rate.
- C. The consumer application is not processing records fast enough.
- D. The data retention period is set too low.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Throttling errors (ProvisionedThroughputExceededException) occur when you exceed the capacity of the stream. Since the capacity of a Kinesis Data Stream is determined by the number of shards, the solution is to increase the number of shards in the stream to provide more ingest capacity.
</details>
    

---

**12. Which statement accurately describes the difference in consumer types between the two services?**

- A. Kinesis Data Streams consumers must be Lambda functions, while Firehose consumers can be EC2 instances.
- B. Kinesis Data Streams has custom consumers (EC2, Lambda), while Firehose delivers to pre-configured AWS service destinations (S3, Redshift, etc.).
- C. Both services can only be consumed by Kinesis Data Analytics.
- D. Kinesis Data Firehose has custom consumers, while Kinesis Data Streams has pre-configured destinations.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is a core distinction. With Kinesis Data Streams, you build your own consumer application to process the data. With Kinesis Data Firehose, you don't build a consumer; you simply configure one of the supported AWS services as the destination, and Firehose handles the delivery.
</details>
    

---

**13. A company wants to ingest IoT data. They need to perform simple transformations, like converting the data format from JSON to Parquet, and then deliver it to an S3 bucket. Which service is the most direct and managed solution?**

- A. Kinesis Data Streams
- B. A custom application on EC2.
- C. Kinesis Data Firehose
- D. Amazon SQS
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Kinesis Data Firehose has built-in capabilities for data format conversion. It can transparently convert incoming JSON data to optimized columnar formats like Apache Parquet or ORC before delivering it to S3. This is a simple, managed feature that doesn't require writing custom code.
</details>
    

---

**14. What is required to ensure that all records for a specific user are processed in order by a Kinesis Data Stream?**

- A. Use a FIFO SQS queue as a buffer.
- B. Use the same `PartitionKey` (e.g., the `UserID`) for all of that user's records.
- C. Increase the number of shards in the stream.
- D. Enable Kinesis Enhanced Fan-Out.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Kinesis guarantees order within a shard. To ensure all records for a particular entity (like a user) are processed sequentially, you must ensure they all land on the same shard. This is achieved by consistently using the same PartitionKey for all records related to that entity.
</details>
    

---

**15. Your consumer application processing a Kinesis Data Stream needs to read data faster. Each shard provides 2 MB/s of egress. How can you increase the total read throughput?**

- A. You can't; read throughput is fixed.
- B. By increasing the number of shards in the stream.
- C. By enabling Kinesis Enhanced Fan-Out.
- D. Both B and C are valid methods.
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** You can scale read throughput in two ways. The traditional way is to increase the number of shards (B), as the total read capacity is the number of shards multiplied by 2 MB/s. A newer and more powerful way is to use Enhanced Fan-Out (C). This feature provides each registered consumer with its own dedicated 2 MB/s read throughput per shard, allowing multiple consumers to read at full speed without competing with each other.
</details>
    

---

**16. A producer sends a 50 KB record to a Kinesis Data Stream. How many write requests is this counted as for billing and throughput purposes?**

- A. 50, as it's based on KB.
- B. 2, because the max size is 25 KB.
- C. 1, as a record is a single request.
- D. It depends on the number of consumers.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Throughput for Kinesis Data Streams is measured in both records per second (1000/shard) AND kilobytes per second (1000 KB/shard). A single PutRecord API call, regardless of its size (up to the 1 MB limit), counts as one record. So this would be 1 out of the 1000 records/sec limit.
</details>
    

---

**17. Which service is better suited for a use case where you need multiple, independent teams to analyze the same real-time data stream for different purposes (e.g., one for fraud detection, one for live dashboards)?**

- A. Kinesis Data Firehose
- B. Kinesis Data Streams
- C. SQS
- D. SNS
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Kinesis Data Streams is ideal for this because it retains data for at least 24 hours. This allows multiple, independent consumer applications to connect to the stream and read the same data at their own pace without interfering with each other. Firehose only delivers to one configured set of destinations.
</details>
    

---

**18. What is the minimum buffer time for a Kinesis Data Firehose delivery stream?**

- A. 1 second
- B. 30 seconds
- C. 60 seconds
- D. 300 seconds
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Kinesis Data Firehose is a "near real-time" service because it buffers data. The minimum configurable buffer interval is 60 seconds. This means even in the best case, there will be a latency of at least one minute before data is delivered.
</details>
    

---

**19. A Kinesis Data Firehose stream is configured to deliver data to Amazon Redshift. Where does Firehose stage the data before loading it into Redshift?**

- A. In an EBS volume.
- B. In an EC2 instance's memory.
- C. In an Amazon S3 bucket.
- D. Directly into Redshift's memory.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** When using Amazon Redshift as a destination, Firehose first delivers the incoming data to an S3 bucket that you specify. It then issues a COPY command to Redshift to load the data from that S3 bucket into the target table. S3 acts as an intermediate staging area.
</details>
    

---

**20. You want to analyze the data in your Kinesis Data Stream in real time using standard SQL queries. Which service provides this capability?**

- A. Kinesis Data Firehose
- B. Kinesis Data Analytics
- C. Amazon Redshift
- D. Amazon RDS
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Kinesis Data Analytics is the service designed for real-time analysis of streaming data. It can ingest data directly from a Kinesis Data Stream (or Firehose) and allows you to write SQL queries that run continuously over the data as it flows through the stream.
</details>
    

---

**21. A Kinesis Data Stream has 10 shards. What is its total data ingest capacity?**

- A. 10 MB/s and 10,000 records/s
- B. 20 MB/s and 20,000 records/s
- C. 1 MB/s and 1,000 records/s
- D. The capacity is unlimited.
<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** Each shard provides 1 MB/s and 1,000 records/s of ingest capacity. Therefore, a stream with 10 shards has a total capacity of 10 * 1 MB/s = 10 MB/s AND 10 * 1,000 records/s = 10,000 records/s. You are limited by whichever of these two limits you hit first.
</details>
    

---

**22. Which service would you choose if your primary goal is to simply and reliably collect streaming log data and archive it in S3?**

- A. Kinesis Data Streams
- B. AWS Glue
- C. Kinesis Data Firehose
- D. Amazon SQS
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is the canonical use case for Kinesis Data Firehose. It provides the simplest, most direct, and fully managed path to ingest streaming data and load it into an S3 bucket for archival.
</details>
