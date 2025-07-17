# Step 58: EventBridge vs CloudWatch Events

## 🎯 Objective

- [x] Master: **EventBridge vs CloudWatch Events**

## 📘 Notes


### **Deep Dive: EventBridge vs. CloudWatch Events**

For the SAA-C03 exam, the most important thing to know is that **Amazon EventBridge is the new name and the evolution of Amazon CloudWatch Events**. They share the same underlying service, API, and infrastructure. EventBridge simply adds new, powerful features on top of what CloudWatch Events provided. Think of EventBridge as "CloudWatch Events++".

---

### **1. Core Concepts (Shared by both)**

- **What it is:** A **serverless event bus** service that makes it easy to connect applications together using data from your own apps, integrated Software-as-a-Service (SaaS) applications, and AWS services.
- **Event:** An "event" is a signal that a system's state has changed. For example, an EC2 instance changing to the `stopped` state is an event. Events are represented as JSON objects.
- **Rule:** A rule matches incoming events and routes them to targets for processing. A rule can filter events based on their content (the event pattern). For example, you can create a rule that only matches events where `source` is `aws.ec2` and `detail-type` is `EC2 Instance State-change Notification`.
- **Target:** A target is a resource or endpoint that processes an event. When a rule matches an event, EventBridge sends the event to the rule's specified target(s). Common targets include:
    - AWS Lambda functions
    - SQS queues
    - SNS topics
    - ECS tasks
    - Step Functions state machines
- **Scheduling:** Both services can be used to generate events on a recurring schedule using **cron** or **rate** expressions (e.g., "run every 5 minutes" or "run at 10 AM GMT every Monday"). This is a primary way to trigger scheduled Lambda functions.

---

### **2. What EventBridge Adds (The Key Differences)**

EventBridge expands on the capabilities of CloudWatch Events with several key features:

- **Multiple Event Buses:**
    - **Default Event Bus:** This is the bus that automatically receives all events from AWS services (this is what CloudWatch Events used).
    - **Custom Event Bus:** You can create your own event buses to receive events from your own custom applications. This is useful for separating concerns and not cluttering the default bus.
    - **SaaS Partner Event Bus:** This is a major feature. It allows you to integrate your AWS account with third-party SaaS providers like Zendesk, Datadog, or Shopify. The partner can send events from their service to a partner event bus in your account, which you can then process just like any other event.
- **Schema Registry & Discovery:**
    - EventBridge can automatically **discover** the structure (schema) of events being sent to your event bus and store them in a **Schema Registry**.
    - This allows you to download code bindings for the event structure directly into your IDE, making it much easier and less error-prone to write code that consumes these events.
- **Pipes:** A newer feature that provides a simple, reliable, and cost-effective way to create point-to-point integrations between event producers and consumers. It allows you to filter and enrich events before they reach the target.

---

### **Summary: Why the Change?**

CloudWatch Events was primarily focused on events *from within* AWS services. Amazon EventBridge broadens this scope to be a central eventing hub for your own applications and third-party SaaS applications as well, making it a true enterprise-wide event bus service.

**For the exam, if you see a question about routing events from SaaS partners like Datadog or Zendesk, or about using a central event bus for your own custom applications, the answer is always EventBridge.**

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company uses a third-party SaaS provider for customer support (e.g., Zendesk). They want to trigger a Lambda function in their AWS account every time a new support ticket is created in the SaaS application. Which AWS service is designed for this type of cross-service integration?**

- A. Amazon SQS
- B. Amazon SNS
- C. Amazon EventBridge
- D. AWS Step Functions

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a primary use case for Amazon EventBridge. It has a specific feature for integrating with SaaS partners. The partner application (Zendesk) publishes events to a partner event bus in your AWS account, and you can then create a rule on that bus to route those events to a target like a Lambda function.
</details>
    

---

**2. What is the fundamental component of Amazon EventBridge that receives events?**

- A. A Rule
- B. A Target
- C. An Event Bus
- D. A Schema

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The Event Bus is the central router that receives all events. AWS services send events to the default event bus. Your own applications can send events to a custom event bus. You then create rules on the event bus to filter and route events to targets.
</details>
    

---

**3. A developer needs to run a Lambda function every 15 minutes. What is the most appropriate, serverless way to configure this trigger?**

- A. A cron job on a small EC2 instance.
- B. An EventBridge rule with a scheduled `rate(15 minutes)` expression.
- C. A `while` loop with a `sleep` command inside the Lambda function.
- D. A message timer in an SQS queue.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A core feature of EventBridge (and its predecessor, CloudWatch Events) is the ability to generate events on a schedule. You can create a rule that triggers on a fixed rate (e.g., every X minutes/hours) or a specific time (using a cron expression) and set your Lambda function as the target.
</details>
    

---

**4. How is Amazon EventBridge related to Amazon CloudWatch Events?**

