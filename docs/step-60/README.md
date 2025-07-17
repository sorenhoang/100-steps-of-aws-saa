# Step 60: EKS

## 🎯 Objective

- [x] Master: **EKS**

## 📘 Notes

### **Deep Dive: Amazon EKS (Elastic Kubernetes Service)**

Amazon EKS is a fully managed **Kubernetes** service. If your organization already uses Kubernetes or wants to adopt the open-source Kubernetes standard, EKS is the service that makes it easier to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane.

---

### **1. Kubernetes & EKS Core Concepts**

- **Kubernetes (K8s):** An open-source container orchestration system for automating software deployment, scaling, and management. It groups containers into logical units for easy management and discovery.
- **EKS Architecture (Control Plane vs. Data Plane):** This is the most important concept. EKS separates the architecture into two distinct parts:
    1. **The Control Plane (Managed by AWS):** This is the "brain" of the Kubernetes cluster. It consists of the Kubernetes master nodes that run the API server, scheduler, and controller manager. With EKS, **AWS provisions, scales, and manages this entire control plane for you** across multiple Availability Zones for high availability. You don't have access to these master nodes.
    2. **The Data Plane (Managed by You):** This is where your actual containers run. It consists of the **worker nodes**. You are responsible for provisioning and managing these nodes.

---

### **2. EKS Data Plane (Worker Node) Options**

You have several options for creating the data plane that will run your containers:

- **Self-Managed EC2 Nodes:**
    - **How it works:** You manually provision and configure EC2 instances, install the necessary software (Docker, kubelet), and then register them with the EKS control plane.
    - **Control:** Gives you the absolute maximum control over the worker node configuration.
    - **Responsibility:** You are responsible for everything: patching the OS, upgrading Kubernetes components, and managing scaling through custom Auto Scaling Groups. This is the most complex option.
- **EKS Managed Node Groups:**
    - **How it works:** This is the recommended and most common approach. You define a configuration (instance type, disk size, etc.), and EKS automatically provisions an Auto Scaling Group of EC2 instances for you.
    - **Control:** A good balance of control and automation.
    - **Responsibility:** EKS handles the provisioning, graceful draining of nodes during termination, and automated upgrades of the nodes with a single command. You are still responsible for choosing the instance types and managing scaling policies.
- **AWS Fargate with EKS:**
    - **How it works:** This is the **serverless** option. You define a "Fargate Profile" that specifies which pods should run on Fargate. When you deploy a pod that matches the profile, AWS automatically provisions the exact right amount of compute capacity to run it.
    - **Control:** You have no access to or control over the underlying nodes.
    - **Responsibility:** AWS manages everything. There are no EC2 instances for you to patch, secure, or scale.
    - **Use Case:** Ideal for applications where you want to eliminate server management, for spiky workloads, or for isolating specific workloads.

---

### **3. Key Kubernetes Objects in EKS**

- **Pod:** The smallest and simplest unit in the Kubernetes object model. A Pod represents a single instance of a running process in your cluster and contains one or more containers. Containers within the same pod share a network namespace and storage.
- **Node:** A worker machine in Kubernetes, which can be a virtual or physical machine. In EKS, this is an EC2 instance or a Fargate micro-VM.
- **Service:** An abstraction that defines a logical set of Pods and a policy by which to access them. For example, a `LoadBalancer` service in EKS will automatically provision an AWS Classic or Network Load Balancer to expose your pods to the internet.
- **Deployment:** A declarative way to manage a set of identical pods. The deployment controller ensures that a specified number of pod "replicas" are running at all times, handling failures and updates.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company has a large team of developers with extensive experience using the open-source Kubernetes platform. They want to run their containerized applications on AWS and prefer to use the standard, open-source Kubernetes APIs. Which AWS container service should they choose?**

- A. Amazon ECS with the EC2 launch type.
- B. Amazon EKS.
- C. AWS Fargate as a standalone service.
- D. AWS Lambda with container images.
- View Answer
    
    Answer: B
    
    Explanation: Amazon EKS is the managed service specifically for running Kubernetes. It provides a standard, conformant Kubernetes control plane, allowing teams to use their existing Kubernetes skills and tools (like kubectl) directly with the service.
    

