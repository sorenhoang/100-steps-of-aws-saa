# Step 36: VPC Endpoints – Gateway & Interface

## 🎯 Objective

- [x] Master: **VPC Endpoints – Gateway & Interface**

## 📘 Notes

### **Deep Dive: VPC Endpoints – Gateway & Interface**

By default, when an EC2 instance in your VPC communicates with an AWS service like S3, that traffic is routed over the public internet. VPC Endpoints solve this by creating a private, secure connection between your VPC and supported AWS services, ensuring traffic never leaves the Amazon network. This is fundamental for security and can also improve performance.

---

### **1. Gateway Endpoints** ⛩️

A Gateway Endpoint is a gateway that you specify as a target for a route in your route table. It acts as a private entry point to the supported service.

- **Architecture:** It functions as a routing target. You create the endpoint and then add a route to your subnet's route table that points the service's prefix list (e.g., `pl-xxxxxxxx` for S3) to the gateway endpoint's ID (e.g., `vpce-xxxxxxxx`).
- **Supported Services:** This is a critical point to memorize. Gateway Endpoints only support **Amazon S3** and **Amazon DynamoDB**.
- **Access Control:** You can use a **VPC Endpoint Policy** (a resource-based policy) attached to the endpoint to control which actions or resources can be accessed through it.
- **Billing:** There are no additional charges for using Gateway Endpoints.
- **Connectivity:** Generally accessible only from within the VPC where it is created.

---

### **2. Interface Endpoints (Powered by AWS PrivateLink)** 🔌

An Interface Endpoint is an **Elastic Network Interface (ENI)** with a private IP address from your VPC's subnet. This ENI serves as a private entry point for traffic destined for the supported service.

- **Architecture:** It creates a network interface directly within your subnet. Your applications connect to the service by sending traffic to the private IP address of this ENI.
- **Supported Services:** Supports a vast number of AWS services, including SQS, SNS, Kinesis, KMS, SageMaker, Systems Manager, and many more. It's the standard endpoint type for most services.
- **Access Control:**
    - **Security Groups:** You can attach security groups to the endpoint's ENI to control which resources *within your VPC* can send traffic to the endpoint.
    - **VPC Endpoint Policy:** You can also attach an endpoint policy for more granular control over the API actions allowed through the endpoint.
- **Billing:** You are billed an hourly charge for each Interface Endpoint you provision, plus a per-GB data processing charge.
- **Connectivity:** Because it has a private IP address, an Interface Endpoint can be accessed from on-premises networks connected via **AWS Direct Connect** or a **VPN**.

---

### **Summary of Key Differences**

| Feature | Gateway Endpoint | Interface Endpoint (PrivateLink) |
| --- | --- | --- |
| **Architecture** | A route table target | An ENI in your subnet |
| **Services** | **S3 & DynamoDB only** | **Most other AWS services** |
| **Access Control** | Endpoint Policy | Security Groups & Endpoint Policy |
| **On-Premises Access** | No | **Yes (via DX/VPN)** |
| **Billing** | Free | Hourly and data processing fees |

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. An application running on EC2 instances in a private subnet needs to make API calls to Amazon SQS. To enhance security, all traffic must remain on the AWS private network and not traverse the public internet. What should be created?**

- A. A VPC Gateway Endpoint for Amazon SQS.
- A VPC Interface Endpoint for Amazon SQS.
- C. A NAT Gateway.
- D. An Internet Gateway.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Amazon SQS is supported by Interface Endpoints. A Gateway Endpoint (A) only supports S3 and DynamoDB. Creating an Interface Endpoint for SQS will place a network interface (ENI) in the private subnet, allowing the EC2 instances to communicate with the SQS service privately.
</details>
---

**2. What is the key architectural difference between a Gateway Endpoint and an Interface Endpoint?**

