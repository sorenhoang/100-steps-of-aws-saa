# Step 57: SNS Topics & Fan‑out Pattern

## 🎯 Objective
- [x] Master: **SNS Topics & Fan‑out Pattern**

## 📘 Notes

### **Deep Dive: SNS Topics & Fan-out Pattern**

Amazon Simple Notification Service (SNS) is a fully managed **publish/subscribe (pub/sub)** messaging service. It allows you to send messages to a large number of subscribers simultaneously. Its primary purpose is to decouple applications, sending high-throughput, push-based notifications to many different endpoints.

-----

#### **1. Core SNS Concepts**

  * **Topic:** A topic is a logical access point and a communication channel. You create a topic for a specific subject (e.g., `new-orders`, `user-signups`). Producers send messages to topics.
  * **Publisher (Producer):** An application or service that sends messages to an SNS topic. Publishers don't need to know who is listening for the messages; they only care about the topic.
  * **Subscriber (Consumer):** An endpoint that receives messages from an SNS topic. When a message is published to a topic, SNS immediately pushes a copy of that message to all of its subscribers.
  * **Subscription:** The configuration that connects a subscriber to a topic.

#### **2. The Fan-out Pattern** 📢

This is the most important architectural pattern associated with SNS. The "fan-out" pattern is where a single message published to one SNS topic is replicated and pushed out to multiple subscriber endpoints to be processed in parallel.

  * **How it works:**

    1.  A **Producer** (e.g., an S3 bucket event, a Lambda function) publishes a single message to an **SNS Topic**.
    2.  The SNS Topic immediately sends a copy of that message to **all** of its subscribers.
    3.  The subscribers can be different types of services. For example, one SQS queue might handle order processing, another SQS queue might handle data archiving, and a Lambda function might handle sending a user notification. All three receive the same message and act on it independently and concurrently.

  * **Benefit:** This pattern allows you to build highly decoupled, parallel, and resilient systems. If the analytics service (one subscriber) fails, it doesn't impact the order processing service (another subscriber) because they are independent.

-----

#### **3. SNS Subscribers & Filtering**

SNS can send messages to a wide variety of subscribers:

  * **SQS Queues:** This is a very common pattern for durable, asynchronous processing.
  * **AWS Lambda functions:** For direct, serverless processing of notifications.
  * **HTTP/HTTPS endpoints:** To send messages to webhooks or your own web applications.
  * **Email (JSON or plain text):** For human-readable notifications.
  * **Mobile Push Notifications:** (e.g., APNS for Apple, FCM for Google).
  * **SMS text messages.**

**Subscription Filter Policies:**
You can configure a subscription so that it only receives a subset of the messages published to a topic. This is done by applying a **filter policy** (a simple JSON object) to the subscription.

  * **Example:** Imagine an `order-events` topic. You could have one SQS queue subscribe to all messages, but another "high-value-orders" Lambda function could have a filter policy like `{"order_total": [{"numeric": [">=", 1000]}]}`. This Lambda would only be invoked for messages where the `order_total` attribute is 1000 or more.

-----

### **AWS SAA-C03 Style Questions & Explanations**

**1. An e-commerce application needs to notify multiple downstream systems whenever a new order is placed. The order processing service needs the message, the analytics service needs the message, and a fraud detection service needs the message. All three services must receive the same message and process it independently. Which AWS service is best suited to facilitate this?**

  * A. Amazon SQS Standard Queue
  * B. Amazon SQS FIFO Queue
  * C. Amazon SNS Topic
  * D. Amazon Kinesis Data Streams

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a classic **fan-out** scenario. By publishing a single "new order" message to an **Amazon SNS Topic**, you can have multiple subscribers (like SQS queues for each service) receive a copy of that message. This decouples the services and allows for parallel processing. An SQS queue (A, B) only delivers a message to one consumer.
</details>

-----