---

**2. In the Amazon EKS shared responsibility model, what is AWS responsible for managing?**

- A. The security patching of the worker node operating systems.
- B. The networking configuration between pods.
- C. The availability and maintenance of the Kubernetes control plane.
- D. The IAM roles assigned to the pods.
- View Answer
    
    Answer: C
    
    Explanation: The primary value of EKS is that AWS manages the complex and critical Kubernetes control plane. This includes the master nodes, the etcd database, and the Kubernetes API server, ensuring it is patched, secure, and highly available across multiple AZs. The customer is responsible for the worker nodes (A).
    

---

**3. A development team wants to run their containers on EKS but wants to completely avoid managing, patching, or scaling any EC2 instances. Which data plane option should they use?**

- A. Self-managed EC2 nodes.
- B. EKS Managed Node Groups.
- C. AWS Fargate with EKS.
- D. Spot Instances.
- View Answer
    
    Answer: C
    
    Explanation: AWS Fargate is the serverless compute engine for containers. When used with EKS, it allows you to run your pods without provisioning or managing any worker nodes. AWS automatically provides the underlying compute for each pod, abstracting away all server management.
    

---

**4. What is the "control plane" in an Amazon EKS cluster?**

- A. The fleet of EC2 worker nodes that run the container pods.
- B. The set of master nodes and software (API server, scheduler, etc.) that manage the state of the cluster.
- C. The VPC and subnets where the cluster is deployed.
- D. The IAM roles and policies that grant permissions to the cluster.
- View Answer
    
    Answer: B
    
    Explanation: The control plane is the "brain" of the Kubernetes cluster. It's responsible for making global decisions about the cluster (like scheduling pods), as well as detecting and responding to cluster events. AWS manages this component for you in EKS.
    

---

**5. A company is using EKS with Managed Node Groups. They need to run a special data processing workload that requires GPU-enabled instances. How can they achieve this?**

- A. This is not possible with EKS.
- B. By using AWS Fargate with a GPU profile.
- C. By creating a new Managed Node Group and specifying a GPU-equipped instance type (e.g., a P3 or G4 instance).
- D. By configuring the EKS control plane to use GPUs.
- View Answer
    
    Answer: C
    
    Explanation: Managed Node Groups give you the flexibility to choose specific EC2 instance types. To run a GPU workload, you would create a node group specifically for this purpose and select an appropriate GPU-optimized instance type from the available EC2 families.
    

---

**6. What is the smallest deployable unit of computing that you can create and manage in Kubernetes?**

- A. A Container
- B. A Node
- C. A Pod
- D. A Service
- View Answer
    
    Answer: C
    
    Explanation: A Pod is the most basic building block in Kubernetes. It represents a single instance of an application and holds one or more tightly coupled containers that share resources like networking and storage.
    

---

**7. When comparing EKS and ECS, which statement is generally TRUE?**

- A. ECS is based on the open-source Kubernetes project, while EKS is proprietary to AWS.
- B. EKS provides a standard Kubernetes experience with more community support and portability, while ECS is an AWS-opinionated solution that is simpler to use.
- C. EKS is always cheaper than ECS.
- D. ECS can only run on EC2, while EKS can only run on Fargate.
- View Answer
    
    Answer: B
    
    Explanation: This is the core trade-off. EKS offers adherence to the open-source Kubernetes standard, which means greater portability and access to a vast ecosystem of community tools. ECS is an AWS-native solution that is often simpler and more tightly integrated with other AWS services, but it is not based on Kubernetes.
    

---

**8. What is the function of the `kubelet` process that runs on every EKS worker node?**

- A. It acts as the DNS server for the cluster.
- B. It is the agent that communicates with the control plane's API server to ensure containers described in PodSpecs are running and healthy.
- C. It provides the container runtime, such as Docker.
- D. It routes network traffic between pods.
- View Answer
    
    Answer: B
    
    Explanation: The kubelet is the primary agent on each node. It receives instructions from the Kubernetes control plane and is responsible for starting, stopping, and maintaining the lifecycle of the pods and containers assigned to its node.
    

---

**9. How do you grant permissions to pods running on EKS to access other AWS services like S3 or DynamoDB?**