- A. A Gateway Endpoint is a route table target, while an Interface Endpoint is an ENI with a private IP in a subnet.
- B. A Gateway Endpoint is stateful, while an Interface Endpoint is stateless.
- C. A Gateway Endpoint is regional, while an Interface Endpoint is zonal.
- D. A Gateway Endpoint uses a security group, while an Interface Endpoint uses an endpoint policy.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** This is the fundamental difference. A Gateway Endpoint works by modifying routing within the VPC. An Interface Endpoint works by creating an actual network resource (an ENI) within your VPC's address space that you can route traffic to.
</details>
---

**3. A company wants to provide its on-premises data center with private access to the AWS Systems Manager (SSM) API over an existing Direct Connect link. Which AWS networking component enables this?**

- A. A VPC Gateway Endpoint.
- B. An AWS Storage Gateway.
- C. A VPC Interface Endpoint.
- D. A NAT Gateway.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Because an Interface Endpoint has a private IP address within your VPC, it can be reached from on-premises networks that are connected via Direct Connect or VPN. You would create an Interface Endpoint for the SSM service, and then configure your on-premises DNS or routing to direct SSM API calls to the endpoint's private IP.
</details>
---

**4. A solutions architect needs to allow EC2 instances in a private subnet to access Amazon DynamoDB without going over the internet. Which VPC Endpoint type should be used?**

- A. Interface Endpoint
- B. NAT Endpoint
- C. Gateway Endpoint
- D. DynamoDB does not support VPC Endpoints.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** Gateway Endpoints support only two services: Amazon S3 and Amazon DynamoDB. This is a critical fact to memorize for the exam. Therefore, a Gateway Endpoint is the correct choice.
</details>
---

**5. Which security mechanism can be used to control which resources *within* a VPC can send traffic to a VPC Interface Endpoint?**

- A. An IAM Policy
- B. A Network ACL
- C. A Security Group attached to the endpoint's ENI
- D. An AWS WAF Web ACL
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A key feature of Interface Endpoints is that they create an ENI in your subnet. Like any other ENI, you can attach Security Groups to it. This allows you to create rules that, for example, only allow traffic from the security group of your application servers, providing a layer of security within your VPC.
</details>
---

**6. A VPC Endpoint Policy is best described as what kind of policy?**

- A. An identity-based policy.
- B. A resource-based policy.
- C. A permissions boundary.
- D. A trust policy.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** A VPC Endpoint Policy is a resource-based policy because it is attached directly to a resource—the endpoint itself. It defines who can use that specific endpoint and what they can do through it.
</details>
---

**7. An EC2 instance has an IAM role that allows full access (`s3:*`) to all S3 buckets. The instance is in a VPC with a Gateway Endpoint for S3. The endpoint has a policy attached that only allows `s3:GetObject` on `my-secure-bucket`. What can the instance do via the endpoint?**

- A. It can perform any S3 action on any bucket.
- B. It can only perform `s3:GetObject` on `my-secure-bucket`.
- C. It cannot access S3 at all because the policies conflict.
- D. It can perform any action, but only on `my-secure-bucket`.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The effective permissions are the intersection of all applicable policies. The IAM policy is very permissive, but the VPC Endpoint Policy acts as a filter. Since the endpoint policy only allows s3:GetObject on my-secure-bucket, that is the only action that will be permitted for traffic passing through that endpoint.
</details>
---

**8. Which VPC Endpoint type is associated with hourly and data processing charges?**

- A. Gateway Endpoint
- B. Interface Endpoint
- C. Both have the same pricing model.
- D. Neither, VPC Endpoints are free.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Interface Endpoints incur an hourly charge for each endpoint provisioned in each AZ, plus a per-gigabyte charge for data processed through them. Gateway Endpoints are free.
</details>
---

**9. A company wants to ensure that all traffic from their VPC to their S3 buckets stays on the AWS private network. They also want to lock down their S3 buckets so they can only be accessed from within their VPC. How can this be achieved?**

