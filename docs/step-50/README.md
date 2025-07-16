# Step 50: ECS, Fargate, Task 

## 🎯 Objective
- [x] Master: **ECS, Fargate, Task**

## 📘 Notes

### **Deep Dive: ECS, Fargate, & Task**

Amazon Elastic Container Service (ECS) is a highly scalable, high-performance container orchestration service that supports Docker containers. It allows you to easily run, stop, and manage containerized applications on AWS. Understanding the core components of ECS and its two different launch types is key.

---

### **1. Core ECS Components** 🧩

- **Container & Image:**
    - An **Image** is the blueprint for your application—a read-only template with instructions for creating a Docker container. You build an image and push it to a registry, like **Amazon Elastic Container Registry (ECR)**.
    - A **Container** is a runnable instance of an image.
- **Task Definition:**
    - This is the most fundamental component. A Task Definition is a JSON file that acts as a **blueprint for your application's tasks**. It does *not* run anything itself; it just defines the parameters.
    - It specifies things like:
        - The Docker image(s) to use for your containers.
        - The CPU and memory to allocate to each task.
        - The launch type to use (EC2 or Fargate).
        - Networking settings (e.g., port mappings).
        - The IAM role the task will assume (the "Task Role").
- **Task:**
    - A **Task** is a running instance of a Task Definition. It is the actual running container (or group of related containers) in your cluster. It is the smallest deployable unit in ECS.
- **Service:**
    - An ECS Service is a long-running process that **maintains a specified number** of tasks simultaneously. It is the "manager."
    - If a task fails its health check or stops for any reason, the ECS Service scheduler will automatically launch a replacement task to maintain the desired count.
    - It can also integrate with an Application Load Balancer to distribute traffic evenly across all the tasks it manages.
- **Cluster:**
    - A Cluster is a logical grouping of tasks or services. It's the environment where your tasks run. When you run tasks, you must specify which cluster they belong to. The underlying infrastructure of the cluster is defined by the launch type.

---

### **2. ECS Launch Types: EC2 vs. Fargate**

This is the most critical concept to understand. You have two choices for the compute infrastructure that will run your ECS Tasks.

- **EC2 Launch Type:** 🖥️ (You manage the servers)
    - **How it works:** You create a cluster, and then you are responsible for provisioning, managing, and scaling a fleet of **EC2 instances** that will be added to this cluster. Your ECS tasks are then placed onto these EC2 instances.
    - **Your Responsibility:** You manage the underlying EC2 instances. This includes selecting the instance type, patching the OS, managing security, and optimizing cluster utilization (a process called "bin packing").
    - **Control:** Provides maximum control. You can choose specific instance types (e.g., GPU-optimized instances), use specific AMIs, and have more granular control over networking.
    - **Use Case:**
        - When you need a high degree of control over the underlying instances.
        - For applications with very specific compute requirements (e.g., needs a GPU).
        - When you want to take advantage of Spot Instances or existing Reserved Instances to optimize costs.
- **AWS Fargate Launch Type:** 🤖 (Serverless)
    - **How it works:** Fargate is a serverless compute engine for containers. You simply package your application, specify the CPU and memory it needs, define networking and IAM policies, and Fargate launches and manages the containers for you.
    - **Your Responsibility:** You don't see or manage any EC2 instances. AWS handles all the underlying infrastructure management, patching, and scaling.
    - **Control:** Less control. You don't choose the instance type or manage the OS.
    - **Pricing:** You pay for the amount of vCPU and memory resources that your containerized application requests, billed by the second.
    - **Use Case:**
        - The simplest way to run containers on AWS.
        - For applications where you want to avoid managing servers.
        - Ideal for microservices, batch processing, and most general-purpose applications.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A development team wants to run their containerized application on AWS. They want the simplest possible solution and do not want to be responsible for managing or patching any underlying EC2 instances. Which ECS launch type should they choose?**

