# Step 55: Serverless Pattern: Lambda + DynamoDB

## 🎯 Objective

- [x] Master: **Serverless Pattern: Lambda + DynamoDB**

## 📘 Notes

### **Deep Dive: Serverless Pattern – Lambda + DynamoDB**

The combination of AWS Lambda and Amazon DynamoDB is the quintessential serverless backend. This pattern allows you to build highly scalable, resilient, and cost-effective applications without managing any servers. It leverages the strengths of both services: Lambda for event-driven compute and DynamoDB for high-performance, scalable NoSQL data storage.

---

### **1. The Core Architecture** 🏗️

The most common implementation of this pattern is for building a RESTful API.

1. **Client:** A user or application makes an HTTP request (e.g., `GET`, `POST`, `DELETE`).
2. **Amazon API Gateway:** This acts as the "front door." It receives the HTTP request, validates it, and triggers the appropriate Lambda function based on the request's path and method.
3. **AWS Lambda:** This is your business logic layer. The Lambda function receives the request data from API Gateway as an `event` object. It contains your code (e.g., in Python or Node.js) to process the request.
4. **Amazon DynamoDB:** This is your persistence layer. The Lambda function uses the AWS SDK to perform operations on a DynamoDB table, such as creating a new item, reading an existing item, or deleting an item.
5. **Response Flow:** The Lambda function gets a result from DynamoDB, formats a response object, and returns it to API Gateway, which then transforms it into an HTTP response and sends it back to the client.

---

### **2. Why This Pattern Works So Well**

- **Fully Serverless:** You manage no servers for the API, the compute, or the database. AWS handles all patching, scaling, and operational management.
- **Independent Scaling:** Each component scales independently and automatically. If your API gets a massive traffic spike, API Gateway and Lambda will scale to handle millions of concurrent requests, and DynamoDB's On-Demand capacity can scale to match the read/write throughput.
- **Pay-per-Use:** You pay only when your API is actually being used. When there are no requests, you pay nothing for API Gateway or Lambda, and only for the data you have stored in DynamoDB. This is extremely cost-effective compared to running an idle server 24/7.
- **Stateless Logic:** Lambda functions are stateless, which encourages good application design. Any state required to process a request is fetched from and stored back into the persistence layer (DynamoDB).

---

### **3. Critical Component: The Lambda Execution Role** 🔑

For this pattern to work, the Lambda function needs permission to talk to the DynamoDB table. This is achieved via its **Execution Role**.

- **What it is:** An IAM Role that you create and assign to your Lambda function.
- **Permissions:** You must attach an IAM policy to this role that grants the necessary DynamoDB permissions.
- **Principle of Least Privilege:** You should only grant the permissions the function absolutely needs.
  - If a function only needs to read user data, its role should only have `dynamodb:GetItem` permission.
  - If a function creates new users, it needs `dynamodb:PutItem` permission.
  - Never grant `dynamodb:*` (all actions) unless it's absolutely necessary.

This role provides temporary, secure credentials to your function at runtime, so you never have to hard-code access keys in your code.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company wants to build a simple REST API to create and retrieve user profiles. They want a solution that requires no server management and scales automatically based on traffic. What is the most common and appropriate serverless architecture for this?**

- A. An EC2 instance running a Node.js application connected to an RDS database.
- B. An Application Load Balancer distributing traffic to an ECS Fargate service.
- C. Amazon API Gateway, AWS Lambda, and Amazon DynamoDB.
- D. Amazon S3 static website hosting with a direct connection to DynamoDB.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the canonical serverless API pattern. API Gateway provides the HTTP endpoint, Lambda provides the serverless compute for the business logic, and DynamoDB provides the scalable, managed NoSQL database. This combination requires zero server management and scales seamlessly.

</details>

---

**2. In the API Gateway -> Lambda -> DynamoDB pattern, how should the Lambda function be given permission to access the DynamoDB table?**

- A. By hard-coding an IAM user's access keys into the Lambda function's environment variables.
- B. By attaching an IAM Role (an Execution Role) to the function with a policy granting the necessary DynamoDB permissions.
- C. By creating a VPC Endpoint for DynamoDB.
- D. By configuring the DynamoDB table to allow public access.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The secure and standard way to grant permissions is by using an Execution Role. This IAM role is assumed by the Lambda function at runtime, providing it with temporary, secure credentials to interact with other AWS services as defined in its attached IAM policy.

</details>

---

**3. What is a major benefit of using DynamoDB as the backend for a Lambda-based API?**

- A. It supports complex SQL JOIN operations.
- B. It offers single-digit millisecond latency and scales seamlessly along with the Lambda function to handle high traffic loads.
- C. It requires manual scaling of read and write capacity.
- D. It provides a built-in caching layer.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Lambda can scale to handle thousands of concurrent requests. It needs a database that can keep up. DynamoDB is designed for high performance at scale, delivering consistent low latency even as the number of requests skyrockets. This makes it a perfect match for Lambda's execution model.

</details>

---

**4. A Lambda function receives a request from API Gateway to create a new user. The function's code attempts to write an item to a DynamoDB table but receives a permissions error. The function's Execution Role has a policy allowing `dynamodb:GetItem` and `dynamodb:Query`. Why is the write operation failing?**