- A. Create a Gateway Endpoint for S3 and modify the S3 bucket policies to only allow access from the VPC's CIDR range.
- B. Create a Gateway Endpoint for S3 and modify the S3 bucket policies to only allow access from the VPC Endpoint ID.
- C. Use a NAT Gateway to route all traffic to S3.
- D. Use an Application Load Balancer as a proxy to S3.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a common security pattern. You create a Gateway Endpoint to keep the traffic private. Then, you use the S3 bucket policy to enforce this. The bucket policy can use a condition key like aws:sourceVpce to specify that requests are only allowed if they originate from that specific VPC Endpoint, effectively locking the bucket to your VPC.
</details>
---

**10. What is the underlying technology that powers VPC Interface Endpoints and allows private connectivity to services?**

- A. VPC Peering
- B. AWS Direct Connect
- C. AWS PrivateLink
- D. AWS Transit Gateway
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** AWS PrivateLink is the technology that enables you to create Interface Endpoints. It provides private connectivity between VPCs, AWS services, and your on-premises networks securely on the Amazon network.
</details>
---

**11. An EC2 instance needs to make private API calls to Amazon Kinesis Data Streams. Which endpoint type is required?**

- A. Gateway Endpoint
- B. Interface Endpoint
- C. Kinesis does not support VPC Endpoints.
- D. A NAT Gateway must be used.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Since Kinesis is not S3 or DynamoDB, it requires an Interface Endpoint for private connectivity.
</details>
---

**12. When you enable the "Private DNS" option for an Interface Endpoint, what does AWS do?**

- A. It creates a public DNS record that resolves to the endpoint's private IP.
- B. It allows your VPC's DNS resolver to resolve the service's public DNS name to the private IP address of the endpoint's ENI.
- C. It assigns an Elastic IP to the endpoint's ENI.
- D. It configures the endpoint to only accept DNS traffic.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The Private DNS feature makes using the endpoint seamless. Your applications can continue to use the standard public DNS hostname for the service (e.g., kms.us-east-1.amazonaws.com). The VPC's internal DNS resolver intercepts this request and resolves it to the private IP of your interface endpoint, automatically directing traffic to the private connection without any code changes.
</details>
---

**13. A Gateway Endpoint is created for S3. How does traffic get directed to the endpoint?**

- A. By attaching a security group to the endpoint.
- B. By associating an Elastic IP with the endpoint.
- C. By modifying the subnet's route table to point to the endpoint.
- D. By configuring the application to use a special S3 endpoint URL.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** A Gateway Endpoint works at the routing layer. After creating it, you must go into the route table of the subnets that need to use it and add a route. This route will have a destination of the S3 service prefix list and a target of the gateway endpoint's ID.
</details>
---

**14. Can an on-premises server connected via a VPN access an AWS service through a VPC Gateway Endpoint?**

- A. Yes, by routing traffic over the VPN.
- B. No, Gateway Endpoints are not accessible from on-premises networks.
- C. Yes, but only for DynamoDB.
- D. Yes, but only if the VPN is attached to a Transit Gateway.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Gateway Endpoints do not have a private IP address and work via route table modifications within the VPC. Therefore, they cannot be accessed from on-premises networks over a VPN or Direct Connect. To provide on-premises access, you must use an Interface Endpoint.
</details>
---

**15. What are the only two services supported by VPC Gateway Endpoints?**

- A. S3 and SQS
- B. S3 and DynamoDB
- C. DynamoDB and Kinesis
- D. S3 and EC2
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** You must memorize this fact for the exam: Gateway Endpoints support Amazon S3 and Amazon DynamoDB only.
</details>
---

**16. An application in a private subnet is trying to access DynamoDB, but the connection is timing out. A Gateway Endpoint for DynamoDB exists and is correctly configured in the subnet's route table. The NACL allows all traffic. What is the most likely cause of the failure?**

- A. The EC2 instance's security group does not have an outbound rule allowing traffic to the DynamoDB service.
- B. The DynamoDB table is in a different region.
- C. The Gateway Endpoint is in a different Availability Zone.
- D. The IAM role on the instance does not have DynamoDB permissions.
<details>
<summary>View Answer</summary>

