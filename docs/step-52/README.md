# Step 52: Lambda Event Sources & Triggers

## 🎯 Objective

- [x] Master: **Lambda Event Sources & Triggers**

## 📘 Notes

### **Deep Dive: Lambda Event Sources & Triggers**

An AWS Lambda function is dormant until something triggers it. An **event source** is the AWS service or application that publishes events, and a **trigger** is the configuration that subscribes a Lambda function to those events. The way Lambda is invoked and how it handles retries depends entirely on the type of event source.

---

### **1. Invocation Models**

There are two primary ways a Lambda function can be invoked:

- **Synchronous Invocation (Push Model):**
  - **How it works:** The calling service invokes Lambda directly and **waits for the function to complete** and return a response. The invocation is a blocking call.
  - **Error Handling:** If the function code throws an error, the error is returned directly to the calling service. It is the **caller's responsibility** to handle retries. Lambda itself does not automatically retry a failed synchronous invocation.
  - **Common Services:**
    - **Amazon API Gateway:** A user makes an HTTP request, API Gateway invokes Lambda, waits for the result, and returns it as an HTTP response.
    - **Application Load Balancer:** An ALB can use a Lambda function as a target.
    - Direct calls from the AWS SDK, CLI, or another Lambda function (`Invoke` API).
- **Asynchronous Invocation (Event Model):**
  - **How it works:** The calling service sends an event to Lambda and **does not wait for a response**. It simply hands off the event and expects Lambda to handle it eventually.
  - **Error Handling:** If the function fails, Lambda has a built-in retry mechanism. It will automatically retry the invocation **twice** more, with delays between attempts.
  - **Dead-Letter Queue (DLQ):** If all three attempts (the original + two retries) fail, Lambda can be configured to send the failed event to a **Dead-Letter Queue**, which is typically an **SQS queue** or an **SNS topic**. This allows you to inspect and manually reprocess failed events so they are not lost.
  - **Common Services:**
    - **Amazon S3:** A new object is created.
    - **Amazon SNS:** A message is published to a topic.
    - **Amazon EventBridge (CloudWatch Events):** A scheduled rule or a matching event occurs.
- **Stream-Based Invocation (Polling Model):**
  - **How it works:** This is a special model for streaming services. The **Lambda service itself polls** the event source (the stream or queue) on your behalf, looking for new records.
  - **Batching:** When Lambda finds records, it reads them in a **batch** and sends the entire batch to a single invocation of your function.
  - **Error Handling:** If the function fails to process a batch, Lambda will **retry the same batch** of records over and over until it succeeds or the data expires from the stream (e.g., 24 hours for DynamoDB Streams). This can block the processing of the stream shard. You can configure the retry policy and send failed batches to a DLQ.
  - **Common Services:**
    - **Amazon DynamoDB Streams**
    - **Amazon Kinesis Data Streams**
    - **Amazon SQS** (standard queues)

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A user makes an HTTP request to an Amazon API Gateway endpoint, which is configured to trigger a Lambda function. The Lambda function processes the request and returns a JSON object. What type of invocation model is this?**

- A. Asynchronous Invocation
- B. Stream-Based Invocation
- C. Synchronous Invocation
- D. Scheduled Invocation

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is a classic Synchronous Invocation. API Gateway invokes the Lambda function and then waits for the function to finish and return a result. It then maps this result into an HTTP response to send back to the user. The entire request/response cycle is a single, blocking operation.

</details>

---

**2. A Lambda function is triggered by a new object being created in an S3 bucket. The function code has a bug and throws an error, causing the invocation to fail. What happens next by default?**

- A. The S3 object is automatically deleted.
- B. AWS Lambda automatically retries the function invocation two more times.
- C. The failed event is sent to a Dead-Letter Queue (DLQ).
- D. An email is sent to the AWS account root user.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 triggers are an Asynchronous Invocation. For asynchronous invocations, the default behavior for a function error is for the Lambda service to automatically retry twice, with delays between the attempts. A DLQ (C) is an optional feature you must configure to capture the event after all retries have failed.

</details>

---

**3. A Lambda function is configured to process records from a DynamoDB Stream. It receives a batch of 100 records but fails while processing the 50th record. What will happen to the batch?**

- A. The first 49 records are considered successful, and the function will be invoked again with the remaining 51 records.
- B. The entire batch of 100 records will be retried until the function successfully processes it or the data expires from the stream.
- C. The failed record is sent to a Dead-Letter Queue, and the rest of the batch is discarded.
- D. The Lambda trigger is automatically disabled.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For stream-based event sources like DynamoDB Streams and Kinesis, processing is atomic at the batch level. If the function invocation returns an error for a batch, the entire batch is retried. This means the shard processing is effectively blocked until that batch can be processed successfully.

</details>

---

**4. A developer needs to ensure that if a Lambda function triggered asynchronously fails after all retry attempts, the original event data is not lost. What should they configure?**