- A. EC2 Launch Type
- B. Fargate Launch Type
- C. Spot Instances
- D. A custom AMI.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the primary use case for AWS Fargate. Fargate is a "serverless" compute engine for containers, meaning AWS handles all the underlying infrastructure management, including provisioning, patching, and scaling the servers. The developer only needs to define their container and its resource requirements.
</details>
    

---

**2. What is the function of an ECS Task Definition?**

- A. It is a running container.
- B. It is a blueprint or JSON template that describes how a task and its containers should be launched.
- C. It is a logical grouping of EC2 instances.
- D. It maintains a desired number of running containers.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Task Definition is a static blueprint. It does not run anything itself but contains all the necessary configuration metadata, such as the Docker image to use, CPU/memory allocation, port mappings, and environment variables. A "Task" is the actual running instance of a Task Definition.
</details>
    

---

**3. A company is deploying a machine learning application that requires powerful GPUs to perform its work. They want to run this application as a container on ECS. Which launch type must they use?**

- A. Fargate, as it can scale automatically.
- B. EC2, because it allows them to choose specific GPU-optimized instance types for their cluster.
- C. Either Fargate or EC2 can be used.
- D. This is not possible on ECS.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The key requirement is a specific hardware type (GPU). Fargate does not provide this level of control; it abstracts away the underlying hardware. The EC2 launch type must be used because it allows you to explicitly choose the instance types (e.g., P3 or G4 instances) that are part of your cluster, giving you access to the required GPU hardware.
</details>
    

---

**4. An ECS Service is configured with a desired count of 3. One of the three running tasks fails its health check from the Application Load Balancer. What will the ECS Service do?**

- A. Nothing, it will wait for an administrator to fix the task.
- B. It will stop the remaining two healthy tasks.
- C. It will automatically detect the failed task, stop it, and launch a new replacement task to maintain the desired count of 3.
- D. It will send a notification to Amazon CloudWatch.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The primary responsibility of an ECS Service is to be a long-running manager that maintains the desired number of healthy tasks. If any task becomes unhealthy or stops, the service's scheduler will automatically launch a replacement based on the task definition to self-heal and return to the desired state.
</details>
    

---

**5. In the context of ECS, what is a "Task"?**

- A. The EC2 instance that runs the containers.
- B. A networking interface for a container.
- C. The Docker image stored in ECR.
- D. A running instance of a Task Definition, consisting of one or more containers.

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** A Task is the smallest deployable unit in ECS. It is the live, running instantiation of a Task Definition blueprint.
</details>
    

---

**6. When using the EC2 launch type for ECS, who is responsible for the security patching of the underlying EC2 instance's operating system?**

- A. AWS, as part of the managed ECS service.
- B. The customer.
- C. The container's application developer.
- D. AWS Fargate.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** When you choose the EC2 launch type, you are using a shared responsibility model. While AWS manages the ECS control plane, the customer is responsible for the EC2 instances themselves. This includes choosing the AMI, patching the OS, managing security groups, and handling any host-level security.
</details>
    

---

**7. A company wants to run a containerized application and wants to use their existing fleet of EC2 Spot Instances to reduce costs. Which ECS launch type supports this?**

- A. Fargate
- B. EC2
- C. Both Fargate and EC2
- D. This is configured in the Task Definition.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To utilize specific purchasing options like Spot Instances or existing Reserved Instances, you must have control over the underlying compute. This is only possible with the EC2 launch type, where you create and manage the fleet of instances that make up your cluster. Fargate abstracts this away and does not currently support a Spot pricing model directly.
</details>
    

---

**8. What is the function of an ECS Cluster?**

- A. It is a physical server that runs containers.
- B. It is a Docker image.
- C. It is a logical grouping of tasks or services within a specific region.
- D. It is a global load balancer for containers.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** An ECS Cluster is a logical namespace or boundary. It provides a grouping for your services and tasks. When you launch a task, you must specify which cluster it should run in. The cluster itself is regional.
</details>
    