- A. By hard-coding IAM user credentials into the container image.
- B. By using IAM Roles for Service Accounts (IRSA).
- C. By attaching an IAM role directly to the EKS cluster.
- D. All pods run with administrator privileges by default.
- View Answer
    
    Answer: B
    
    Explanation: The secure and recommended method is IAM Roles for Service Accounts (IRSA). This feature allows you to associate an IAM role with a Kubernetes service account. You then configure your pods to use that service account, which allows them to automatically receive temporary, secure credentials from the associated IAM role to make calls to other AWS services.
    

---

**10. When using EKS with Fargate, how are you billed?**

- A. For the number of EC2 instances running in the cluster.
- B. A flat monthly fee per cluster.
- C. For the amount of vCPU and memory resources consumed by your pods, calculated from the time the pod's image is pulled until the pod terminates.
- D. Based on the number of pods running, regardless of their size.
- View Answer
    
    Answer: C
    
    Explanation: Similar to ECS with Fargate, the pricing model is serverless. You pay for the vCPU and memory resources that your pods request, for the duration they are running. This removes the need to pay for entire EC2 instances that may be underutilized.
    

---

**11. What is an EKS "Managed Node Group"?**

- A. A serverless compute engine for EKS.
- B. An Auto Scaling Group of EC2 instances that is automatically provisioned and managed by EKS for running worker nodes.
- C. The EKS control plane.
- D. A group of pods that are managed by a single deployment.
- View Answer
    
    Answer: B
    
    Explanation: A Managed Node Group automates the provisioning and lifecycle management of EC2 worker nodes for an EKS cluster. It simplifies tasks like upgrading Kubernetes versions on the nodes and gracefully draining pods during termination.
    

---

**12. In Kubernetes, what is the role of a "Service" object?**

- A. To define the desired number of running pods.
- B. To provide a stable endpoint (a single IP address and DNS name) to access a logical set of pods.
- C. To run a one-time batch job.
- D. To define the container image and resource requests for a pod.
- View Answer
    
    Answer: B
    
    Explanation: Pods are ephemeral; they can be created and destroyed, and their IP addresses change. A Kubernetes Service provides a stable abstraction layer. It creates a single, persistent endpoint and automatically routes traffic to the healthy pods that match its selector, acting as an internal load balancer.
    

---

**13. Which of the following is a responsibility of the CUSTOMER when using Amazon EKS with Managed Node Groups?**

- A. Patching the Kubernetes control plane software.
- B. Ensuring the high availability of the Kubernetes API server.
- C. Deciding when to upgrade the Kubernetes version of the worker nodes.
- D. Replacing failed master nodes.
- View Answer
    
    Answer: C
    
    Explanation: While EKS automates the process of upgrading worker nodes in a managed node group, the customer is responsible for initiating that upgrade. AWS manages the control plane (A, B, D).
    

---

**14. When would a company choose EKS with self-managed EC2 nodes over managed node groups?**

- A. When they want the simplest possible setup.
- B. When they need to use a custom AMI for their worker nodes that is not the EKS-optimized AMI.
- C. When they want to use Fargate.
- D. When they want AWS to handle all OS patching automatically.
- View Answer
    
    Answer: B
    
    Explanation: The primary reason to use self-managed nodes is to have maximum control and customization. If you have a specific compliance or security requirement that necessitates a custom-built operating system image, you would need to use self-managed nodes, as managed node groups require the use of the EKS-optimized AMI.
    

---

**15. What component of the Kubernetes control plane is responsible for watching for newly created pods and assigning them to a node?**

- A. The API Server
- B. The etcd database
- C. The kubelet
- D. The Scheduler
- View Answer
    
    Answer: D
    
    Explanation: The Kubernetes Scheduler is the component that makes placement decisions. It watches for pods that don't have a node assigned and, based on resource requirements, affinity rules, and other constraints, chooses the best available node to run that pod on.
    

---

**16. How does AWS ensure the high availability of the EKS control plane?**