- A. The Lambda function needs to be placed inside a VPC.
- B. The Execution Role is missing the `dynamodb:PutItem` permission.
- C. The DynamoDB table is in a different region.
- D. The API Gateway does not have permission to write to DynamoDB.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic "principle of least privilege" issue. The role only grants read permissions (GetItem, Query). To create or update an item, the function requires the dynamodb:PutItem permission. The error is occurring because the Lambda function itself lacks the authorization to perform the write action.

</details>

---

**5. Which component in this serverless pattern is responsible for handling the business logic?**

- A. API Gateway
- B. DynamoDB
- C. AWS Lambda
- D. The client application.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The AWS Lambda function contains your custom application code. It's where you define the business logic for what should happen when a specific API endpoint is called (e.g., validate input, transform data, interact with the database).

</details>

---

**6. A serverless application experiences massive, unpredictable traffic spikes. Which capacity mode for the backend DynamoDB table would be most appropriate and easiest to manage?**

- A. Provisioned Throughput with Auto Scaling.
- B. Provisioned Throughput with a high, fixed capacity.
- C. On-Demand Capacity.
- D. A manual snapshot and restore process.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** For unpredictable and spiky workloads, On-Demand Capacity is the ideal choice. It eliminates the need for capacity planning and automatically scales to meet the demand from the Lambda functions, charging on a per-request basis. While Auto Scaling (A) works, it can lag behind very sudden spikes, whereas On-Demand is instantaneous.

</details>

---

**7. In this pattern, what acts as the "front door" for all incoming HTTP requests?**

- A. AWS Lambda
- B. Amazon CloudFront
- C. The DynamoDB table endpoint.
- D. Amazon API Gateway

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Amazon API Gateway is the managed service that provides the public HTTP endpoint. It receives requests from clients, handles things like authorization and throttling, and then triggers the appropriate backend logic (the Lambda function).

</details>

---

**8. Why is the combination of Lambda and DynamoDB considered a "serverless" pattern?**

- A. Because no servers are involved anywhere in the process.
- B. Because developers do not need to provision, manage, or patch any underlying servers for either the compute or the database layer.
- C. Because the code runs on the user's local machine.
- D. Because it does not require a VPC.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The term "serverless" refers to the abstraction of servers from the developer's perspective. AWS manages the entire underlying infrastructure for both Lambda and DynamoDB, allowing developers to focus solely on their application code and data model.

</details>

---

**9. How does API Gateway pass request data (like query parameters or a request body) to a Lambda function?**

- A. It writes the data to a temporary S3 bucket for the Lambda to read.
- B. It passes the data as command-line arguments to the function.
- C. It packages the request data into a JSON object, which becomes the `event` parameter in the Lambda handler function.
- D. It streams the data over a WebSocket connection.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** API Gateway acts as a translator. It takes the incoming HTTP request and transforms it into a structured JSON event object. This object, containing all the details of the request, is then passed as the first argument to the invoked Lambda function's handler.

</details>

---

**10. What is a major cost benefit of the Lambda + DynamoDB pattern compared to a traditional server-based approach (e.g., EC2 + RDS)?**

- A. Data transfer between Lambda and DynamoDB is always free.
- B. The pay-per-use model means you are not charged for idle compute or database capacity.
- C. It requires fewer IAM roles, reducing complexity.
- D. DynamoDB storage is cheaper than RDS storage.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** With a traditional approach, you pay for your EC2 and RDS instances 24/7, even when they are idle. With the serverless pattern, you only pay when an API request is made and the Lambda function executes. This elimination of payment for idle resources is a primary cost advantage, especially for applications with variable traffic.

</details>

---

**11. A Lambda function needs to perform a `Scan` operation on a large DynamoDB table. What is a potential issue with this design?**

- A. `Scan` operations are not allowed by Lambda functions.
- B. A `Scan` operation can be slow and consume a large number of Read Capacity Units, potentially leading to high costs and function timeouts.
- C. The Lambda function will need a larger memory allocation to perform a `Scan`.
- D. The results of a `Scan` cannot be returned through API Gateway.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Scan operations are inefficient because they read every item in a table. On a large table, this can be very slow, potentially causing the Lambda function to exceed its maximum 15-minute timeout. It also consumes a large number of RCUs, which can be expensive. The proper design is to use a Query operation with a primary key or a GSI.

</details>

---

**12. When using this serverless pattern, where should application state (like user session information) be stored?**

- A. In global variables within the Lambda function code.
- B. In the `/tmp` directory of the Lambda execution environment.
- C. In the DynamoDB table.
- D. In environment variables for the Lambda function.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Lambda functions are stateless. You should never store persistent state in the function itself, as you have no guarantee you will get the same execution environment for the next request. All state must be externalized, and the DynamoDB table is the perfect place to store it.

</details>

---

**13. Which component of this architecture is responsible for authenticating and authorizing API requests, for example, by validating a JWT token?**

