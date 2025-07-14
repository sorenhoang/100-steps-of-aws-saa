# Step 20: AWS WAF, AWS Shield, AWS Firewall Manager

## 🎯 Objective
- [x] Master: **AWS WAF, AWS Shield, AWS Firewall Manager**

## 📘 Notes

### **Deep Dive: WAF, Shield, & Firewall Manager**

As applications become more exposed to the internet, they become targets for various attacks. AWS provides a suite of managed security services that work together to create a layered defense. AWS Shield protects against DDoS attacks, AWS WAF protects against application-layer attacks, and Firewall Manager helps you manage these protections centrally.

---

### **1. AWS WAF (Web Application Firewall)** 🛡️

AWS WAF is a web application firewall that helps protect your web applications from common web exploits that could affect application availability, compromise security, or consume excessive resources. It operates at **Layer 7 (the Application Layer)**.

- **How it works:** You create a set of rules, called a **Web Access Control List (Web ACL)**, that define filtering logic. WAF inspects incoming HTTP/HTTPS requests and allows or blocks them based on these rules.
- **What it protects against:**
    - **SQL Injection:** Attackers embedding malicious SQL code in web requests to manipulate your database.
    - **Cross-Site Scripting (XSS):** Attackers injecting malicious scripts into content that is then delivered to other users' browsers.
    - It also protects against other common attacks like command injection, file inclusion, and more.
- **Types of Rules:**
    - **IP-based rules:** Block traffic from specific IP addresses or ranges.
    - **Rate-based rules:** Block IPs that send an excessive number of requests in a short period.
    - **Content-based rules:** Inspect parts of the request like the URI, query string, HTTP headers, and body for malicious patterns (e.g., block if the body contains `<script>`).
    - **AWS Managed Rule Groups:** Pre-configured sets of rules managed by AWS or AWS Marketplace sellers that protect against common threats (like the OWASP Top 10). This is the easiest way to get started.
- **Integration:** WAF is not a standalone service. It must be attached to a supported AWS resource. These include:
    - **Amazon CloudFront** distributions
    - **Application Load Balancers (ALB)**
    - **Amazon API Gateway** REST APIs
    - **AWS AppSync** GraphQL APIs

---

### **2. AWS Shield** 🌊

AWS Shield is a managed **Distributed Denial of Service (DDoS)** protection service that safeguards applications running on AWS. DDoS attacks attempt to make an online service unavailable by overwhelming it with traffic from multiple sources.

There are two tiers of AWS Shield:

- **AWS Shield Standard:**
    - **Protection:** Defends against most common, network and transport layer (Layer 3 & 4) DDoS attacks, such as SYN floods or UDP reflection attacks.
    - **Cost:** **Free**. It is automatically enabled for all AWS customers on services like CloudFront, Route 53, and Elastic Load Balancers.
    - **Scope:** Provides baseline, always-on network flow monitoring and protection.
- **AWS Shield Advanced:**
    - **Protection:** Provides significantly more sophisticated and powerful protection for internet-facing applications on EC2, ELB, CloudFront, Route 53, and Global Accelerator. It offers enhanced detection, protection against large and sophisticated Layer 7 (application layer) DDoS attacks, and near real-time visibility into attacks.
    - **Key Features:**
        1. **24/7 DDoS Response Team (DRT):** You get access to AWS security experts who can help you during a DDoS attack.
        2. **DDoS Cost Protection:** Provides financial protection against scaling charges incurred as a result of a DDoS attack. For example, if your Auto Scaling group scales out to handle attack traffic, AWS may provide service credits for those charges.
        3. **Advanced Reporting:** Gives you detailed diagnostics and reports on attacks targeting your resources.
        4. **Bundled AWS WAF:** Using AWS WAF is included at no extra cost for your protected resources.
    - **Cost:** Paid service with a significant monthly fee plus data transfer fees. It is designed for businesses with critical, availability-sensitive applications.

---

### **3. AWS Firewall Manager** 🧑‍✈️

AWS Firewall Manager is a security management service that allows you to centrally configure and manage firewall rules across multiple accounts and resources in your **AWS Organization**.

- **Purpose:** As you scale to dozens or hundreds of accounts, manually configuring WAF rules or security groups in each account becomes unmanageable. Firewall Manager solves this by providing a single place to deploy and monitor your security rules.
- **What it Manages:**
    - **AWS WAF rules:** Centrally deploy a common set of WAF rules to all your ALBs, API Gateways, and CloudFront distributions across the organization.
    - **AWS Shield Advanced:** Centrally enable Shield Advanced protection for all relevant resources.
    - **VPC Security Groups:** Audit and control security groups (e.g., enforce a rule that blocks inbound SSH from the internet on all instances).
    - **AWS Network Firewall:** Centrally deploy and manage stateful network firewall rules.
    - **Route 53 Resolver DNS Firewall:** Centrally deploy DNS filtering rules.