**2. What is the architectural pattern where a single message from one source is delivered to multiple, different destination systems?**

  * A. Point-to-Point
  * B. Fan-out
  * C. Hub-and-Spoke
  * D. Client-Server

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The **fan-out** pattern describes a one-to-many messaging model. A single message is published once and then "fanned out" to multiple subscribers. SNS is the primary AWS service for implementing this pattern.
</details>

-----

**3. Which of the following can be a subscriber to an SNS topic? (Select TWO)**

  * A. An S3 bucket
  * B. An AWS Lambda function
  * C. An SQS queue
  * D. An EC2 instance directly
  * E. An EBS volume

<details>
<summary>View Answer</summary>
<br>

**Answers: B and C**

**Explanation:** SNS can push messages to a wide variety of endpoints. The most common in serverless architectures are **AWS Lambda functions** (for direct invocation) and **SQS queues** (for durable, asynchronous processing). While you can send a message to an email address that an EC2 instance checks, you cannot subscribe an instance directly.
</details>

-----

**4. A company wants to send promotional SMS text messages to its customers. Which service can be used to send these messages directly to mobile devices?**

  * A. Amazon SQS
  * B. Amazon SES (Simple Email Service)
  * C. Amazon SNS
  * D. Amazon Pinpoint

<details>
<summary>View Answer</summary>
<br>

**Answer: C or D**

**Explanation:** While **Amazon Pinpoint (D)** is the more advanced service for customer engagement campaigns, **Amazon SNS (C)** provides the fundamental capability to publish messages directly as SMS text messages to phone numbers. For the SAA-C03 exam, SNS is the direct answer for pub/sub messaging to SMS endpoints.
</details>

-----

**5. How is a message from an SNS topic delivered to a subscribed SQS queue?**

  * A. The SQS queue consumer must poll the SNS topic for new messages.
  * B. SNS **pushes** the message directly into the SQS queue.
  * C. A Lambda function is required to move messages from SNS to SQS.
  * D. SNS sends the message to an S3 bucket, which then triggers the SQS queue.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** SNS is a **push-based** service. When a message is published to a topic, the SNS service itself actively **pushes** a copy of that message to all its subscribers, including placing it directly into any subscribed SQS queues. The consumer then polls the SQS queue, not the SNS topic.
</details>

-----

**6. A company has an SNS topic for all "vehicle-events." They have a Lambda function that should ONLY be invoked for events related to "trucks." How can this be achieved without changing the publisher's code?**

  * A. Code the Lambda function to check the message content and exit if the vehicle is not a truck.
  * B. Create a separate SNS topic just for truck events.
  * C. Apply an SNS subscription filter policy to the Lambda subscription.
  * D. Use an SQS FIFO queue to filter the messages.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A **subscription filter policy** is the perfect solution for this. You can add a policy to the specific Lambda subscription that filters messages based on their attributes (e.g., `{"vehicle_type": ["truck"]}`). This way, SNS only invokes the function when a message with that attribute is published, saving costs and simplifying the Lambda code.
</details>

-----