- A. AWS Lambda
- B. Amazon DynamoDB
- C. AWS WAF
- D. Amazon API Gateway

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** API Gateway provides built-in authorization mechanisms. You can configure it with a Cognito User Pool authorizer or a Lambda authorizer to validate tokens (like JWTs) before the request is ever passed to your backend Lambda function. This secures the front door to your application.

</details>

---

**14. A Lambda function successfully writes an item to DynamoDB. The function then immediately returns a success message to API Gateway. What consistency model will a subsequent read for that item from a different Lambda function experience?**

- A. It is guaranteed to be a strongly consistent read.
- B. By default, it will be an eventually consistent read, and might not see the new item for a brief moment.
- C. DynamoDB reads are always transactional.
- D. The consistency depends on the API Gateway cache settings.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** By default, all read operations in DynamoDB (e.g., GetItem, Query) are eventually consistent. This provides the best read performance. While the data is replicated across multiple locations in milliseconds, there's a possibility that an immediate read might not see the latest write. You can request a strongly consistent read, but that is not the default.

</details>

---

**15. If a Lambda function needs access to resources in a VPC (like an ElastiCache cluster) in addition to DynamoDB, what needs to be configured?**

- A. A VPC Peering connection between Lambda and the VPC.
- B. The Lambda function must be configured to run inside the VPC, and a VPC Endpoint for DynamoDB should be used.
- C. The ElastiCache cluster needs a public IP address.
- D. This architecture is not possible.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** To access VPC resources, the Lambda must be launched into the VPC. However, doing so cuts off its default path to public AWS service endpoints like DynamoDB. To regain access privately, you must create a VPC Endpoint for DynamoDB (a Gateway Endpoint) and add it to the VPC's route table.

</details>

---

**16. How does this serverless pattern handle scaling?**

- A. You must use an Auto Scaling group to manage the Lambda functions.
- B. Each component (API Gateway, Lambda, DynamoDB) scales automatically and independently based on demand.
- C. You must manually increase the capacity of each service during peak times.
- D. Scaling is handled by a central AWS Control Plane service.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a key benefit of the pattern. You don't configure scaling for the whole stack. Each service manages its own scaling. API Gateway handles concurrent connections, Lambda scales by creating more concurrent execution environments, and DynamoDB (in On-Demand mode) scales its throughput instantly.

</details>

---

**17. A developer is building an API to get an item from a DynamoDB table. The API Gateway method and Lambda integration are set up. What is the minimum permission needed in the Lambda Execution Role?**

- A. `dynamodb:*`
- B. `dynamodb:PutItem`
- C. `dynamodb:GetItem`
- D. `dynamodb:ReadData`

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Following the principle of least privilege, if the function only needs to read a single item, it only needs the dynamodb:GetItem permission for the specific table resource. Granting wider permissions like dynamodb:\* is a security risk.

</details>

---

**18. What is a key benefit of using API Gateway's Lambda Proxy Integration?**

- A. It provides more granular control over the HTTP response code and headers.
- B. It simplifies the setup by passing the entire raw HTTP request to Lambda and expecting a specific JSON structure back for the response.
- C. It allows the Lambda function to bypass IAM for permissions.
- D. It automatically caches all responses.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Lambda Proxy Integration is a streamlined way to connect API Gateway to Lambda. You don't need to create mapping templates. API Gateway simply forwards the entire request as-is within the event object. Your Lambda function is then responsible for interpreting it and returning a response in a specific JSON format that API Gateway understands, which includes the statusCode, headers, and body.

</details>

---

**19. In what scenario would this simple serverless pattern be a poor architectural choice?**

- A. A public-facing REST API for a mobile application.
- B. A webhook receiver that processes incoming data from a third-party service.
- C. A long-running data processing job that needs to run for 2 hours.
- D. An internal API used by other microservices.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AWS Lambda has a maximum execution timeout of 15 minutes. A job that needs to run for 2 hours is not suitable for this pattern. It would require a different compute service like AWS Batch, ECS, or a long-running process on an EC2 instance.

</details>

---

**20. A company wants to add rate limiting to its public API to prevent abuse. Where should this be configured?**

- A. In the Lambda function's code.
- B. On the DynamoDB table.
- C. In the API Gateway stage settings or usage plans.
- D. In a security group.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** API Gateway has built-in throttling capabilities. You can set a rate and burst limit on a per-stage or per-method basis. For more granular control (e.g., per-customer), you can use Usage Plans and API Keys with a REST API. This is more efficient and reliable than trying to implement rate limiting in the Lambda code.

</details>

---

**21. The data passed from API Gateway to Lambda is in what format?**

- A. XML
- B. A binary blob
- C. A URL-encoded string
- D. JSON

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The event payload that API Gateway sends to invoke a Lambda function is a structured JSON object. It contains details like the HTTP method, path, headers, query string parameters, and the request body.

</details>

---

**22. Which part of this architecture is responsible for persisting the data durably?**

- A. API Gateway
- B. AWS Lambda
- C. Amazon DynamoDB
- D. The client's browser cache.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** API Gateway and Lambda are stateless compute and routing layers. Amazon DynamoDB is the persistence layer. It is the service responsible for durably storing the application's data across multiple Availability Zones.

</details>