**Answer:** A

**Explanation:** Even though the route exists, the traffic must still be permitted by the instance's own firewall, the Security Group. The security group needs an outbound rule that allows TCP traffic on port 443 (for HTTPS) to the destination of the DynamoDB service prefix list. If this rule is missing, the connection will be blocked.
</details>
---

**17. A company wants to provide private access to their own custom application, hosted in their VPC, to a customer in another VPC. Which technology enables this "service provider" model?**

- A. VPC Peering
- B. AWS PrivateLink and a Network Load Balancer
- C. An Internet Gateway
- D. AWS DataSync
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** AWS PrivateLink is the technology that allows you to offer a service privately to other VPCs. You, as the service provider, would place your application behind a Network Load Balancer and create a "VPC endpoint service." The customer then creates an Interface Endpoint in their own VPC to connect to your service, all without the traffic ever leaving the AWS network.
</details>
---

**18. Which endpoint type is generally more expensive?**

- A. Gateway Endpoint
- B. Interface Endpoint
- C. Both cost the same.
- D. Pricing is based on the service, not the endpoint type.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** Interface Endpoints have an hourly provisioning charge and a data processing charge. Gateway Endpoints are free. Therefore, Interface Endpoints are the more expensive option.
</details>
---

**19. An EC2 instance has an IAM policy that denies all S3 actions. The instance is in a VPC with a Gateway Endpoint for S3 whose endpoint policy allows `s3:GetObject`. What can the instance do?**

- A. It can perform `s3:GetObject`.
- B. It can perform no S3 actions.
- C. It can perform all S3 actions.
- D. The outcome is unpredictable.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** An explicit Deny in any policy always wins. Since the instance's IAM policy has an explicit Deny, the request is blocked before it even reaches the endpoint. The Allow in the endpoint policy is irrelevant.
</details>
---

**20. A solutions architect has created an Interface Endpoint for Systems Manager and attached a security group to it. What is the purpose of this security group?**

- A. To control which IAM users can use the endpoint.
- B. To control which EC2 instances (by referencing their own security groups) are allowed to send traffic to the endpoint's ENI.
- C. To filter traffic leaving the endpoint and going to the AWS service.
- D. To ensure the endpoint can communicate with the Internet Gateway.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** The security group on the endpoint's ENI acts as an inbound firewall for the endpoint itself. It controls which resources within the VPC can initiate a connection to the endpoint. For example, you could configure it to only accept traffic from your "App-Server-SG."
</details>
---

**21. A Gateway Endpoint is created in a VPC. Can it be used by instances in a peered VPC?**

- A. Yes, if the route tables are updated correctly.
- B. No, a Gateway Endpoint cannot be extended to a peered VPC.
- C. Yes, but only for S3.
- D. Yes, but only if the peered VPC is in the same AWS account.
<details>
<summary>View Answer</summary>

**Answer:** B

**Explanation:** This is a known limitation. A Gateway Endpoint is local to the VPC in which it is created. Its route cannot be used as a target in a route table of a peered VPC. To provide private access to S3 or DynamoDB for a peered VPC, the peered VPC would need to create its own endpoint.
</details>
---

**22. You need private connectivity to the EC2 API (e.g., to run `DescribeInstances`) from a Lambda function in a private subnet. What should you configure?**

- A. A Gateway Endpoint for the EC2 API.
- B. A NAT Gateway for the Lambda function.
- C. An Interface Endpoint for the EC2 API.
- D. This is not possible; Lambda functions cannot use VPC endpoints.
<details>
<summary>View Answer</summary>

**Answer:** C

**Explanation:** The EC2 API is a service supported by Interface Endpoints. By creating an Interface Endpoint for com.amazonaws.<region>.ec2 and placing your Lambda function in the same VPC, the function can make API calls to the EC2 service without needing internet access via a NAT Gateway.
</details>
