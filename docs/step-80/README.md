# Step 80: Architecture Review

## 🎯 Objective
- [x] Master: **Architecture Review**

## 📘 Notes

Let's do an **Architectural Review Summary** instead. I'll provide a consolidated overview of the key architectural patterns we've covered, focusing on how the services fit together to solve common problems.

---
### **Architectural Review Summary**

Here are the core, resilient, and scalable patterns that are fundamental to the SAA-C03 exam:

#### 1. The Scalable & Highly Available Web Application
This is the most important pattern. It uses multiple services across multiple Availability Zones to eliminate single points of failure.
* **DNS:** **Route 53** directs users to the application.
* **CDN/Edge:** **CloudFront** caches static content (images, CSS, JS) globally for performance and acts as a security front door with **WAF** and **Shield**.
* **Load Balancing:** An **Application Load Balancer (ALB)** distributes incoming HTTP/HTTPS traffic across the web servers. It is deployed across multiple AZs.
* **Compute:** An **Auto Scaling Group (ASG)** manages a fleet of EC2 instances across multiple AZs. It provides elasticity (scaling out/in) and self-healing (replacing failed instances).
* **Session State:** The EC2 instances are **stateless**. User session data is offloaded to a fast, shared data store like **ElastiCache** or **DynamoDB**.
* **Database:** An **RDS** or **Aurora** database is deployed in a **Multi-AZ** configuration for fault tolerance and automatic failover. Read-heavy workloads are scaled using **Read Replicas**.

---

#### 2. The Serverless Data Processing Pipeline
This pattern is for event-driven architectures where code runs only in response to an event, requiring no server management.
* **Ingestion:** **S3** acts as the durable, scalable landing zone for raw data (e.g., images, logs).
* **Trigger:** **S3 Event Notifications** fire automatically when a new object is created.
* **Decoupling (Optional but Recommended):** The event is sent to an **SQS queue**. This acts as a buffer, increasing resilience and allowing the processing tier to handle spikes in traffic gracefully.
* **Compute:** **AWS Lambda** is triggered by the SQS queue (or directly by S3). It contains the business logic to process the data (e.g., resize an image, parse a log file).
* **Metadata & Results:** The processed output is typically stored in another **S3 bucket**. Metadata about the job is stored in **DynamoDB** for fast, indexed lookups.

---

#### 3. Core Principles Recap
* **High Availability vs. Fault Tolerance:** HA minimizes downtime (e.g., an ASG replacing an instance), while FT aims for zero downtime (e.g., RDS Multi-AZ failover).
* **Disaster Recovery (DR):** Strategies range from slow/cheap (**Backup and Restore**) to fast/expensive (**Multi-Site Active-Active**), with **Pilot Light** and **Warm Standby** in between. The choice is driven by your RTO and RPO.
* **Decoupling:** Use **SQS** for 1-to-1 message-based decoupling. Use **SNS** for 1-to-many (fan-out) notifications. Use **EventBridge** for advanced, enterprise-wide event routing from AWS, custom, and SaaS sources.
* **Security:** Apply security at all layers (defense in depth). Use IAM roles, encrypt everything (KMS), and protect the network perimeter (Security Groups, NACLs, WAF). Audit with **CloudTrail** and detect threats with **GuardDuty**.
* **Cost Optimization:** Use the right pricing model (**Savings Plans** for steady state, **Spot** for interruptible, **On-Demand** for unpredictable). Use serverless services and automation (like **S3 Lifecycle Policies**) to eliminate idle waste.

Let me know what you'd like to cover next!