- A. They are competing services from AWS.
- B. EventBridge is a feature within CloudWatch Events.
- C. EventBridge is the new name for CloudWatch Events and expands its features with things like custom and partner event buses.
- D. CloudWatch Events is for logging, while EventBridge is for messaging.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** EventBridge is the evolution of CloudWatch Events. It uses the same API and underlying infrastructure but adds significant new capabilities, making it the preferred service for all event-driven architectures.
</details>
    

---

**5. An event rule is configured with an "event pattern." What is the purpose of this pattern?**

- A. To define the schedule on which the rule should run.
- B. To filter incoming events so that the rule only matches events with a specific structure or content.
- C. To define the IAM permissions for the rule's target.
- D. To specify the target that will process the event.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** An event pattern acts as a filter. It's a JSON object that specifies the attributes an incoming event must have to match the rule. For example, you can create a pattern that only matches if the source field is aws.ec2 and the detail.state field is terminated.
</details>
    

---

**6. Which of the following can be a target for an EventBridge rule? (Select TWO)**

- A. An AWS Lambda function
- B. An Amazon S3 bucket
- C. An SQS queue
- D. An IAM user
- E. An EBS volume

<details>
<summary>View Answer</summary>
<br>

**Answers: A and C**

**Explanation:** EventBridge can route events to a wide variety of targets. The most common are AWS Lambda functions (for custom processing) and SQS queues (for durable, asynchronous processing). Other valid targets include SNS topics, Step Functions, and ECS tasks. You cannot target an S3 bucket directly (though a Lambda function could write to one) or an IAM user.
</details>
    

---

**7. A company wants to build an event-driven architecture for its own custom applications. They want to ensure that events from their "billing" application are kept separate from events from their "inventory" application. What EventBridge feature should they use?**

- A. The default event bus with complex filter rules.
- B. A separate custom event bus for each application (billing and inventory).
- C. AWS CloudTrail.
- D. Two separate AWS accounts.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The ability to create custom event buses is a key feature of EventBridge. This allows you to create logical separation for your own application's events. The billing application would publish all its events to the billing-bus, and the inventory application would publish to the inventory-bus, keeping the event streams completely separate.
</details>
    

---

**8. What is the EventBridge Schema Registry?**

- A. A registry of all IAM roles that can be assumed by EventBridge.
- B. A collection of event structures (schemas) that allows you to easily discover the format of events and generate code bindings for them.
- C. A database that stores all events for long-term archival.
- D. The DNS registry used by EventBridge to find targets.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Schema Registry solves a common problem in event-driven architectures: knowing the exact structure of an event. By discovering and storing schemas, it allows developers to reliably write consumer code that won't break if the event structure changes, and it enables features like code generation in IDEs.
</details>
    

---

**9. Which of the following is an example of an event that would be automatically sent to the default event bus?**

- A. A custom event published by an on-premises application.
- B. An EC2 instance changing its state from `running` to `stopped`.
- C. An event from a third-party SaaS partner like Datadog.
- D. A scheduled event from a custom rule.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The default event bus automatically receives events generated by AWS services. An EC2 instance state change is a classic example of an AWS service event. Custom events (A) go to custom buses, SaaS events (C) go to partner buses, and scheduled events (D) are generated by EventBridge itself.
</details>
    

---

**10. How does EventBridge compare to SNS for building event-driven architectures?**

- A. SNS is more suitable for complex event filtering based on message content.
- B. EventBridge is a more powerful and flexible service that supports advanced filtering, multiple buses, and SaaS integrations.
- C. SNS can have Lambda functions as a target, while EventBridge cannot.
- D. EventBridge is push-based, while SNS is pull-based.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** While SNS is excellent for simple fan-out notifications, EventBridge is a more capable enterprise-level event bus. Its main advantages are its advanced event pattern filtering (which is much more powerful than SNS filter policies), its ability to integrate with third-party SaaS applications, and the use of multiple buses to segregate event streams.
</details>
    

---

**11. A rule on the default event bus has the event pattern: `{"source": ["aws.ec2"]}`. Which event will this rule match?**

- A. An object being uploaded to an S3 bucket.
- B. A message being published to an SNS topic.
- C. An Auto Scaling group launching a new EC2 instance.
- D. A new item being written to a DynamoDB table.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The event pattern is filtering for any event where the source field is aws.ec2. When an Auto Scaling group launches a new instance, the EC2 service emits an "EC2 Instance State-change Notification" event, which has aws.ec2 as its source. All the other options have different sources (e.g., aws.s3, aws.sns).
</details>
    

---

**12. When you use EventBridge to trigger a Lambda function on a schedule, what is the "event source"?**

- A. The Lambda function itself.
- B. Amazon S3.
- C. Amazon EventBridge (acting as the scheduler).
- D. The AWS Management Console.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** For a scheduled invocation, Amazon EventBridge is both the scheduler and the event source. It generates a "Scheduled Event" JSON object at the specified time and sends it to the target Lambda function.
</details>
    

---

**13. What is the relationship between an event, a rule, and a target in EventBridge?**

