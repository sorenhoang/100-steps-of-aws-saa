# Step 56: SQS – Standard vs FIFO

## 🎯 Objective

- [x] Master: **SQS – Standard vs FIFO**

## 📘 Notes

### **Deep Dive: SQS – Standard vs. FIFO**

Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications. Instead of components calling each other directly, they communicate by sending messages to an SQS queue. This creates a buffer that increases fault tolerance and reliability.

---

### **1. Core SQS Concepts**

- **Queue:** A temporary repository for messages that are awaiting processing.
- **Message:** A packet of data sent to the queue by a producer. A message can be up to 256 KB of text in any format.
- **Producer:** The application component that sends messages to the queue.
- **Consumer:** The application component that retrieves and processes messages from the queue (e.g., an EC2 instance or a Lambda function).
- **Visibility Timeout:** This is a crucial concept. When a consumer retrieves a message, it doesn't disappear from the queue. Instead, it becomes "invisible" for a period of time defined by the visibility timeout. This prevents other consumers from processing the same message. If the consumer processes the message successfully, it must **delete** the message from the queue. If it fails and the visibility timeout expires, the message becomes visible again for another consumer to process.
- **Dead-Letter Queue (DLQ):** A separate SQS queue that acts as a destination for messages that have failed processing a specified number of times. This prevents "poison pill" messages (messages that always cause an error) from being retried indefinitely and allows developers to inspect failed messages.

---

### **2. SQS Queue Types**

SQS offers two types of queues with very different behaviors.

- **Standard Queue:**
  - **This is the default queue type.**
  - **Ordering:** **Best-Effort Ordering**. It generally delivers messages in roughly the same order they are sent, but it makes **no guarantee** about the order. A message might be delivered out of order.
  - **Delivery:** **At-Least-Once Delivery**. A message is delivered at least once, but occasionally, due to the highly distributed architecture, more than one copy of a message might be delivered. Your application must be **idempotent** (safe to process the same message multiple times).
  - **Throughput:** **Nearly unlimited throughput**. It is designed for maximum scalability and can handle a massive number of messages per second.
  - **Use Case:** The best choice for most scenarios where high throughput is important and occasional out-of-order delivery or message duplication can be tolerated. Examples include decoupling microservices, batch processing, and task distribution.
- **FIFO (First-In, First-Out) Queue:**
  - **Purpose:** Designed for applications where the **order of operations and events is critical**.
  - **Ordering:** **Strict Ordering**. The order in which messages are sent is the exact order in which they are received.
  - **Delivery:** **Exactly-Once Processing**. As long as the message is successfully processed and deleted, SQS ensures it will not be delivered again. It also prevents duplicates from being introduced into the queue.
  - **Throughput:** Lower throughput than Standard Queues (by default, up to 3,000 messages per second with batching, or 300 without).
  - **Message Groups:** FIFO queues use a `MessageGroupId` to group related messages. All messages with the same group ID will be processed in order, one after another. Different message groups can be processed in parallel.
  - **Use Case:** Use cases where order is essential, such as financial transactions (debits must happen before credits), user command processing in a chat app, or any workflow where the sequence of steps matters.

---

### **Summary of Key Differences**

| Feature        | Standard Queue              | FIFO Queue                           |
| -------------- | --------------------------- | ------------------------------------ |
| **Ordering**   | Best-Effort                 | **First-In, First-Out**              |
| **Delivery**   | At-Least-Once               | **Exactly-Once Processing**          |
| **Throughput** | **Nearly Unlimited**        | High, but limited                    |
| **Use Case**   | Decoupling, high throughput | **Order is critical**, no duplicates |
| **Duplicates** | Possible                    | **No duplicates**                    |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. An online banking application needs to process financial transactions. It is absolutely critical that debit operations are processed before their corresponding credit operations for each user account. Which SQS queue type should be used?**

- A. Standard Queue
- B. FIFO Queue
- C. Both can be configured to meet the requirement.
- D. This requires a relational database with transactions, not SQS.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The key requirement is the strict ordering of operations. A FIFO (First-In, First-Out) Queue is the only SQS type that guarantees the order of message processing. You would use the user's account ID as the MessageGroupId to ensure all transactions for a specific account are processed sequentially.

</details>

---

**2. A consumer application pulls a message from an SQS queue. The application begins processing the message, but it crashes before it can delete the message. The visibility timeout for the message is 60 seconds. What will happen to the message?**

- A. The message is permanently lost.
- B. The message is automatically moved to a Dead-Letter Queue (DLQ).
- C. After the 60-second visibility timeout expires, the message will become visible in the queue again for another consumer to process.
- D. SQS will automatically retry the message on the same consumer.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This describes the purpose of the visibility timeout. It's a safety mechanism. When a consumer fails to delete a message within the timeout period, SQS assumes something went wrong and makes the message visible again so it can be re-processed, ensuring the message is not lost.

</details>

---

**3. A company is designing a system to collect and process votes from a popular TV show. The system needs to handle millions of votes per minute, but the exact order in which individual votes are counted is not critical. Which SQS queue type is the most appropriate?**

