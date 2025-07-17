# Step 63: AWS Config, Inspector, GuardDuty

## 🎯 Objective

- [x] Master: **AWS Config, Inspector, GuardDuty**

## 📘 Notes

### **Deep Dive: Config, Inspector, & GuardDuty**

Think of securing your AWS account like securing a house. AWS Config is the inspector who checks if all your windows and doors are built to code. Amazon Inspector checks if the locks on those doors are easy to pick. Amazon GuardDuty is the intelligent security system that watches for burglars trying to get in.

---

### **1. AWS Config** 📋

**What it is:** A service that continuously monitors and records your AWS resource **configurations** and allows you to automate the evaluation of recorded configurations against desired policies.

- **Core Question:** *"Is my resource configured correctly according to my rules?"*
- **How it works:**
    1. **Recording:** Config discovers and records the current configuration of your supported AWS resources (e.g., an EC2 instance, a security group, an S3 bucket). It also records every change to that configuration over time.
    2. **Evaluation:** You use **Config Rules** to define your desired configuration state. A rule can be a managed rule provided by AWS (e.g., "ebs-encryption-enabled") or a custom rule you create with a Lambda function.
    3. **Compliance:** Config continuously evaluates your resources against your rules and flags them as **COMPLIANT** or **NON_COMPLIANT**.
- **Key Features:**
    - **Configuration History:** Provides a detailed history of all configuration changes for a resource, which is invaluable for troubleshooting and auditing.
    - **Remediation:** Can be configured to automatically remediate non-compliant resources using AWS Systems Manager Automation documents.
- **Use Case:**
    - **Compliance Auditing:** Answering questions like, "Are all of my EBS volumes encrypted?" or "Do any of my security groups allow unrestricted SSH access?"
    - **Change Management:** Tracking all configuration changes in your environment.
    - **Troubleshooting:** Identifying which configuration change caused a recent operational issue.

---

### **2. Amazon Inspector** 🕵️

**What it is:** An automated **vulnerability management service** that continuously scans your AWS workloads for software vulnerabilities and unintended network exposure.

- **Core Question:** *"Does my EC2 instance or container image have known software vulnerabilities (CVEs) or is it exposed to the network?"*
- **How it works:**
    - Once enabled, Inspector automatically discovers all your running EC2 instances and container images residing in ECR.
    - It continuously scans these resources without needing to install an agent (for EC2, it uses the AWS Systems Manager Agent).
    - It compares the installed software packages against a database of Common Vulnerabilities and Exposures (CVEs).
    - It also analyzes network accessibility to identify any unintended open paths to your instances.
- **Key Features:**
    - **Continuous Scanning:** It's not a one-time scan; it continuously monitors your resources as they are deployed and changed.
    - **Risk Score:** It provides a highly contextualized risk score for each finding, helping you prioritize the most critical vulnerabilities.
- **Use Case:**
    - **Vulnerability Management:** Finding unpatched software with known CVEs (e.g., an outdated version of Nginx or Log4j).
    - **Network Exposure Analysis:** Identifying if an instance with a critical vulnerability is also reachable from the internet.

---

### **3. Amazon GuardDuty** 👮

**What it is:** An intelligent **threat detection service** that continuously monitors your AWS accounts and workloads for malicious activity and unauthorized behavior.

- **Core Question:** *"Is someone trying to compromise my AWS account or resources right now?"*
- **How it works:** GuardDuty analyzes multiple AWS data sources without you needing to enable them. These include:
    - **VPC Flow Logs**
    - **DNS Logs**
    - **AWS CloudTrail Events** (Management and Data Events)
- **What it Detects:** It uses machine learning, anomaly detection, and integrated threat intelligence to identify potential threats, such as:
    - An EC2 instance communicating with a known crypto-mining or command-and-control server.
    - Unusual API calls from a strange geographic location.
    - Port scanning or attempts to probe your instances.
    - Potential account compromise, like API calls being made from a TOR node.
- **Key Features:**
    - **Intelligent and Managed:** You enable it with one click, and it starts analyzing data sources without any agents or complex setup.
    - **Actionable Findings:** It generates security findings with severity levels (High, Medium, Low) that are easy to understand. Findings can be sent to EventBridge to trigger automated responses.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A security team needs to ensure that all EBS volumes attached to EC2 instances are encrypted. If an unencrypted volume is found, they want to be notified. Which AWS service is designed to continuously check for this type of configuration compliance?**

- A. Amazon Inspector
- B. AWS Config
- C. Amazon GuardDuty
- D. AWS Shield
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is a question about the configuration of a resource. AWS Config is the service designed to audit resource configurations. You would use the managed Config Rule encrypted-volumes to check if EBS volumes are encrypted and flag any non-compliant resources.
</details>
    

---

**2. An administrator enables a service that begins analyzing VPC Flow Logs and DNS logs to detect anomalies, such as an EC2 instance communicating with a known malicious IP address. Which service was enabled?**

- A. Amazon Inspector
- B. AWS Config
- C. Amazon GuardDuty
- D. AWS CloudTrail
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Amazon GuardDuty is the intelligent threat detection service that automatically analyzes data sources like VPC Flow Logs and DNS logs. Detecting communication with known malicious IPs is a primary feature of GuardDuty's threat intelligence feeds.
</details>
    