- A. Increase the function's timeout to 15 minutes.
- B. Configure a Dead-Letter Queue (DLQ), specifying an SQS queue or SNS topic as the target.
- C. Enable Provisioned Concurrency for the function.
- D. Write the event to an S3 bucket from within the Lambda function code.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Dead-Letter Queue (DLQ) is the feature designed for this purpose. After the initial invocation and the two automatic retries for an asynchronous event fail, the Lambda service will send the failed event object to the configured SQS queue or SNS topic. This allows developers to inspect the failed event and decide how to reprocess it later.

</details>

---

**5. Which service uses a "polling" model, where the Lambda service itself polls for new records and invokes a function with a batch of them?**

- A. Amazon S3
- B. Amazon API Gateway
- C. Amazon SQS
- D. Amazon SNS

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** SQS (along with DynamoDB Streams and Kinesis) uses the stream-based/polling model. The Lambda service acts as the consumer, continuously polling the queue for messages. When it finds messages, it gathers them into a batch and invokes your function. S3 (B) and SNS (D) use the asynchronous "push" model.

</details>

---

**6. A Lambda function is invoked synchronously by an Application Load Balancer. If the function fails due to a code error, who is responsible for handling retries?**

- A. The Lambda service will retry automatically.
- B. The Application Load Balancer will retry automatically.
- C. The client (e.g., the user's browser) that made the original request is responsible for retrying.
- D. The function will be sent to a DLQ.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** For synchronous invocations, Lambda returns the error directly to the caller. In this case, the ALB receives the error and will likely return an HTTP 502 Bad Gateway error to the client. The Lambda service does not perform automatic retries for synchronous failures. It is up to the original client to decide whether to retry the request.

</details>

---

**7. A company wants to run a data processing job every night at 2:00 AM. What is the most appropriate, serverless way to trigger this Lambda function?**

- A. A cron job on an EC2 instance.
- B. An Amazon EventBridge (CloudWatch Events) scheduled rule.
- C. A `while` loop with a `sleep` command inside the Lambda function itself.
- D. Manually invoking the function from the console each night.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Amazon EventBridge is the AWS native service for scheduling. You can create a rule with a cron expression (e.g., cron(0 2 \* _ ? _)) that will trigger your Lambda function automatically at the specified time. This is a fully serverless and reliable solution.

</details>

---

**8. Which of the following is NOT a valid event source for AWS Lambda?**

- A. Amazon S3
- B. Amazon DynamoDB Streams
- C. Amazon RDS
- D. Amazon SQS

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While you can write code inside a Lambda function to connect to an RDS database, Amazon RDS itself does not have a native event source integration to directly trigger a Lambda function (e.g., "trigger on new row"). S3, DynamoDB Streams, and SQS are all common, direct event sources.

</details>

---

**9. When Lambda reads from an SQS standard queue, what is the data passed to the function?**

- A. A single SQS message.
- B. A batch containing one or more SQS messages.
- C. Only the Message ID of a single message.
- D. The entire contents of the queue.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The SQS event source mapping is designed for efficiency. The Lambda service polls the queue and retrieves a batch of messages (up to 10,000 by default, though typically smaller) and passes this entire batch as an array in the event payload to a single function invocation.

</details>

---

**10. What is an "event source mapping"?**

- A. An IAM policy that grants a service permission to invoke a Lambda function.
- B. A resource managed by the Lambda service that reads from an event source (like SQS or Kinesis) and invokes a Lambda function.
- C. A DNS record that maps a domain name to a Lambda function URL.
- D. A diagram showing the flow of events in an application.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An event source mapping is the configuration that connects a polling-based event source to a Lambda function. It's an object you create that contains the details of the stream/queue to poll and the function to invoke. The Lambda service uses this mapping to manage the poller on your behalf.

</details>

---

**11. An S3 event notification is configured to trigger a Lambda function. The function processes the event and succeeds. What happens to the event?**

- A. The event is removed from the S3 event queue.
- B. The event is sent to a DLQ for logging purposes.
- C. Nothing; the event was a one-time invocation and is now gone.
- D. The event is stored in a DynamoDB Stream for 24 hours.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** For asynchronous "push" model sources like S3 and SNS, the invocation is a "fire-and-forget" operation. S3 invokes your function with the event payload. Once the function executes successfully, the process is complete. There is no queue to remove the event from.

</details>

---

**12. When using an Application Load Balancer as a trigger for a Lambda function, what kind of invocation does it use?**

- A. Asynchronous
- B. Synchronous
- C. Stream-based
- D. Scheduled

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The ALB acts as a synchronous caller. It sends the HTTP request information to the Lambda function and waits for the function to return a response object, which the ALB then translates into an HTTP response to send back to the client.

</details>

---

**13. A Lambda function processes records from a Kinesis Data Stream. The function experiences a transient error and fails. The Kinesis stream has a data retention period of 7 days. What will happen?**

- A. The failed batch of records is lost permanently.
- B. The Lambda service will retry processing the same batch of records until it succeeds or the records expire from the stream after 7 days.
- C. The function will be automatically disabled.
- D. The failed batch is moved to an SQS DLQ.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For stream-based sources, Lambda is very persistent. If a batch fails, the event source mapping will repeatedly retry that exact same batch of records. It will continue retrying until either the function returns a success, or the data itself expires based on the stream's retention period (which can be up to 365 days for Kinesis).

</details>

---

**14. Which invocation model provides a built-in, configurable retry mechanism managed by the Lambda service itself?**

- A. Synchronous
- B. Asynchronous
- C. Both Synchronous and Asynchronous
- D. Neither

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The built-in retry mechanism (2 retries by default) is a key feature of the asynchronous invocation model. For synchronous calls, the caller is responsible for implementing any desired retry logic.

</details>

---

**15. You need to trigger a Lambda function once a day to generate a report. Which service is the most appropriate trigger?**

- A. Amazon API Gateway
- B. Amazon S3
- C. Amazon EventBridge
- D. AWS Step Functions

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Amazon EventBridge (formerly CloudWatch Events) is the AWS native scheduler. You can create a rule that runs on a schedule (e.g., once per day) and set your Lambda function as the target.

</details>

---

**16. The `event` object that is passed to a Lambda function handler contains what information?**

- A. The IAM credentials for the function's execution role.
- B. The payload and metadata sent by the triggering service.
- C. The function's configuration, such as memory and timeout.
- D. The logs from the previous invocation.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The event object is the input data for the function. Its structure and content depend entirely on the event source. For an S3 trigger, it will contain information about the bucket and object key. For an API Gateway trigger, it will contain the HTTP method, headers, and body.

</details>

---

**17. A Dead-Letter Queue (DLQ) for a failed asynchronous Lambda invocation can be which two types of AWS resources? (Select TWO)**

- A. A Kinesis Data Stream
- B. An Amazon SNS topic
- C. An Amazon S3 bucket
- D. An Amazon SQS queue
- E. Another Lambda function

<details>
<summary>View Answer</summary>

**Answers: B and D**

**Explanation:** When you configure a DLQ for a Lambda function, you can specify either an Amazon SQS queue or an Amazon SNS topic as the destination for the failed event payloads.

</details>

---

**18. Which invocation model processes records in batches by default?**

- A. Synchronous (e.g., API Gateway)
- B. Asynchronous (e.g., S3)
- C. Stream-based (e.g., DynamoDB Streams)
- D. All models use batching.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The stream-based invocation model used for SQS, Kinesis, and DynamoDB Streams is designed around batching. The Lambda service reads multiple records from the source at once and passes them to the function in a single invocation to improve efficiency.

</details>

---

**19. A developer manually invokes a Lambda function using the `Invoke` API call with an `InvocationType` of `RequestResponse`. What does this mean?**

- A. The function will be invoked asynchronously.
- B. The `Invoke` call will return immediately with a status code, and the function will run in the background.
- C. The `Invoke` call will wait for the function to complete and will return the function's result in the response.
- D. The function will be triggered, but no response will be returned.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** RequestResponse is the parameter for a synchronous invocation. The API call is a blocking call that will wait until the Lambda function finishes its execution and then return the payload from the function as the API response.

</details>

---

**20. If you manually invoke a Lambda function with an `InvocationType` of `Event`, what type of invocation is this?**

- A. Synchronous
- B. Asynchronous
- C. Scheduled
- D. Dry Run

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Event is the parameter for an asynchronous invocation. The Invoke API call will return a 202 Accepted status code immediately, and the Lambda service will handle the execution of the function in the background, including any retries if it fails.

</details>

---

**21. A Lambda function processes data from a high-volume Kinesis Data Stream. The function is timing out because it cannot process the large batches of records it receives within the configured timeout. What is a potential solution?**

- A. Decrease the batch size in the event source mapping.
- B. Switch to an asynchronous invocation model.
- C. Decrease the function's memory.
- D. Increase the Kinesis stream's retention period.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** By decreasing the batch size, you send fewer records to each Lambda invocation. This means each invocation has less work to do and is more likely to complete within the timeout period. This will result in more Lambda invocations overall but can solve the timeout issue.

</details>

---

**22. What is the primary purpose of configuring a DLQ for a Lambda function?**

- A. To improve the performance of the function.
- B. To provide a destination for failed asynchronous event payloads so they can be analyzed and reprocessed.
- C. To log all successful invocations.
- D. To automatically retry synchronous invocations.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Dead-Letter Queue (DLQ) is an error handling mechanism. Its main purpose is to capture and store events that have failed processing after all automatic retries have been exhausted. This prevents data loss and allows for later debugging and reprocessing.

</details>
