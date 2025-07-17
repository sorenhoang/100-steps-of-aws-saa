# Step 53: API Gateway – REST vs HTTP vs WebSocket

## 🎯 Objective

- [x] Master: **API Gateway – REST vs HTTP vs WebSocket**

## 📘 Notes

### **Deep Dive: API Gateway – REST vs. HTTP vs. WebSocket**

Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. It acts as a "front door" for applications to access data, business logic, or functionality fro1m your backend services, such as AWS Lambda, EC2, or any web application.

---

### **1. API Gateway - Common Concepts**

- **Integration:** API Gateway connects the frontend (the method request from the client) to the backend (the integration request to your service). It can integrate with Lambda functions, other HTTP endpoints, and various AWS services directly.
- **Throttling:** Protects your backend from being overwhelmed by limiting the rate of requests per second.
- **Security:** Provides multiple mechanisms for authentication and authorization, including IAM roles, Cognito User Pools, and Lambda authorizers.
- **Custom Domains:** Allows you to use your own domain name for your API.
- **Stages:** You can create different deployment stages for your API, like `dev`, `test`, and `prod`, each with its own endpoint and configuration.

---

### **2. The Three API Types**

- **REST API:** ↔️ (The Feature-Rich Original)
  - **What it is:** This is the original, most feature-rich type of API Gateway. It provides a comprehensive set of tools for building and managing RESTful APIs.
  - **Key Features:**
    - Supports both stateful (WebSocket) and stateless (REST) communication.
    - Offers advanced features like API keys, per-client throttling, request validation, caching, and integration with AWS WAF.
    - Provides more control over the request and response transformation.
  - **Use Case:** The best choice when you need advanced API management capabilities, fine-grained control, or features like API keys and caching.
- **HTTP API:** 🚀 (The Lighter, Faster, Cheaper Option)
  - **What it is:** A newer, streamlined API type designed for building low-latency and cost-effective APIs.
  - **Key Features:**
    - **Lower Latency & Cost:** Offers better performance and is significantly cheaper than REST APIs.
    - **Simplified Features:** Provides core functionality like routing, JWT authorizer integration, and custom domains, but lacks the advanced features of REST APIs (like API keys, caching, WAF integration).
    - **Easy Integration:** Designed for easy, direct integration with AWS Lambda and other HTTP endpoints.
  - **Use Case:** The ideal choice for building simple, high-performance, serverless APIs that don't require the extensive feature set of a REST API. It is the recommended starting point for new HTTP APIs.
- **WebSocket API:** 🔄 (For Real-Time, Two-Way Communication)
  - **What it is:** This API type is specifically designed to build real-time, two-way communication applications.
  - **How it works:** It maintains a persistent, stateful connection between the client and the backend. Both the client and the server can send messages to each other at any time over this open connection.
  - **Key Features:**
    - Manages the persistent connections for you.
    - Uses "routes" based on the content of the message to invoke backend services (like Lambda). Common routes include `$connect`, `$disconnect`, and `$default`.
  - **Use Case:** Real-time applications like chat apps, live sports tickers, collaborative editing tools, and multiplayer games.

---

### **Summary of Key Differences (HTTP vs. REST)**

| Feature                   | HTTP API                              | REST API                    |
| ------------------------- | ------------------------------------- | --------------------------- |
| **Performance**           | **Higher Performance, Lower Latency** | Standard Performance        |
| **Cost**                  | **Lower Cost**                        | Higher Cost                 |
| **Core Use Case**         | Simple, low-latency, serverless APIs  | Advanced, feature-rich APIs |
| **API Keys**              | No                                    | **Yes**                     |
| **Per-Client Throttling** | No                                    | **Yes**                     |
| **Caching**               | No                                    | **Yes**                     |
| **WAF Integration**       | No                                    | **Yes**                     |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company is building a real-time chat application where users need to receive messages instantly without constantly polling a server. Which API Gateway type is designed for this kind of persistent, two-way communication?**

- A. HTTP API
- B. REST API
- C. WebSocket API
- D. A regional REST API.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the primary use case for a WebSocket API. It is specifically designed to handle stateful, full-duplex (two-way) communication, which is necessary for real-time applications like chat rooms, notifications, and live dashboards.

</details>

---

**2. A startup is creating a new serverless backend using AWS Lambda. Their primary requirements are the lowest possible cost and the fastest possible performance for their HTTP API. They do not need advanced features like API keys or caching. Which API Gateway type is the most appropriate choice?**

- A. REST API
- B. HTTP API
- C. WebSocket API
- D. They should use an Application Load Balancer instead.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The HTTP API is purpose-built for this scenario. It is a newer, streamlined version of API Gateway that offers significantly lower latency and lower cost compared to the REST API, making it the ideal choice for high-performance, cost-sensitive serverless applications that don't need extensive features.

</details>

---

**3. A company needs to provide API access to its partners and wants to track usage and control access on a per-partner basis. Which API Gateway feature, available only with REST APIs, allows for this?**