- A. By allowing customers to create and manage their own master nodes.
- B. By running the control plane infrastructure across multiple Availability Zones.
- C. By creating a daily snapshot of the control plane.
- D. By using a single, large EC2 instance for the master node.
- View Answer
    
    Answer: B
    
    Explanation: A key benefit of EKS is that AWS automatically runs the control plane components (API server, etcd) across at least three Availability Zones. This ensures that the cluster's management plane is resilient to an outage in a single AZ.
    

---

**17. What is the function of a Kubernetes "Deployment"?**

- A. To provide a stable network endpoint for a set of pods.
- B. To declare the desired state for a set of replica pods and manage their lifecycle, including updates and rollbacks.
- C. To orchestrate a multi-step workflow.
- D. To store container images.
- View Answer
    
    Answer: B
    
    Explanation: A Deployment is a higher-level object that manages a ReplicaSet. You tell the Deployment you want, for example, "3 replicas of my web server pod." The Deployment controller will then work to ensure that 3 healthy replicas are always running. It's the standard way to run stateless applications on Kubernetes.
    

---

**18. You need to run a containerized application on AWS. Your company has no prior experience with containers and wants the solution with the lowest learning curve and operational overhead. Which service would you recommend?**

- A. Amazon EKS with managed nodes.
- B. Amazon EKS with Fargate.
- C. Amazon ECS with Fargate.
- D. Self-managed Kubernetes on EC2.
- View Answer
    
    Answer: C
    
    Explanation: While EKS is powerful, Kubernetes itself has a steep learning curve. Amazon ECS is an AWS-opinionated orchestrator that is generally considered simpler to learn and operate. Combining ECS with Fargate provides the absolute simplest, fully serverless experience for running containers on AWS, making it ideal for teams new to containers.
    

---

**19. When using EKS with Fargate, how is networking handled for pods?**

- A. All pods share the networking of a single host EC2 instance.
- B. Each Fargate pod is given its own Elastic Network Interface (ENI) and runs in its own network isolation boundary.
- C. All pods are exposed directly to the public internet.
- D. Networking must be managed manually by the user.
- View Answer
    
    Answer: B
    
    Explanation: When you run a pod on Fargate, it gets its own dedicated ENI from the VPC subnet you specified. This provides strong network isolation at the pod level, as each pod has its own IP address and can have its own security group rules applied via the ENI.
    

---

**20. A container running in an EKS pod needs to access a secret stored in AWS Secrets Manager. What is the recommended approach?**

- A. Mount the secret as a file into the pod using the AWS Secrets and Configuration Provider.
- B. Hard-code the secret value into the container's environment variables in the pod spec.
- C. Store the secret in the Docker container image.
- D. Allow public access to the secret.
- View Answer
    
    Answer: A
    
    Explanation: The best practice is to avoid exposing secrets directly as environment variables. The recommended approach is to use a provider like the AWS Secrets and Configuration Provider (for the CSI driver) or the Secrets Store CSI Driver. This allows you to mount secrets from Secrets Manager or SSM Parameter Store as files into the pod's filesystem, which is more secure than environment variables.
    

---

**21. What is the primary benefit of using EKS Managed Node Groups over self-managed nodes?**

- A. It allows the use of custom AMIs.
- B. It automates the provisioning and lifecycle management of worker nodes, including version upgrades and graceful termination.
- C. It provides a serverless experience.
- D. It is the only way to use Spot Instances.
- View Answer
    
    Answer: B
    
    Explanation: Managed Node Groups significantly reduce the operational burden of running the data plane. EKS handles the creation of the Auto Scaling Group and provides simplified, often one-click, commands to perform complex tasks like upgrading the kubelet version across the entire fleet or safely draining nodes before termination.
    

---

**22. Which part of the EKS architecture are you, the customer, responsible for paying for directly? (Select TWO)**

- A. The Kubernetes master nodes.
- B. The EC2 worker nodes or Fargate resources that run your pods.
- C. An hourly fee for the EKS control plane.
- D. The `etcd` database instances.
- E. The `kubectl` command-line tool.
- View Answer
    
    Answers: B and C
    
    Explanation: The EKS pricing model has two main components: 1) A flat hourly fee for the managed control plane itself (C). 2) The cost of the data plane resources you consume, which is either the cost of the EC2 instances in your node groups or the vCPU/memory cost for your Fargate pods (B).