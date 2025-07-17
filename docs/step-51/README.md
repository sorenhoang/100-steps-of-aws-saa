# Step 51: Lambda Fundamentals & Limits

## 🎯 Objective

- [x] Master: **Lambda Fundamentals & Limits**

## 📘 Notes

### **Deep Dive: Lambda Fundamentals & Limits**

AWS Lambda is a **serverless, event-driven compute service** that lets you run code for virtually any type of application or backend service without provisioning or managing servers. You can trigger Lambda functions from over 200 AWS services and SaaS applications, and you only pay for what you use.

---

### **1. The Lambda Model: Event-Driven & Serverless**

- **Serverless:** This is the most important concept. "Serverless" doesn't mean there are no servers; it means **you** don't manage them. AWS handles all the provisioning, patching, scaling, and operational overhead of the underlying compute infrastructure. You just provide your code.
- **Event-Driven:** Lambda functions do not run continuously. They are **event-driven**, meaning they execute only in response to a specific **trigger** or **event**.
  - **Example Triggers:**
    - An HTTP request from Amazon API Gateway.
    - A new object being uploaded to an S3 bucket.
    - A new message arriving in an SQS queue.
    - A new record in a DynamoDB Stream.
    - A scheduled event from Amazon EventBridge (e.g., "run this function every hour").
- **Execution:** When a trigger occurs, Lambda provisions a secure, isolated execution environment, runs your function's code, and then tears down the environment. Each invocation is handled independently and can be scaled massively in parallel.

---

### **2. Key Lambda Concepts & Configuration**

- **Function Code & Runtimes:** You can write Lambda functions in popular languages like Python, Node.js, Java, Go, Ruby, and .NET. AWS provides managed **runtimes** for these languages. You can also provide your own custom runtime or package your function as a container image.
- **Execution Role:** Every Lambda function has an IAM role associated with it. This **Execution Role** grants the function the necessary permissions to interact with other AWS services (e.g., permission to write to a DynamoDB table or read from an S3 bucket).
- **Resource Allocation:**
  - **Memory:** You allocate memory to your function, from 128 MB up to 10,240 MB (10 GB).
  - **CPU:** CPU power is allocated **proportionally** to the amount of memory you configure. The more memory you give your function, the more CPU power it gets.

---

### **3. Critical Lambda Limits**

Understanding these limits is crucial for designing applications with Lambda.

- **Execution Timeout:** This is the maximum amount of time a single invocation of your function is allowed to run.
  - The timeout can be set from 1 second up to a maximum of **900 seconds (15 minutes)**.
  - If your function's execution exceeds this timeout, Lambda will terminate it. This makes Lambda unsuitable for very long-running batch jobs that cannot be broken down into smaller pieces.
- **Concurrency:** This is the number of invocations of your function that can run simultaneously in a given region.
  - By default, your AWS account has a total concurrency limit of **1,000** across all functions in a region. This is a soft limit that can be increased.
  - **Reserved Concurrency:** You can reserve a specific amount of the account-level concurrency for a critical function. For example, you can reserve 100 concurrent executions for `function-A`. This guarantees that `function-A` can always scale up to 100, but it also means no other function can use that reserved capacity.
  - **Provisioned Concurrency:** This is used to eliminate "cold starts." You can pay to keep a specified number of execution environments "warm" and ready to respond instantly.
- **Deployment Package Size:** The size of your zipped deployment package (`.zip` file) is limited to 50 MB (zipped) and 250 MB (unzipped). For larger dependencies, you should use Lambda Layers or package the function as a container image.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A developer needs to run a script that processes a file uploaded to S3. The script typically takes 5 minutes to run and should be executed automatically every time a new file is uploaded. The developer does not want to manage any servers. Which AWS service is the best fit for this task?**

- A. An EC2 instance with a cron job.
- B. AWS Lambda.
- C. AWS Batch.
- D. Amazon ECS with Fargate.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic use case for AWS Lambda. It is an event-driven, serverless compute service. You can configure an S3 bucket event to act as a trigger, which automatically invokes the Lambda function whenever a new object is created. The 5-minute runtime is well within Lambda's 15-minute limit.

</details>

---

**2. What is the maximum execution timeout for a single AWS Lambda function invocation?**

- A. 60 seconds (1 minute)
- B. 300 seconds (5 minutes)
- C. 900 seconds (15 minutes)
- D. 3600 seconds (1 hour)

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is a key limit to memorize for the exam. A Lambda function can run for a maximum of 900 seconds (15 minutes). If the execution exceeds this limit, the function will be terminated.

</details>

---

**3. A Lambda function needs to read items from a DynamoDB table. What is the correct way to grant it the necessary permissions?**

- A. Hard-code an IAM user's access keys into the Lambda function's code.
- B. Assign an IAM Role (an Execution Role) to the Lambda function that has a policy allowing read access to the DynamoDB table.
- C. Configure the DynamoDB table to allow public read access.
- D. Attach a security group to the Lambda function.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The standard and secure way to grant permissions to a Lambda function is through its Execution Role. This IAM role provides the function with temporary, automatically-rotated credentials to call other AWS services. Hard-coding credentials (A) is a major security anti-pattern.