- A. Throttling
- B. Custom Domains
- C. API Keys and Usage Plans
- D. Lambda Authorizers

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** API Keys and Usage Plans are features specific to the REST API type. You can generate unique API keys for each partner, associate them with a usage plan that defines throttling limits and request quotas, and then track their usage. This is not available with HTTP APIs.

</details>

---

**4. What is the key difference between an HTTP API and a WebSocket API?**

- A. HTTP APIs are stateless, while WebSocket APIs are stateful.
- B. HTTP APIs are more expensive than WebSocket APIs.
- C. HTTP APIs support Lambda integration, while WebSocket APIs do not.
- D. HTTP APIs are for internal use, while WebSocket APIs are for public use.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** This is the fundamental difference. An HTTP API follows the standard request/response model, where each request is independent and the connection is closed (stateless). A WebSocket API establishes a persistent, long-lived connection between the client and server, allowing both to send messages at any time (stateful).

</details>

---

**5. A REST API is configured with caching enabled. When a user makes a `GET` request, what is the process?**

- A. The request always goes directly to the backend integration.
- B. API Gateway checks its cache first. If the response is in the cache (a cache hit), it returns the cached response immediately without calling the backend.
- C. The cache only stores `POST` requests.
- D. The request is cached at the client's browser, not on API Gateway.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** API Gateway caching is designed to reduce the number of calls to your backend. For incoming requests, it checks its cache for a matching response. A cache hit results in a fast response directly from the gateway, improving latency and reducing load on your backend services.

</details>

---

**6. Which API Gateway type offers more advanced features like request validation, mocking, and integration with AWS WAF?**

- A. HTTP API
- B. WebSocket API
- C. REST API
- D. All API types offer these features.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The REST API is the original, feature-rich offering. It includes a comprehensive suite of API management tools, including request/response validation, mocking endpoints for testing, detailed transformation templates, and direct integration with AWS WAF for security. These advanced features are not available in the simpler HTTP API.

</details>

---

**7. An API Gateway is configured with a Lambda integration. What is the role of the API Gateway in this setup?**

- A. It runs the Lambda function's code.
- B. It acts as the "front door," receiving HTTP requests from clients and triggering the appropriate Lambda function to handle the business logic.
- C. It stores the Lambda function's deployment package.
- D. It monitors the Lambda function for errors.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** API Gateway is the entry point. It handles all the work of accepting HTTP connections, processing headers, authorizing requests, and then invoking the configured backend—in this case, an AWS Lambda function.

</details>

---

**8. What is a "deployment stage" in API Gateway?**

- A. The physical location of the API Gateway servers.
- B. A logical snapshot of your API configuration, often used to create different versions like `dev`, `test`, and `prod`.
- C. The underlying EC2 instance type used by the API.
- D. The type of API being deployed (HTTP, REST, or WebSocket).

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A deployment stage is like a named version or environment for your API. You can deploy your API configuration to a dev stage for testing and then later promote that same configuration to a prod stage for your production users. Each stage has its own unique invocation URL.

</details>

---

**9. When would you choose a REST API over an HTTP API?**

- A. When you need the lowest possible cost.
- B. When your primary goal is the lowest possible latency.
- C. When you need to support real-time, two-way communication.
- D. When you require features like API keys, request validation, and WAF integration.

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** You choose the REST API when you need its advanced feature set. If your API requires usage plans with API keys for third-party developers, or if you need to protect it with AWS WAF, the REST API is the required choice, as these features are not supported by the simpler HTTP API.

</details>

---

**10. In a WebSocket API, what is the purpose of the `$connect` route key?**

- A. It is invoked every time a client sends a message.
- B. It is invoked when a client first connects to the API.
- C. It is invoked when a client disconnects from the API.
- D. It is a default route for any messages that don't match other routes.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The $connect route is a special route key that is triggered only when a client establishes a new WebSocket connection. It is typically used to handle authorization logic and to register the new connectionId in a database like DynamoDB.

</details>

---

**11. An API Gateway needs to be accessible via a user-friendly domain name like `api.example.com` instead of the default execute-api URL. What feature enables this?**

- A. API Keys
- B. Deployment Stages
- C. Custom Domains
- D. Resource Policies

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Custom Domains feature in API Gateway allows you to associate your own registered domain name with your API. This requires creating an ACM SSL certificate and configuring a DNS record (typically a CNAME or Alias) to point your custom domain to the API Gateway endpoint.

</details>

---

**12. Which API Gateway integration type simply passes the entire request and response through to a backend HTTP endpoint with minimal modification?**

- A. Lambda Proxy Integration
- B. HTTP Proxy Integration
- C. AWS Service Integration
- D. Mock Integration

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An HTTP Proxy Integration is a lightweight integration where API Gateway passes the client's method request directly to a backend HTTP endpoint. It's a simple passthrough, in contrast to a non-proxy (custom) integration where you would have more control to modify the request before it's sent to the backend.