- A. FIFO Queue, to ensure fairness.
- B. Standard Queue, because it offers the highest possible throughput.
- C. A Dead-Letter Queue.
- D. A FIFO Queue with a high batch size.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The primary requirement is high throughput (millions of votes per minute). A Standard Queue is designed for nearly unlimited throughput and is the best choice for this use case. Since the exact order of votes isn't critical, the best-effort ordering of a Standard Queue is perfectly acceptable.

</details>

---

**4. What is the main purpose of configuring a Dead-Letter Queue (DLQ) for an SQS queue?**

- A. To store a backup of all messages that are successfully processed.
- B. To act as an overflow queue when the main queue is full.
- C. To isolate and store messages that have failed processing multiple times, so they can be analyzed later.
- D. To increase the throughput of the main queue.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Dead-Letter Queue is specifically designed to handle problematic messages. After a message has been received and returned to the queue a configured number of times (maxReceiveCount), it's automatically moved to the DLQ. This prevents poison messages from blocking the main queue and allows developers to investigate what went wrong.

</details>

---

**5. Which statement accurately describes the delivery guarantee of an SQS Standard Queue?**

- A. Exactly-once delivery.
- B. At-most-once delivery.
- C. At-least-once delivery.
- D. Ordered delivery.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A Standard Queue provides at-least-once delivery. This means every message will be delivered, but on rare occasions (due to the distributed nature of the service), a message might be delivered more than once. Applications using Standard Queues should be idempotent.

</details>

---

**6. What is the maximum size of a message that can be sent to an SQS queue?**

- A. 64 KB
- B. 256 KB
- C. 1 MB
- D. 2 MB

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The maximum payload size for a single SQS message is 256 KB. For messages larger than this, the common pattern is to store the large payload in S3 and send a message to SQS containing a pointer (the S3 object key) to that payload.

</details>

---

**7. In a FIFO queue, what is the role of the `MessageGroupId`?**

- A. It ensures the message is encrypted.
- B. It acts as a unique identifier for the message itself.
- C. It groups messages together. All messages with the same `MessageGroupId` will be processed in strict order.
- D. It determines the priority of the message.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The MessageGroupId is essential for enabling parallel processing while maintaining order in a FIFO queue. All messages sharing a group ID form a distinct, ordered "lane." SQS will process messages from different message groups concurrently, but messages within a single group are always processed one after another in the order they were sent.

</details>

---

**8. A consumer application retrieves a batch of 10 messages from an SQS queue. It successfully processes 9 messages but fails on the 10th message and does not delete any of them. What happens after the visibility timeout expires?**

- A. All 10 messages become visible again in the queue.
- B. Only the 10th failed message becomes visible again.
- C. The 9 successful messages are deleted, and the 10th message becomes visible.
- D. All 10 messages are moved to the DLQ.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Message deletion is an explicit, individual action. If the consumer fails to delete any of the messages in the batch it received, then all messages from that batch will become visible again after the visibility timeout expires. The application is responsible for tracking which messages it successfully processed and only deleting those.

</details>

---

**9. What is the default visibility timeout for a message in a new SQS queue?**

- A. 0 seconds
- B. 30 seconds
- C. 5 minutes
- D. 12 hours

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The default visibility timeout is 30 seconds. This is an important value to know. You should adjust it to be longer than the typical time it takes your consumer to process a message.

</details>

---

**10. Which SQS queue type supports a much higher number of transactions per second (TPS) by default?**

- A. FIFO Queue
- B. Standard Queue
- C. Both have the same throughput limits.
- D. Throughput depends on the message size, not the queue type.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Standard Queues are designed for maximum throughput and offer nearly unlimited TPS. FIFO queues, because they have the extra overhead of managing ordering and deduplication, have a lower, though still high, throughput limit.

</details>

---

**11. An application needs to send a message to multiple different consumer applications simultaneously. For example, when a new user signs up, the billing service, the analytics service, and the welcome email service all need to be notified. What is the best architectural pattern for this?**

- A. Send the message to a single SQS queue that all services poll.
- B. Use an Amazon SNS topic to publish the message, with multiple SQS queues (one for each service) subscribed to the topic.
- C. Create three separate SQS queues and have the producer send the same message to all three.
- D. Use a single SQS FIFO queue with a different `MessageGroupId` for each service.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic fan-out pattern. By publishing a single message to an SNS topic, you can have multiple SQS queues subscribe to that topic. SNS will then deliver a copy of the message to each subscribed queue, allowing each downstream service to consume the message independently and at its own pace.

</details>

---

**12. What does it mean for a consumer application to be "idempotent"?**

- A. It processes messages very quickly.
- B. It can process the same message multiple times with no adverse side effects.
- C. It can only process one message at a time.
- D. It automatically deletes messages after processing.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Idempotency is a critical concept when working with Standard SQS queues, which have at-least-once delivery. An idempotent process is one that can be safely run over and over again. For example, an operation to set a user's status to processed is idempotent; running it 5 times has the same result as running it once. An operation to add $10 to a user's balance is not idempotent.

</details>

---

**13. What is SQS short polling versus long polling?**