---

**9. In an ECS Task Definition, what does the "Task Role" define?**

- A. The permissions that the EC2 instance has to pull images from ECR.
- B. The IAM permissions that the containers within the task have to call other AWS services.
- C. The IAM role used by the ECS service to launch new tasks.
- D. The permissions needed to create the Task Definition itself.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Task Role is a critical security feature. It is an IAM role that is granted to the running task itself. This allows the application code inside your container to make secure API calls to other AWS services (like S3, DynamoDB, or SQS) without needing to hard-code any credentials.
</details>
    

---

**10. How is billing handled for the AWS Fargate launch type?**

- A. You pay a flat monthly fee per cluster.
- B. You pay for the EC2 instances that Fargate launches in your account.
- C. You pay for the amount of vCPU and memory resources consumed by your tasks, billed per second.
- D. Fargate is a free service.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Fargate has a serverless pricing model. You are billed based on the vCPU and memory resources you request in your Task Definition, for the duration that your task is running. This allows you to pay only for the precise resources your application needs.
</details>
    

---

**11. Which of the following is defined within an ECS Task Definition? (Select TWO)**

- A. The desired number of tasks to run.
- B. The Docker image to be used.
- C. The scaling policies for the service.
- D. The CPU and memory allocations for the task.
- E. The VPC and subnets to launch into.

<details>
<summary>View Answer</summary>
<br>

**Answers: B and D**

**Explanation:** The Task Definition is the blueprint for the task itself. It defines which Docker image(s) to use (B) and the CPU and memory resources that should be allocated to it (D). The desired count (A) and scaling policies (C) are part of the ECS Service configuration. The VPC/subnets (E) are specified when you run the task or create the service.
</details>
    

---

**12. When should you choose the EC2 launch type over Fargate?**

- A. When you want the simplest possible deployment experience.
- B. When your application has specific compliance requirements that necessitate control over the host environment.
- C. When your application has a very small and predictable workload.
- D. When you want to avoid managing security groups.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The primary reason to choose the EC2 launch type is for control. If you have specific host-level requirements, such as needing to meet a compliance standard that requires certain OS configurations, or if you need to use a specific type of hardware like a GPU, you need the control that the EC2 model provides.
</details>
    

---

**13. What component of ECS ensures that if a task stops or fails, a new one is launched to replace it?**

- A. The ECS Task Definition
- B. The ECS Cluster
- C. The ECS Agent running on the host
- D. The ECS Service

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** The ECS Service acts as the long-running process manager. Its scheduler constantly works to maintain the "desired count" of tasks. If the actual running count drops below the desired count for any reason (like a task failure), the scheduler's reconciliation loop will launch a new task.
</details>
    

---

**14. A task definition specifies that a container's port 80 should be mapped to the host's port 8080. The task is launched using the Fargate launch type. How is this handled?**

- A. This is not possible with Fargate, as there is no host to map to.
- B. Fargate automatically handles the port mapping on the underlying managed infrastructure.
- C. You must manually configure the port mapping on a NAT Gateway.
- D. The launch will fail with a configuration error.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Even though you don't see the host with Fargate, the concept of port mapping still exists within the managed environment. Fargate handles this abstraction for you. You define the mapping in your task definition, and Fargate ensures traffic sent to the ENI's port 8080 is correctly routed to port 80 inside your container.
</details>
    

---

**15. What is the function of the "Task Execution Role" in ECS?**

- A. It grants the running application code permission to call other AWS services.
- B. It grants the ECS agent permission to pull container images from ECR and send logs to CloudWatch.
- C. It is the IAM role assumed by the EC2 instance itself.
- D. It is an IAM role used by the ECS service scheduler.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The Task Execution Role is used by the ECS infrastructure on your behalf before the task starts running. Its primary responsibilities are giving the ECS agent permission to pull the required container image from Amazon ECR and to publish application logs from your container to Amazon CloudWatch Logs.
</details>
    