</details>

---

**13. A company wants to protect its REST API from aggressive clients and brute-force attacks by limiting the number of requests a client can make per second. What feature should they configure?**

- A. Caching
- B. Throttling and Usage Plans
- C. A Lambda authorizer
- D. A VPC Link

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** API Gateway's throttling feature is designed for this. You can set overall steady-state and burst limits for your API. With a REST API, you can use Usage Plans and API Keys to set even more granular throttling limits for specific clients.

</details>

---

**14. What are the three types of APIs you can create with Amazon API Gateway?**

- A. Public, Private, and Hybrid
- B. GraphQL, REST, and SOAP
- C. HTTP, REST, and WebSocket
- D. Basic, Standard, and Enterprise

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The three distinct product offerings under the API Gateway service are HTTP APIs, REST APIs, and WebSocket APIs.

</details>

---

**15. What is a "Lambda authorizer" (formerly custom authorizer) used for?**

- A. To grant the API Gateway permission to invoke a Lambda function.
- B. To use a Lambda function to perform custom authentication and authorization logic for API requests.
- C. To define the pricing model for the API.
- D. To transform the request body before sending it to the backend.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Lambda authorizer is a powerful security feature. It allows you to use your own Lambda function to control access to your API methods. API Gateway invokes this function with details from the incoming request (like a bearer token in a header). The function then returns an IAM policy that either allows or denies the request, providing a flexible, custom authorization mechanism.

</details>

---

**16. Which API type is generally recommended by AWS for new, non-complex HTTP APIs due to its cost and performance benefits?**

- A. REST API
- B. WebSocket API
- C. SOAP API
- D. HTTP API

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** For new serverless HTTP APIs that do not require the advanced features of the REST API, AWS recommends starting with the HTTP API. It provides a better price-performance ratio for common use cases.

</details>

---

**17. How does a WebSocket API handle messages from a client?**

- A. Each message is treated as a separate HTTP POST request.
- B. It uses "route keys" based on the message content to direct the message to a specific backend integration, like a Lambda function.
- C. All messages are sent to a single SQS queue.
- D. It forwards all messages to a single, pre-defined backend endpoint.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A WebSocket API uses a routing system. You define routes based on the content of the JSON message (e.g., if a message contains "action": "sendMessage"). When a message arrives, API Gateway matches its content to a defined route and invokes the corresponding backend integration.

</details>

---

**18. To make an API accessible only from within a specific VPC, what feature would you use?**

- A. A private API endpoint.
- B. API Gateway caching.
- C. A usage plan with a quota of zero.
- D. An AWS WAF rule.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** A private API endpoint is an endpoint that is only accessible from within your VPC via a VPC Interface Endpoint. This ensures that the API is not exposed to the public internet and can only be called by your internal resources.

</details>

---

**19. What is the key functional difference between a REST API and a WebSocket API?**

- A. REST APIs are for stateless request/response communication, while WebSocket APIs are for stateful, bidirectional communication.
- B. REST APIs use JSON, while WebSocket APIs use XML.
- C. REST APIs are more secure than WebSocket APIs.
- D. REST APIs can be cached, while WebSocket APIs cannot.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** This is the core difference in their communication model. A REST API follows the classic, stateless client-server model where the client makes a request and gets a response. A WebSocket API establishes a persistent connection, allowing both the client and the server to send messages to each other independently at any time.

</details>

---

**20. A company needs to connect their API Gateway to a service running on an EC2 instance inside their VPC without exposing the instance to the public internet. What should they use?**

- A. A VPC Link.
- B. A NAT Gateway.
- C. An Internet Gateway.
- D. A public Elastic IP on the EC2 instance.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** A VPC Link is a feature that allows your API Gateway to connect to private resources within your VPC (like an EC2 instance or an internal Network Load Balancer) without the need for those resources to have public IP addresses. It creates a private, secure connection between the API Gateway service and your VPC.

</details>

---

**21. You are choosing an API type for a new project. You know that in the future you will need to integrate with AWS WAF for security. Which API type should you choose?**

- A. HTTP API, and then upgrade it later.
- B. REST API.
- C. WebSocket API.
- D. Any type, as WAF works with all of them.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Direct integration with AWS WAF is a feature that is only available for the REST API type. If WAF is a future requirement, you should build with the REST API from the start to avoid a difficult migration later.

</details>

---

**22. Which two features are available for REST APIs but NOT for HTTP APIs? (Select TWO)**

- A. Lambda proxy integration
- B. Custom Domains
- C. API Caching
- D. AWS WAF Integration
- E. JWT Authorizers

<details>
<summary>View Answer</summary>

**Answers: C and D**

**Explanation:** The key differentiators are the advanced features. API Caching (C) and AWS WAF Integration (D) are both powerful features that are exclusively available for the REST API type. Both API types support Lambda integration, custom domains, and JWT authorizers.

</details>