- A. Short polling queries all SQS servers, while long polling only queries one.
- B. Short polling returns a response immediately, even if the queue is empty, while long polling waits for a specified time for a message to arrive before returning.
- C. Long polling is more expensive than short polling.
- D. Short polling is for FIFO queues, and long polling is for Standard queues.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Long polling is the preferred and more efficient method. When a consumer requests messages, a long poll will "wait" on the connection for up to a specified duration (e.g., 20 seconds) if the queue is empty. This reduces the number of empty responses and API calls, lowering cost and improving efficiency. Short polling returns immediately, leading to many empty responses if the queue is not busy.

</details>

---

**14. What is the maximum retention period for a message in an SQS queue?**

- A. 24 hours
- B. 7 days
- C. 14 days
- D. 30 days

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** You can configure a message retention period for an SQS queue from 60 seconds up to a maximum of 14 days. After this period, the message will be automatically deleted from the queue if it hasn't already been processed and deleted.

</details>

---

**15. What feature of a FIFO queue prevents a producer from sending an exact duplicate of a message within a 5-minute window?**

- A. Visibility Timeout
- B. Content-based deduplication
- C. Message Timers
- D. A Dead-Letter Queue

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** FIFO queues have a built-in deduplication mechanism. You can either provide a unique MessageDeduplicationId or enable content-based deduplication, where SQS calculates a hash of the message body. If a message arrives with the same deduplication ID or content hash as one sent within the 5-minute deduplication interval, the new message is accepted but not delivered, guaranteeing exactly-once processing.

</details>

---

**16. How can a consumer process messages from an SQS queue in parallel?**

- A. This is not possible; SQS is single-threaded.
- B. By using a FIFO queue with multiple `MessageGroupId`s.
- C. By having multiple consumer instances (e.g., EC2 or Lambda) all polling the same queue.
- D. Both B and C are valid methods.

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Parallel processing is a key benefit of SQS. You can have many consumers polling the same Standard queue concurrently (C). For FIFO queues, you achieve parallelism by using multiple different MessageGroupIds, as SQS can process messages from different groups concurrently (B).

</details>

---

**17. What is the purpose of the visibility timeout?**

- A. To define how long a message can stay in the queue before being deleted.
- B. To hide a message for a period of time after it has been received by a consumer, to prevent other consumers from processing it simultaneously.
- C. To set a delay before a new message becomes visible in the queue.
- D. To control the polling frequency of consumers.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The visibility timeout is the core locking mechanism in SQS. It makes a message "invisible" to other consumers once it has been picked up, giving the first consumer a specific amount of time to process and delete the message without interference.

</details>

---

**18. You are designing an application that requires a message queue. The tasks can be processed independently, and maximizing throughput is the highest priority. Which queue type should you choose?**

- A. FIFO Queue
- B. Standard Queue
- C. An SNS Topic
- D. A Kinesis Data Stream

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When maximum throughput is the goal and strict ordering is not a requirement, a Standard Queue is always the correct choice. It is designed to scale to handle an extremely high volume of messages per second.

</details>

---

**19. What is the `ReceiveMessageWaitTimeSeconds` parameter used for in SQS?**

- A. To set the visibility timeout for messages.
- B. To configure a message delay timer.
- C. To enable and configure long polling for a queue.
- D. To set the message retention period.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Setting the ReceiveMessageWaitTimeSeconds parameter on a queue to a value greater than 0 (up to 20 seconds) enables long polling. This tells the ReceiveMessage API call to wait for that duration for messages to arrive before returning an empty response.

</details>

---

**20. A message that consistently causes a consumer to error out is sometimes referred to as a:**

- A. Poison Pill message
- B. Zombie message
- C. Orphan message
- D. Dead-Letter message

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** A "poison pill" is a common term for a message that is malformed or contains data that will always cause the processing logic to fail. Dead-Letter Queues (DLQs) are the mechanism used to handle these poison pill messages.

</details>

---

**21. A consumer needs more time to process a specific message it has already received. The initial visibility timeout is about to expire. How can the consumer prevent the message from becoming visible to other consumers?**

- A. By deleting the message and sending it back to the queue.
- B. By calling the `ChangeMessageVisibility` API action to extend the timeout for that specific message.
- C. This is not possible; the consumer must finish within the initial timeout.
- D. By increasing the default visibility timeout on the entire queue.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The consumer can call the ChangeMessageVisibility API for a message it is currently processing. This allows it to extend the timeout on a case-by-case basis, giving it more time to complete the work without another consumer inadvertently picking up the same message.

</details>

---

**22. Which two features are exclusive to FIFO queues? (Select TWO)**

- A. Message ordering
- B. Dead-Letter Queues
- C. Content-based deduplication
- D. Visibility timeouts
- E. Long polling

<details>
<summary>View Answer</summary>

**Answer: A and C**

**Explanation:** The defining features of FIFO queues are guaranteed message ordering (A) and exactly-once processing, which is achieved through a deduplication mechanism (C). Dead-Letter Queues, visibility timeouts, and long polling are available for both Standard and FIFO queues.

</details>