---

**16. The process of placing containers onto EC2 instances to maximize the use of resources is often called:**

- A. Bin Packing
- B. Load Balancing
- C. Right Sizing
- D. Containerization

<details>
<summary>View Answer</summary>
<br>

**Answer: A**

**Explanation:** Bin Packing is the term used to describe the challenge of efficiently placing containers (with varying CPU/memory requirements) onto a fleet of EC2 instances to minimize wasted resources. This is a key operational responsibility when using the EC2 launch type, and a complexity that is completely handled for you by Fargate.
</details>
    

---

**17. Which ECS launch type would be more cost-effective for a very large, stable application that runs 24/7, where the company has already purchased a large number of EC2 Reserved Instances?**

- A. Fargate, because it is serverless.
- B. EC2, because it can take advantage of the existing Reserved Instance discounts.
- C. Both would cost exactly the same.
- D. Neither, ECS cannot use Reserved Instances.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** If a company has already made a commitment to EC2 Reserved Instances, the most cost-effective strategy is to use the EC2 launch type. This allows them to place their ECS tasks on those reserved instances, fully utilizing the discounts they have already paid for.
</details>
    

---

**18. Can a single ECS Task contain multiple containers?**

- A. No, a task is always a single container.
- B. Yes, a task can be composed of multiple containers that are deployed and managed as a single unit.
- C. Only when using the EC2 launch type.
- D. Only when using the Fargate launch type.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Yes, this is a common pattern. A single Task can include multiple containers. This is often used for "sidecar" patterns, where a primary application container runs alongside a secondary container for logging, monitoring, or service mesh proxying. These containers share a network namespace and can communicate easily.
</details>
    

---


**19. What is Amazon ECR?**

- A. A container orchestration service.
- B. A fully-managed Docker container registry service.
- C. A serverless compute engine for containers.
- D. A service for managing EC2 instance configurations.

<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Amazon Elastic Container Registry (ECR) is a managed service where you store, manage, and deploy your Docker container images. It is the AWS-native equivalent of Docker Hub. Your ECS Task Definitions will typically point to an image stored in an ECR repository.
</details>

---


**20. A developer needs to pass a database password to their container at runtime without hard-coding it in the Docker image or task definition. What is a secure way to do this?**

- A. Store the password in a text file in a public S3 bucket.
- B. Pass the password as an environment variable directly in the task definition.
- C. Store the password in AWS Secrets Manager and reference it securely from the task definition.
- D. SSH into the container after it starts and set the password.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The best practice for handling secrets is to use a dedicated secrets management service. You would store the password in AWS Secrets Manager (or SSM Parameter Store SecureString). Then, in your task definition, you can configure it to securely fetch the secret and inject it into the container as an environment variable at runtime. This requires giving the task's Task Execution Role permission to access the secret.
</details>

---


**21. When comparing the two ECS launch types, which one offers more granular control over networking and the underlying host?**

- A. Fargate
- B. EC2
- C. Both offer the same level of control.
- D. Neither offers network control.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The EC2 launch type provides significantly more control. You are responsible for the EC2 instances, so you can choose the instance type, the AMI, configure host-level networking, install custom monitoring agents, and precisely manage how container network interfaces are attached.
</details>

---

**22. An ECS Service is connected to an Application Load Balancer. The service's desired count is 5. How does the ALB know which tasks to send traffic to?**

- A. The ALB sends traffic to all 5 tasks equally.
- B. The ECS service automatically registers the IP addresses and ports of its healthy tasks with the ALB's target group.
- C. You must manually update the ALB's target group every time a task is replaced.
- D. The ALB queries the ECS Cluster to get a list of running tasks.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The integration between the ECS Service and the ALB is seamless. When the service launches a new task, it automatically registers that task's network information with the specified target group. When a task becomes unhealthy or is terminated, the service automatically de-registers it. This ensures the ALB is always sending traffic only to the healthy, running tasks.
</details>