</details>

---

**4. A company has two Lambda functions. `Function-A` is critical and must always be able to handle traffic spikes. `Function-B` is a non-critical, background processing task. The company is concerned that a large number of invocations for `Function-B` could use up all the account's available concurrency and prevent `Function-A` from running. What should be done?**

- A. Increase the memory allocated to `Function-A`.
- B. Set a higher timeout for `Function-A`.
- C. Configure Reserved Concurrency for `Function-A`.
- D. Run `Function-B` in a different AWS Region.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Reserved Concurrency is designed for this exact scenario. By reserving a specific amount of concurrency (e.g., 200) for the critical Function-A, you guarantee that it will always have that capacity available. This also has the side effect of limiting Function-A to never exceed that reserved amount, protecting downstream resources.

</details>

---

**5. How is CPU power allocated to an AWS Lambda function?**

- A. It is a fixed amount for all functions.
- B. You can select the number of vCPUs independently of memory.
- C. CPU power is allocated proportionally to the amount of memory configured for the function.
- D. You can choose a specific EC2 instance type to run the function on.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** In Lambda, you only directly control the memory allocation. CPU power, network bandwidth, and other resources are allocated proportionally to the memory setting. A function with 1024 MB of memory will receive more CPU power than a function with 128 MB of memory.

</details>

---

**6. What does "serverless" mean in the context of AWS Lambda?**

- A. The code runs without a server.
- B. The code runs directly on the user's browser.
- C. Developers do not need to provision, manage, or patch the underlying servers; AWS handles it automatically.
- D. The service is free to use.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** "Serverless" is about abstraction. Servers are still involved, but they are completely managed by AWS. This allows developers to focus only on their application code and not on the operational overhead of managing the infrastructure it runs on.

</details>

---

**7. A Lambda function is configured with a timeout of 10 seconds. The code it runs typically finishes in 2 seconds, but sometimes an external API call can take up to 20 seconds to respond. What will happen during an invocation where the API call takes 20 seconds?**

- A. The Lambda function will automatically extend its timeout to 20 seconds.
- B. The Lambda function will be terminated after 10 seconds, and an error will be logged.
- C. The function will complete successfully but with a high latency warning.
- D. The function will be retried automatically with a longer timeout.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The configured timeout is a hard limit. If the function's execution time exceeds the timeout setting, AWS Lambda will forcibly terminate the execution environment, and the invocation will be registered as a failure (timeout error).

</details>

---

**8. Which of the following is a valid trigger for a Lambda function?**

- A. A new object being created in an S3 bucket.
- B. An EC2 instance entering the `stopped` state.
- C. A new message arriving in an SQS queue.
- D. All of the above.

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** AWS Lambda is deeply integrated across the AWS ecosystem. S3 events (A), EC2 state changes via EventBridge (B), and SQS queues (C) are all common and valid event sources that can be used to trigger a Lambda function.

</details>

---

**9. What is the default account-level concurrency limit for Lambda functions in a region?**

- A. 100
- B. 1,000
- C. 10,000
- D. There is no limit.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** By default, an AWS account can have up to 1,000 concurrent Lambda executions running at any given moment across all functions within a single region. This is a soft limit that can be increased by submitting a request to AWS Support.

</details>

---

**10. A developer needs to include a large library (100 MB) that their Python Lambda function depends on. The zipped deployment package for the function code itself is small. What is the best way to manage this large dependency?**

- A. Upload the library to S3 and have the function download it on every invocation.
- B. Increase the function's memory to 10 GB to accommodate the library.
- C. Package the dependency as a Lambda Layer and attach it to the function.
- D. This is not possible due to the deployment package size limit.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Lambda Layers are designed for this exact purpose. A layer is a .zip file archive that can contain libraries, a custom runtime, or other dependencies. You can create a layer with the large library and then attach that layer to multiple functions. This keeps your function deployment package small and simplifies dependency management.

</details>

---

**11. When is a user billed for an AWS Lambda function?**

- A. A flat monthly fee per function.
- B. Based on the number of hours the function is deployed.
- C. Based on the number of requests (invocations) and the duration of the execution (in milliseconds).
- D. Based on the amount of data transferred out of the function.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Lambda has a pay-per-use pricing model. The two primary billing dimensions are the number of invocations and the compute duration (measured in milliseconds), which is also scaled by the amount of memory you've allocated to the function. There is also a generous free tier.

</details>

---

**12. What is a "cold start" in the context of AWS Lambda?**

- A. When a function is invoked in a cold weather region.
- B. The extra latency experienced on the first invocation of a function after a period of inactivity, as AWS has to create a new execution environment.
- C. An error that occurs when a function's code has a bug.
- D. When a function is invoked with no event data.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A cold start refers to the setup time required for a new execution environment. If a function hasn't been used recently, Lambda tears down its environment. The next time it's invoked, AWS has to provision the environment, download the code, and initialize the runtime, which adds a small amount of latency to that first request. Subsequent requests to the "warm" environment are much faster.

</details>

---