- **How it Works:** You define a **policy**, select which accounts or OUs it applies to, and Firewall Manager automatically enforces that policy, ensuring that both existing and newly created resources are compliant.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company's web application is experiencing a common web attack where malicious SQL code is being inserted into user input fields to exploit the backend database. Which AWS service is specifically designed to prevent this type of attack?**

- A. AWS Shield Advanced
- B. AWS WAF
- C. Network ACLs
- D. Amazon GuardDuty

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This describes a SQL Injection attack, which is a Layer 7 (application-layer) exploit. AWS WAF is the service designed to inspect HTTP/HTTPS requests and block common exploits like SQL Injection and Cross-Site Scripting (XSS). Shield (A) is for DDoS, and NACLs (C) operate at the network layer, not the application layer.
</details>
    

---

**2. All AWS customers receive automatic protection against common, network-layer DDoS attacks at no additional cost. Which service provides this baseline protection?**

- A. AWS WAF
- B. AWS Shield Standard
- C. AWS Shield Advanced
- D. AWS Firewall Manager

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** AWS Shield Standard is enabled by default for all customers and provides protection against most common network and transport layer (Layer 3/4) DDoS attacks. This service is free and always-on for services like ELB, CloudFront, and Route 53.
</details>
    

---

**3. A large enterprise with hundreds of AWS accounts wants to ensure that a mandatory set of AWS WAF rules is applied to every new Application Load Balancer created in any account within their AWS Organization. What is the most efficient way to manage and enforce this policy?**

- A. Manually create the WAF rules in each account.
- B. Use AWS Shield Advanced to automatically deploy the rules.
- C. Use AWS Firewall Manager to create a policy that deploys the WAF rules across the organization.
- D. Create a Lambda function that triggers on new ALBs and applies the WAF rules.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is the primary use case for AWS Firewall Manager. It is designed for centralized management of security rules across an AWS Organization. You can create a Firewall Manager policy to automatically associate a common Web ACL with all ALBs (or other supported resources) in specified accounts or OUs, ensuring consistent protection.
</details>
    

---

**4. A company subscribes to AWS Shield Advanced. During a large DDoS attack, their Auto Scaling group scales out significantly, leading to a large EC2 bill. What benefit does Shield Advanced offer in this situation?**

- A. It automatically scales the instances back in.
- B. It provides DDoS Cost Protection, offering service credits for charges incurred due to the attack.
- C. It blocks the attack traffic at the Internet Gateway.
- D. It provides no financial protection; it only mitigates the attack.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A key feature of AWS Shield Advanced is DDoS Cost Protection. It provides a safeguard against bill shock caused by scaling up resources to absorb a DDoS attack. Customers can request service credits for these attack-related charges.
</details>
    

---

**5. AWS WAF can be integrated with which of the following AWS services? (Select TWO)**

- A. Network Load Balancer
- B. Application Load Balancer
- C. EC2 Instances
- D. Amazon CloudFront
- E. NAT Gateway

<details>
<summary>View Answer</summary>
<br>

**Answers: B and D**

**Explanation:** AWS WAF is a Layer 7 firewall and can only be attached to AWS resources that operate at Layer 7. The primary integration points are Application Load Balancers (ALB) and Amazon CloudFront distributions. It also integrates with API Gateway and AppSync. It cannot be attached directly to an EC2 instance or a Layer 4 service like an NLB.
</details>
    

---

**6. What is the primary difference between AWS Shield Standard and AWS Shield Advanced?**

- A. Shield Standard protects against Layer 7 attacks, while Shield Advanced protects against Layer 3/4 attacks.
- B. Shield Standard is a paid service, while Shield Advanced is free.
- C. Shield Advanced provides access to the 24/7 AWS DDoS Response Team (DRT) and cost protection, while Shield Standard does not.
- D. Shield Standard is managed by Firewall Manager, while Shield Advanced is not.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The key value propositions of Shield Advanced are the additional services and protections it offers beyond the baseline. This includes access to human experts on the DDoS Response Team (DRT), financial protection against DDoS-related charges, and more sophisticated detection of large-scale attacks.
</details>
    

---

**7. A WAF rule is configured to block requests if the request originates from a specific country. What type of rule is this?**