**7. What is a key difference between SNS and SQS?**

  * A. SNS is push-based (pub/sub), while SQS is pull-based (polling).
  * B. SNS guarantees the order of messages, while SQS does not.
  * C. SNS stores messages for up to 14 days, while SQS does not store messages.
  * D. SNS can only have one subscriber, while SQS can have many consumers.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** This is the fundamental difference in their models. **SNS is a push-based pub/sub system**; it broadcasts messages to all subscribers. **SQS is a queue that is pull-based**; consumers must poll the queue to retrieve messages. SNS does not store messages long-term (it's a transient push), whereas SQS does.
</details>

-----

**8. An SNS message is published with message attributes. A subscription has a filter policy. What happens if the message attributes do not match the filter policy?**

  * A. The message is sent to a Dead-Letter Queue.
  * B. The message is rejected by the SNS topic and the publisher receives an error.
  * C. The message is held in the topic for 24 hours to see if the filter policy changes.
  * D. The message is simply not delivered to that specific subscriber.

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** The filter policy works at the subscription level. If a message's attributes do not match the filter criteria for a particular subscription, SNS will simply discard the message for that subscriber. Other subscribers on the same topic without a filter (or with a matching filter) will still receive the message.
</details>

-----

**9. In what format are messages sent from an SNS topic?**

  * A. Always XML.
  * B. It is a key-value store with no defined format.
  * C. A JSON document that contains message metadata and the message body.
  * D. It depends on the subscriber type.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** While the message body itself can be any string, the overall message delivered by SNS is a **JSON object**. This JSON object acts as a wrapper containing the `Message` payload itself, a `MessageId`, the `TopicArn`, a `Timestamp`, and any `MessageAttributes`.
</details>

-----

**10. A company needs to ensure that messages sent to an SNS topic are encrypted both in transit and at rest. How can this be achieved?**

  * A. This is the default behavior and requires no configuration.
  * B. By enabling Server-Side Encryption (SSE) on the SNS topic using a KMS key.
  * C. By using an SQS FIFO queue as the subscriber.
  * D. By publishing messages over an HTTPS endpoint, which handles both.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To encrypt messages **at rest** within the SNS topic, you must explicitly enable **Server-Side Encryption (SSE)** in the topic's configuration, which uses AWS KMS. Encryption in transit is achieved by publishing messages to the SNS topic's HTTPS endpoint.
</details>

-----

**11. Which of the following is NOT a valid subscriber for an SNS topic?**

  * A. An SQS queue.
  * B. A mobile phone number (for SMS).
  * C. An email address.
  * D. An Amazon DynamoDB table.

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** You cannot subscribe a **DynamoDB table directly** to an SNS topic. To get data from an SNS notification into DynamoDB, you would use a Lambda function as the subscriber. The Lambda function would then receive the message and write the data to the DynamoDB table.
</details>

-----

**12. When you subscribe an HTTP/HTTPS endpoint to an SNS topic, what must the endpoint do to confirm the subscription?**

  * A. It must respond to the confirmation message with a `200 OK` status code.
  * B. It must visit a confirmation URL contained in the body of the subscription confirmation message.
  * C. It must publish a confirmation message back to the same SNS topic.
  * D. The subscription is confirmed automatically.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To prevent malicious subscriptions, SNS uses a confirmation step. It sends a special message to the HTTP/HTTPS endpoint containing a `SubscribeURL`. The owner of the endpoint must make an HTTP GET request to this unique URL to prove they have control of the endpoint and want to receive messages.
</details>

-----

**13. What is a "Topic Policy" in SNS?**

  * A. A policy that filters which messages a subscriber receives.
  * B. An IAM policy attached to a user that allows them to publish to a topic.
  * C. A resource-based policy attached to the SNS topic itself that defines who has permission to publish or subscribe to it.
  * D. A policy that defines the encryption settings for the topic.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A **Topic Policy** is a resource-based IAM policy. It is the primary way to control access to the topic, especially for cross-account access. For example, you can create a topic policy that allows an S3 bucket in another AWS account to publish event notifications to your topic.
</details>

-----

**14. A single SNS message about a new order is published. One subscriber is an SQS queue for processing, and the other is a Lambda function for logging. The SQS consumer application is down, but the Lambda function is running. What happens?**

  * A. The message delivery fails for both subscribers because the SQS queue is unavailable.
  * B. The Lambda function receives and processes the message successfully. The message remains in the SQS queue until its consumer comes online to process it.
  * C. The SNS topic retries sending to the SQS queue until it succeeds.
  * D. The message is delivered to the Lambda function and then deleted from the topic.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This demonstrates the decoupling provided by SNS and SQS. The two subscriptions are independent. The successful delivery to the Lambda function has no bearing on the delivery to the SQS queue. SNS successfully pushes the message to the queue, where it will be stored durably until the SQS consumer application is available to poll and process it.
</details>

-----

**15. To send a message to a single, specific SQS queue, which service should a producer use?**

  * A. SNS, by creating a topic with only one SQS subscriber.
  * B. SQS, by sending the message directly to the queue URL.
  * C. Kinesis, as it provides a direct integration.
  * D. EventBridge, with a rule that targets the queue.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** If you only need to send a message to one destination queue (a point-to-point pattern), there is no need to add the extra component of SNS. The simplest and most direct approach is for the producer to send the message **directly to the SQS queue**. You only introduce SNS when you need to fan-out to *multiple* destinations.
</details>

-----

**16. What are "Message Attributes" in an SNS message?**

  * A. The main body of the message.
  * B. A set of key-value pairs that provide structured metadata about the message.
  * C. The encryption key for the message.
  * D. A list of all subscribers for the topic.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** **Message Attributes** are separate from the message body. They are key-value pairs used to provide metadata that a subscriber can use for filtering or routing without having to parse the main message body. This is what subscription filter policies use to make decisions.
</details>

-----

**17. Can you have an SQS FIFO queue subscribe to an SNS Standard topic?**

  * A. No, FIFO queues can only subscribe to FIFO topics.
  * B. Yes, but the ordering and deduplication will be lost.
  * C. Yes, and SNS will automatically ensure messages are sent in order.
  * D. No, SQS queues cannot subscribe to SNS topics.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** To preserve the strict ordering and deduplication guarantees of FIFO, an **SQS FIFO queue** can only subscribe to an **SNS FIFO topic**. You cannot mix and match standard and FIFO types in this way.
</details>

-----

**18. What is the benefit of using SNS and SQS together for decoupling?**

  * A. It makes the application run faster.
  * B. It provides durability and resilience. If a subscribing service is down, its SQS queue will hold the messages until the service recovers.
  * C. It reduces the cost of sending messages.
  * D. It guarantees the order of message processing.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The combination provides the best of both worlds. SNS provides the fan-out capability, and SQS provides the **durability**. If a consumer application (e.g., a Lambda function or EC2 instance) is down, its dedicated SQS queue will reliably store the messages pushed by SNS until the application is healthy again and can process them.
</details>

-----

**19. A message published to an SNS topic fails to be delivered to one of its subscribed HTTP endpoints because the endpoint is down. What is the default SNS behavior?**

  * A. SNS will retry delivery to the failed endpoint multiple times with a backoff policy.
  * B. The message is immediately discarded for that subscriber.
  * C. The message is sent to a Dead-Letter Queue.
  * D. All other deliveries to healthy subscribers are halted until the failed one succeeds.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** SNS has a built-in **retry policy** for HTTP/HTTPS endpoints. If a delivery attempt fails (e.g., due to a 5xx server error or a connection timeout), SNS will automatically retry sending the message several times over a period of minutes or hours before finally giving up.
</details>

-----

**20. You have an SNS topic with 5 SQS queues subscribed to it. If you publish one message to the topic, how many messages are sent and how many are received in total?**

  * A. 1 message sent, 1 message received.
  * B. 5 messages sent, 5 messages received.
  * C. 1 message sent, 5 messages received.
  * D. 5 messages sent, 1 message received.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The publisher performs only **one** `Publish` action to the SNS topic. The SNS service then creates a copy of that message for each subscription and pushes them out. Therefore, **five** messages are ultimately received by the SQS queues.
</details>

-----

**21. When is a subscription filter policy evaluated?**

  * A. By the producer before publishing the message.
  * B. By the SNS service before delivering the message to a specific subscriber.
  * C. By the subscriber after receiving the message.
  * D. By a Lambda function that sits between the topic and the subscriber.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The filtering happens within the **SNS service itself**. After a message is published to a topic, SNS checks the filter policy of each individual subscription. It only delivers the message to subscriptions whose policies match the attributes of the message.
</details>

-----

**22. Which service is a pub/sub messaging service?**

  * A. Amazon SQS
  * B. Amazon MQ
  * C. Amazon SNS
  * D. Amazon Kinesis

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** **Amazon SNS** is the AWS native service for the **publish/subscribe (pub/sub)** messaging model, where publishers are decoupled from subscribers. SQS (A) is a queuing service.
</details>