**13. A company wants to eliminate cold start latency for a mission-critical Lambda function that handles API requests. What feature should they use?**

- A. Reserved Concurrency
- B. Provisioned Concurrency
- C. A larger memory allocation.
- D. Lambda Layers

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Provisioned Concurrency is the feature designed to solve the cold start problem. You pay to keep a specified number of execution environments initialized and "warm" at all times. This ensures that when a request comes in, it can be served instantly by one of these pre-warmed environments, eliminating cold start latency.

</details>

---

**14. If you allocate more memory to a Lambda function, what other resource is increased proportionally?**

- A. The execution timeout.
- B. The number of environment variables.
- C. The CPU power.
- D. The deployment package size limit.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** CPU is not an independently configurable setting. AWS allocates CPU power in proportion to the amount of memory you configure. If you have a CPU-bound task, increasing the memory is the way to give it more processing power.

</details>

---

**15. What component of the AWS serverless ecosystem is most commonly used to create an HTTP endpoint that triggers a Lambda function?**

- A. Application Load Balancer
- B. Amazon API Gateway
- C. Amazon SQS
- D. Amazon CloudFront

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Amazon API Gateway is the standard service for creating, publishing, and securing APIs. A very common serverless pattern is to create an API Gateway endpoint that triggers a Lambda function to handle the business logic for incoming HTTP requests.

</details>

---

**16. How can a Lambda function be granted access to resources inside a VPC, such as an RDS database?**

- A. By default, Lambda functions can access any resource in the same AWS account.
- B. By attaching an Elastic IP address to the Lambda function.
- C. By configuring the Lambda function to run inside the VPC, which attaches an Elastic Network Interface (ENI) to it.
- D. By creating a VPC Peering connection between the Lambda service and the VPC.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** To access resources within a VPC, you must configure the Lambda function to launch into that VPC. When you do this, AWS creates an ENI in your specified subnets, giving the function a private IP address and allowing it to communicate with other resources like RDS databases via their private IPs.

</details>

---

**17. A Lambda function needs to process a large number of messages from an SQS queue. How are the messages passed to the Lambda function?**

- A. The function is invoked once for each message in the queue.
- B. The Lambda service polls the queue and invokes the function with a batch of messages.
- C. The SQS queue pushes messages directly to the function's HTTP endpoint.
- D. The function must contain code to poll the SQS queue itself.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For SQS triggers, the AWS Lambda service itself polls the queue on your behalf. To improve efficiency, it retrieves messages in a batch (the size of which you can configure) and passes that entire batch to a single invocation of your Lambda function.

</details>

---

**18. For which of the following tasks would AWS Lambda be an unsuitable choice due to its limits?**

- A. Resizing an image uploaded to S3.
- B. Processing a real-time API request.
- C. Running a data analytics job that takes 45 minutes to complete.
- D. A scheduled task that runs for 2 minutes every hour.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The maximum execution timeout for a Lambda function is 15 minutes. A job that takes 45 minutes to run will be terminated by the Lambda service before it can complete. For long-running batch jobs like this, services like AWS Batch or Fargate/ECS would be more appropriate.

</details>

---

**19. What is a Lambda "runtime"?**

- A. The total execution time of a function.
- B. A pre-configured environment provided by AWS that includes a specific programming language version and required libraries.
- C. The amount of memory allocated to the function.
- D. The number of concurrent executions allowed for the function.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A runtime is the execution environment that runs your code. AWS provides managed runtimes for popular languages (e.g., python3.9, nodejs18.x) so you don't have to build the environment yourself.

</details>

---

**20. A function that processes images needs the `Pillow` library in Python. This library is not included in the standard Lambda runtime. How should this dependency be provided to the function?**

- A. Use a Lambda Layer that contains the `Pillow` library.
- B. The function must be written in a different language that includes the library.
- C. The function must be run on Fargate instead of Lambda.
- D. Request that AWS add the library to the standard runtime.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Lambda Layers are the perfect solution for including third-party libraries or custom dependencies. You can package the Pillow library into a layer and then attach that layer to your function, making it available to your code at runtime.

</details>

---

**21. What happens if a Lambda function throws an unhandled error during execution?**

- A. The invocation is recorded as successful but with a warning.
- B. The execution environment is permanently deleted.
- C. The invocation is recorded as a failure, and the error is logged in CloudWatch Logs.
- D. AWS Support is automatically notified.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** If your function's code crashes or throws an exception that is not caught, the Lambda service will terminate that specific invocation and mark it as a failure. Details about the error, including the stack trace, will be sent to the function's associated CloudWatch Logs log group for debugging.

</details>

---

**22. Which service is used to trigger a Lambda function on a recurring schedule (e.g., every day at midnight)?**

- A. Amazon SQS
- B. AWS Step Functions
- C. Amazon EventBridge (CloudWatch Events)
- D. AWS Lambda itself has a built-in scheduler.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Amazon EventBridge is AWS's serverless event bus service. One of its key features is the ability to create "rules" that run on a fixed schedule, defined either by a rate (e.g., every 5 minutes) or a cron expression. You can set the target of this rule to be your Lambda function, effectively creating a scheduled task.

</details>