- A. A target generates a rule, which filters an event.
- B. An event matches a rule, and the rule routes the event to a target.
- C. A rule generates an event, which is sent to a target.
- D. An event is sent to a target, which uses a rule to process it.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This describes the core flow: An event arrives at the event bus. The service evaluates all rules on that bus. If the event's structure matches a rule's event pattern, the rule is triggered, and the event is sent to that rule's configured target(s).
</details>
    

---

**14. Which feature is exclusive to EventBridge and not available in CloudWatch Events?**

- A. The ability to trigger a Lambda function on a schedule.
- B. The ability to receive events from AWS services like EC2 and S3.
- C. The ability to create custom event buses and integrate with SaaS partners.
- D. The ability to filter events based on their content.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The features that differentiate EventBridge are those that extend beyond the default AWS service events. The ability to create custom event buses for your own applications and to receive events from SaaS partners are the key capabilities that EventBridge added to the original CloudWatch Events service.
</details>
    

---

**15. A developer wants to make it easier to write consumer code for events coming from a new custom application. They want to avoid errors caused by typos in attribute names. Which EventBridge feature can help?**

- A. The default event bus.
- B. The Schema Registry and Discovery feature.
- C. EventBridge Pipes.
- D. A rule with a wildcard event pattern.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Schema Registry is designed to solve this problem. It can discover the structure of events and make that schema available. Developers can then use this schema in their IDEs (like VS Code or JetBrains) to download code bindings, which provides auto-completion and validation, preventing simple typos and making development faster and more reliable.
</details>
    

---

**16. What kind of expression is used to define a recurring schedule for an EventBridge rule?**

- A. A SQL query.
- B. A regular expression.
- C. A rate or cron expression.
- D. A JSONPath expression.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Scheduled rules use one of two formats: a rate expression (e.g., rate(5 minutes)) for a fixed frequency, or a standard cron expression (e.g., cron(0 18 ? * MON-FRI *)) for more complex schedules like "at 6 PM every weekday."
</details>
    

---

**17. Can a single EventBridge rule have multiple targets?**

- A. No, each rule can only have one target.
- B. Yes, a single rule can route a matching event to multiple targets (e.g., a Lambda function and an SQS queue simultaneously).
- C. Only if the targets are all Lambda functions.
- D. Only if the rule is on a custom event bus.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Yes, a single rule can have up to 5 targets. When an event matches the rule's pattern, EventBridge will send the event to all configured targets, allowing for parallel processing similar to an SNS fan-out.
</details>
    

---

**18. Which service would you use to build a loosely coupled, event-driven architecture where different microservices react to events happening in other parts of the system?**

- A. AWS Step Functions
- B. Amazon SQS
- C. Amazon EventBridge
- D. Application Load Balancer

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Amazon EventBridge is the central nervous system for event-driven architectures on AWS. It allows producer services to emit events without any knowledge of the consumer services. Consumer services can then subscribe to the events they care about via rules, creating a highly decoupled and scalable system.
</details>
    

---

**19. What is a "partner event source" in EventBridge?**

- A. An AWS service from a partner region.
- B. A third-party SaaS application (like Shopify, Datadog, etc.) that has been integrated with EventBridge to send events to your AWS account.
- C. A custom application running in a partner's AWS account.
- D. An open-source eventing standard.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A partner event source is an offering from a SaaS partner that allows you to subscribe to their event stream. For example, you can create a partner event source for your Zendesk account, which then creates a partner event bus in your AWS account where you can receive events like "ticket created" or "ticket updated."
</details>
    

---

**20. What is the default event bus in an AWS account?**

- A. A bus named "default" that is automatically created and receives events from all AWS services.
- B. A bus named "custom" where your own applications can send events.
- C. There is no default bus; you must create one.
- D. A bus that receives events only from Amazon CloudWatch.

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** Every AWS account has a default event bus. You do not need to create it. This bus automatically receives all events that are generated by AWS services operating in your account.
</details>
    

---

**21. An event comes into an event bus. There are two rules on the bus. Rule A's pattern matches the event. Rule B's pattern does not. What happens?**

- A. The event is sent to the targets of both Rule A and Rule B.
- B. The event is sent only to the target(s) of Rule A.
- C. The event is rejected because not all rules matched.
- D. The event is held until a rule is created that matches it.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** EventBridge evaluates an incoming event against all rules on the bus independently. The event is only sent to the targets of the specific rules whose event patterns it successfully matches. In this case, only Rule A's target(s) will be invoked.
</details>
    

---

**22. If you want your on-premises application to send events to be processed in your AWS cloud architecture, which EventBridge feature would you use?**

- A. The default event bus.
- B. A partner event bus.
- C. A custom event bus.
- D. EventBridge Pipes.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** You would create a custom event bus to act as an ingestion point for your own application's events. Your on-premises application would then use the AWS SDK to make a PutEvents API call, sending the event data to your custom event bus in the cloud.
</details>