- A. Rate-based rule
- B. SQL injection match rule
- C. Geographic match rule
- D. Size constraint rule

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS WAF supports geographic match conditions, allowing you to create rules that allow or block requests based on the country of origin, as determined by the source IP address.
</details>
    

---

**8. Which AWS service is required as a prerequisite for using AWS Firewall Manager?**

- A. AWS Control Tower
- B. AWS Organizations
- C. AWS Trusted Advisor
- D. AWS Systems Manager

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** AWS Firewall Manager is designed for multi-account management. Therefore, you must have AWS Organizations set up first. Firewall Manager uses the organizational structure (OUs and accounts) to apply its security policies.
</details>
    

---

**9. What does a "rate-based rule" in AWS WAF do?**

- A. It blocks requests based on their country of origin.
- B. It blocks requests that contain malicious SQL code.
- C. It tracks the rate of requests from individual IP addresses and blocks an IP if its request rate exceeds a defined threshold.
- D. It charges users based on the number of requests they make.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A rate-based rule is a powerful tool to mitigate web-layer DDoS attacks or brute-force login attempts. You set a threshold (e.g., 2000 requests per 5 minutes), and WAF will automatically block any source IP that exceeds this rate.
</details>
    

---

**10. A company wants to use a pre-configured set of WAF rules that protect against the most common vulnerabilities, such as those listed in the OWASP Top 10. What is the easiest way to implement this?**

- A. Write custom WAF rules for each of the OWASP Top 10 vulnerabilities.
- B. Use an AWS Managed Rule Group for WAF.
- C. Subscribe to AWS Shield Advanced.
- D. Create a Network ACL with rules for each vulnerability.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** AWS provides Managed Rule Groups which are curated sets of rules designed to provide protection against common threats without requiring you to write the rules yourself. There are specific managed rule groups for threats like the OWASP Top 10, known bad IPs, Amazon IP reputation lists, and more.
</details>
    

---

**11. Which service provides protection against volumetric UDP reflection attacks?**

- A. AWS WAF
- B. AWS Shield
- C. AWS Firewall Manager
- D. Amazon Inspector

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** UDP reflection attacks are a common type of Layer 3/4 DDoS attack. AWS Shield (both Standard and Advanced) is the service designed to detect and mitigate these network-layer volumetric attacks. WAF operates at Layer 7 and would not be effective against this type of attack.
</details>
    

---

**12. A company has deployed AWS WAF on its Application Load Balancer. They now want to deploy the exact same WAF rules on their Amazon CloudFront distribution. What is the most efficient method?**

- A. Manually re-create all the rules in a new Web ACL for CloudFront.
- B. Create one Web ACL and associate it with both the Application Load Balancer and the CloudFront distribution.
- C. Use AWS Firewall Manager to apply the same policy to both resources.
- D. This is not possible; a Web ACL can only be associated with one resource type.

<details>
<summary>View Answer</summary>
<br>

**Answer: B or C**

**Explanation:** Both B and C are valid approaches.
    
- A Web ACL can be associated with multiple resources of the same type (e.g., multiple ALBs) or different types (an ALB and a CloudFront distribution). So, creating one Web ACL and associating it with both is an efficient solution (B).
- If managing this at scale across many accounts, **AWS Firewall Manager (C)** would be the superior solution for applying the policy centrally. In a single-account context, B is the most direct answer.
</details>

---

**13. A company is using AWS Shield Advanced. What additional benefit do they receive regarding AWS WAF?**

- A. They get a 50% discount on WAF usage.
- B. They can use AWS WAF on Network Load Balancers.
- C. The cost of using AWS WAF on their protected resources is included with their Shield Advanced subscription.
- D. They get access to a special version of WAF called WAF Advanced.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A key benefit of subscribing to AWS Shield Advanced is that the usage costs for AWS WAF on any resource protected by Shield Advanced are waived. This encourages customers to use both services together for layered protection.
</details>
    

---

**14. What is the primary role of the DDoS Response Team (DRT) available to AWS Shield Advanced customers?**

- A. To write custom WAF rules for your application.
- B. To provide 24/7 expert assistance during a DDoS attack to help with mitigation.
- C. To manage your AWS Firewall Manager policies.
- D. To perform penetration testing on your application.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The DDoS Response Team (DRT) is a group of AWS experts who can be engaged during a critical DDoS event. They help analyze the attack and can apply additional, non-public mitigations to help keep your application online. This human-in-the-loop support is a major feature of the Shield Advanced service.
</details>
    

---