---

**3. A company needs to perform regular security assessments on their running EC2 instances to identify any software vulnerabilities, such as an unpatched version of the Apache web server. Which service should be used?**

- A. Amazon GuardDuty
- B. AWS Systems Manager Patch Manager
- C. Amazon Inspector
- D. AWS Config
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The key requirement is to find software vulnerabilities (like outdated packages with known CVEs) on the EC2 instance itself. Amazon Inspector is the vulnerability management service designed for this exact purpose. Patch Manager (B) is used to apply patches, but Inspector is used to find them.
</details>
    

---

**4. Which service answers the question, "Has the configuration of my S3 bucket changed recently, and what did it change from?"**

- A. AWS CloudTrail
- B. AWS Config
- C. Amazon Inspector
- D. Amazon GuardDuty
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** While CloudTrail (A) would show the API call that made the change, AWS Config is the service that specifically records the state of a resource's configuration over time. It provides a detailed configuration history, allowing you to easily see a timeline of changes and diffs between different points in time.
</details>
    

---

**5. Which service provides protection by analyzing data sources like CloudTrail logs but does NOT require the user to enable those logs first?**

- A. AWS Config
- B. Amazon Inspector
- C. AWS Trusted Advisor
- D. Amazon GuardDuty
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** A key benefit of Amazon GuardDuty is that it has direct, independent access to data streams like CloudTrail events, VPC Flow Logs, and DNS logs. You can enable GuardDuty with one click, and it will start analyzing this data without you having to manually enable and manage those individual log sources yourself.
</details>
    

---

**6. What is a "Config Rule"?**

- A. A rule that detects malicious network traffic.
- B. A rule that defines a desired configuration state for an AWS resource and is used to check for compliance.
- C. A rule that scans for software vulnerabilities.
- D. A rule that automatically patches an operating system.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Config Rule is the core component of AWS Config's evaluation engine. It represents your definition of "good." You create rules to check for specific settings (e.g., "S3 buckets should not have public read access"), and Config evaluates your resources against them.
</details>
    

---

**7. Amazon Inspector has found a critical vulnerability on an EC2 instance. What other information does Inspector provide to help prioritize the finding?**

- A. A history of all API calls made by the instance.
- B. An analysis of the instance's network reachability from the internet.
- C. A list of all IAM users who have access to the instance.
- D. The cost savings associated with patching the vulnerability.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon Inspector provides a contextual risk score. A key part of this is analyzing network reachability. A critical vulnerability on an instance that is completely isolated in a private network is less urgent than the same vulnerability on an instance that is exposed to the internet.
</details>
    

---

**8. Which of the following findings would be generated by Amazon GuardDuty?**

- A. An S3 bucket has a policy that allows public access.
- B. An EC2 instance has an outdated version of OpenSSL with a known CVE.
- C. An IAM user's access keys are being used to make API calls from a TOR exit node.
- D. The number of EC2 instances in an account has exceeded a service limit.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Detecting anomalous behavior, such as API calls from an unusual location or a known malicious source like a TOR node, is a hallmark of GuardDuty. The other findings would come from AWS Config or Trusted Advisor (A, D) and Amazon Inspector (B).
</details>
    

---

**9. What is the primary purpose of AWS Config?**

- A. Real-time threat detection.
- B. Vulnerability scanning.
- C. Configuration auditing and compliance checking.
- D. Cost optimization.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The core function of AWS Config is to continuously monitor and record your resource configurations. This makes it the primary tool for auditing your environment against internal policies or external compliance frameworks.
</details>
    

---

**10. How does Amazon Inspector scan EC2 instances for vulnerabilities without requiring a dedicated agent to be installed by the user?**

- A. It uses VPC Flow Logs to analyze network traffic.
- B. It uses the AWS Systems Manager (SSM) Agent, which is installed by default on many AWS-provided AMIs.
- C. It takes a snapshot of the EBS volume and analyzes it offline.
- D. It analyzes the CloudTrail logs for the instance.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The modern version of Amazon Inspector leverages the AWS Systems Manager (SSM) Agent. Since this agent is already present on most Amazon Linux and Windows AMIs, Inspector can use it to collect the necessary information about installed software packages without requiring the user to deploy a separate, dedicated Inspector agent.
</details>
    

---

**11. Which service would you use to answer the question, "Show me a history of all changes made to my VPC's main route table over the last month"?**

- A. Amazon Inspector
- B. AWS Config
- C. Amazon GuardDuty
- D. VPC Flow Logs
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is a question about configuration history. AWS Config records every change to a resource's configuration and provides a timeline of those changes. You can use it to see exactly when a route was added or removed and what its previous state was.
</details>
    

---

**12. To detect if an EC2 instance is being used to perform a port scan against other instances in your VPC, which service should you use?**

- A. AWS Config
- B. Amazon CloudWatch
- C. AWS Trusted Advisor
- D. Amazon GuardDuty
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** Port scanning is a type of reconnaissance activity that indicates potentially malicious behavior. Amazon GuardDuty is designed to detect such threats by analyzing VPC Flow Logs and identifying patterns that are indicative of a port scan.
</details>
    