**15. What is a Web ACL in the context of AWS WAF?**

- A. A type of network firewall rule.
- B. A collection of rules that define the criteria for inspecting and filtering web traffic.
- C. A log of all allowed and blocked web requests.
- D. The IAM policy required to use AWS WAF.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Web Access Control List (Web ACL) is the core resource you create in AWS WAF. It is the container for all the rules (IP match, geographic match, SQL injection match, etc.) that you want to apply to your web traffic. You then associate this Web ACL with a resource like an ALB or CloudFront distribution.
</details>
    

---

**16. How does AWS Firewall Manager help with compliance?**

- A. By encrypting all data at rest.
- B. By providing a single point to configure, enforce, and audit security rules across an entire AWS Organization.
- C. By performing vulnerability scans on EC2 instances.
- D. By providing free access to third-party auditors.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Firewall Manager provides a centralized way to enforce a security baseline. This helps with compliance by ensuring that all accounts and resources adhere to mandatory rules (e.g., "all public web applications must have WAF protection"). It also provides a central view to audit this compliance, simplifying the process of proving to auditors that security controls are in place.
</details>
    

---

**17. An application is being targeted by a DDoS attack that attempts to exhaust server resources with a flood of legitimate-looking HTTP GET requests. This is an example of what kind of attack?**

- A. A Layer 3 (Network Layer) attack.
- B. A Layer 4 (Transport Layer) attack.
- C. A Layer 7 (Application Layer) attack.
- D. A DNS amplification attack.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a classic Layer 7 DDoS attack. The requests themselves are well-formed (unlike a SYN flood), but they come in such high volume that they overwhelm the application server's ability to process them. Shield Advanced and AWS WAF (especially with rate-based rules) are designed to mitigate these attacks.
</details>
    

---

**18. Which service would you use to centrally manage and deploy VPC security group rules across all accounts in your organization?**

- A. AWS Systems Manager
- B. AWS WAF
- C. AWS Config
- D. AWS Firewall Manager

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** In addition to managing WAF and Shield, AWS Firewall Manager can also be used to manage and audit VPC security groups. For example, you can create a policy to audit for any security groups that allow unrestricted SSH access and have it automatically remediate the issue.
</details>
    

---

**19. A company wants to protect its Application Load Balancer from a Cross-Site Scripting (XSS) attack. What is the most direct solution?**

- A. Enable AWS Shield Standard.
- B. Associate an AWS WAF Web ACL with a rule to block XSS patterns.
- C. Create a Network ACL rule to block port 443.
- D. Place the ALB behind a NAT Gateway.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** XSS is a Layer 7 attack. The most direct and appropriate way to block it is by using AWS WAF. You can create a Web ACL with either a custom rule or, more easily, use the AWS Managed Rule Group for Core Rule Set (CRS), which includes protection against XSS patterns.
</details>
    

---

**20. What is the scope of AWS Shield Standard's protection?**

- A. It only protects resources within a single Availability Zone.
- B. It protects against all types of Layer 7 attacks.
- C. It provides automatic protection for supported AWS resources against common network and transport layer DDoS attacks.
- D. It only protects EC2 instances with Elastic IP addresses.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Shield Standard provides a baseline level of protection against the most common, volumetric DDoS attacks at Layer 3 and Layer 4. This protection is automatically and transparently applied to edge services like CloudFront, Route 53, and ELB.
</details>
    

---

**21. A Firewall Manager policy is created to enforce a specific WAF Web ACL. What happens when a developer creates a new Application Load Balancer in an account that is within the scope of this policy?**

- A. The developer must manually attach the Web ACL.
- B. The launch of the ALB will fail until the policy is removed.
- C. Firewall Manager will automatically detect the new ALB and associate the specified Web ACL with it.
- D. The developer will receive an email notification reminding them to attach the Web ACL.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** A key benefit of Firewall Manager is its ability to enforce compliance on both existing and new resources. It continuously monitors the accounts within its scope and will automatically apply the configured policy (e.g., attach the required WAF Web ACL) to any new resource that matches the policy's criteria.
</details>
    

---

**22. To get access to the AWS DDoS Response Team (DRT), what must a customer do?**

- A. Nothing, the DRT is available to all AWS customers.
- B. Open a "critical" support case with AWS Support.
- C. Subscribe to the AWS Business Support plan.
- D. Subscribe to AWS Shield Advanced.

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** Access to the 24/7 DDoS Response Team (DRT) is an exclusive feature of the AWS Shield Advanced service. It is not included with Shield Standard or any of the standard support plans.
</details>