---

**13. A company needs to automatically fix any non-compliant resources. For example, if a security group is created that allows all traffic (`0.0.0.0/0`), it should be automatically corrected. What feature enables this?**

- A. GuardDuty automated remediation.
- B. Inspector automated patching.
- C. AWS Config Rules with remediation actions.
- D. CloudTrail automated rollbacks.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS Config has a remediation feature. You can associate a Config Rule with a remediation action, which is typically an AWS Systems Manager (SSM) Automation document. When the rule detects a non-compliant resource, it can automatically trigger the SSM document to fix the configuration.
</details>
    

---

**14. Which service is designed to be enabled with a single click and immediately begins providing value without requiring complex configuration?**

- A. AWS Config
- B. Amazon Inspector
- C. Amazon GuardDuty
- D. All of the above.
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** While all three services are relatively easy to enable, Amazon GuardDuty is the one that provides the most value with the least configuration. You enable it, and it immediately starts analyzing multiple data sources in the background to detect threats. Config (A) requires you to set up rules, and Inspector (B) requires the SSM agent to be running.
</details>
    

---

**15. If you wanted to find out if any of your running EC2 instances have network ports open to the entire internet, which service would you use?**

- A. AWS CloudTrail
- B. Amazon GuardDuty
- C. Amazon Inspector
- D. AWS IAM Access Analyzer
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Amazon Inspector performs two types of scans: software vulnerability scans and network reachability analysis. The network reachability scan will identify any instances that have open ports accessible from the internet and flag them as unintended network exposure.
</details>
    

---

**16. What are the primary data sources for Amazon GuardDuty? (Select TWO)**

- A. Amazon S3 Access Logs
- B. VPC Flow Logs
- C. AWS CloudTrail Logs
- D. Application logs from EC2 instances
- E. Amazon Inspector findings
<details>
<summary>View Answer</summary>
<br>

**Answer: B and C**

**Explanation:** GuardDuty's intelligence is fueled by analyzing several high-volume data sources. The primary ones you need to know are VPC Flow Logs, AWS CloudTrail events, and DNS logs. It analyzes these sources automatically without you needing to manage them.
</details>
    

---

**17. Which service helps you answer the question, "Is my account being compromised by crypto-mining activity?"**

- A. AWS Config
- B. Amazon Inspector
- C. AWS Cost Explorer
- D. Amazon GuardDuty
<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** Amazon GuardDuty has specific threat detection models for identifying patterns associated with cryptocurrency mining. It can detect if an EC2 instance is communicating with known crypto-mining domains or IP addresses.
</details>
    

---

**18. A company wants to ensure all newly created S3 buckets have server-side encryption enabled. Which service can enforce and report on this policy?**

- A. Amazon GuardDuty
- B. Amazon Macie
- C. AWS Config
- D. Amazon Inspector
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a resource configuration policy. AWS Config is the service to use. You would enable the s3-bucket-server-side-encryption-enabled managed rule to continuously check all S3 buckets for this specific configuration setting.
</details>
    

---

**19. What is the key difference between Amazon Inspector and Amazon GuardDuty?**

- A. Inspector is for auditing, while GuardDuty is for vulnerability scanning.
- B. Inspector finds vulnerabilities *within* your workloads (e.g., unpatched software), while GuardDuty detects active threats *against* your account and workloads (e.g., malware communication).
- C. Inspector is free, while GuardDuty is a paid service.
- D. Inspector analyzes CloudTrail logs, while GuardDuty analyzes VPC Flow Logs.
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the core distinction. Inspector is proactive vulnerability management (looking for potential weaknesses). GuardDuty is reactive threat detection (looking for active, malicious behavior).
</details>
    

---

**20. A compliance officer needs a report of all IAM policies that were changed in the last 90 days. Which service provides this information?**

- A. IAM Access Analyzer
- B. AWS CloudTrail
- C. AWS Config
- D. Amazon Inspector
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Changing an IAM policy is an API call (PutUserPolicy, CreatePolicyVersion, etc.). AWS CloudTrail records all these API calls. You can search the CloudTrail event history to find all IAM-related API calls and see who made them and when. While Config (C) also tracks this, CloudTrail is the direct log of the API action.
</details>
    

---

**21. A security finding from which service would indicate that an application is vulnerable to a known CVE?**

- A. Amazon GuardDuty
- B. AWS WAF
- C. Amazon Inspector
- D. AWS Shield
<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Amazon Inspector's primary function is to scan the software packages on your resources and compare them against a database of known Common Vulnerabilities and Exposures (CVEs).
</details>
    

---

**22. Which service's findings would prompt you to patch an EC2 instance?**

- A. AWS Config
- B. Amazon Inspector
- C. Amazon GuardDuty
- D. VPC Flow Logs
<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Amazon Inspector identifies software vulnerabilities. The remediation for a software vulnerability is almost always to apply a patch to update the software to a secure version. Therefore, an Inspector finding directly leads to a patching action.
</details>