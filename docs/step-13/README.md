# Step 13: EC2 User Data & Bootstrapping

## 🎯 Objective

- [x] Master: **EC2 User Data & Bootstrapping**

## 📘 Notes

### **Deep Dive: EC2 User Data & Bootstrapping**

When you launch a new EC2 instance from an AMI, it starts with a clean, standardized software configuration. However, you almost always need to perform initial setup tasks, like installing or updating software, downloading application code, or configuring system settings. Performing these tasks manually is slow, error-prone, and not scalable. **Bootstrapping** is the process of automating these initial configurations. The primary mechanism AWS provides for bootstrapping is **EC2 User Data**.

### **What is EC2 User Data?** 📜✍️

EC2 User Data is a script that you can provide to an EC2 instance at launch time. This script is automatically executed by the instance the **very first time it boots up**. It is the most common way to customize an instance as it launches.

- **How it works:** You pass the script to the instance as part of the launch configuration. On Linux instances, this script is executed by the **`cloud-init`** package, which runs during the initial boot sequence. On Windows instances, it's handled by **EC2Launch** or **EC2Config**.
- **Execution Frequency:** This is a critical point to remember. The User Data script runs **only once** when the instance is first launched. It does not run again if you stop and start the instance or if you reboot it.
- **Permissions:** The script runs with **root** (on Linux) or **Administrator** (on Windows) privileges, so it can perform system-level tasks like installing packages, modifying configuration files, and starting services.
- **Size Limit:** The User Data script is limited to **16 KB** in size. For more complex configurations, the best practice is to have the User Data script download a larger, more detailed configuration script from a source like S3 or a Git repository.
- **Viewing Output:** You can view the output of the `cloud-init` process and your User Data script by checking the log files on the instance. On Amazon Linux, the main log file is at `/var/log/cloud-init-output.log`.

---

### **Bootstrapping: What Can You Do with User Data?**

User Data scripts are incredibly versatile. Common bootstrapping tasks include:

- **Installing Updates:** Running `yum update -y` or `apt-get update -y` to apply the latest security patches.
- **Installing Software:** Installing necessary packages like a web server (`httpd`, `nginx`), a database, or monitoring agents.
- **Downloading Application Code:** Using `git clone` to pull the latest version of your application from a code repository.
- **Starting Services:** Ensuring that services like `httpd` or your application service are enabled and started on boot.
- **Configuring the Instance:** Writing configuration files, setting environment variables, or joining the instance to a domain.

**Example User Data Script (Shell Script):**

This is a typical script for bootstrapping a simple web server on Amazon Linux.1

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A solutions architect needs to ensure that all newly launched EC2 instances automatically install the latest security patches and a specific monitoring agent upon their first boot. What is the most efficient way to achieve this?**

- A. Manually SSH into each instance after it launches and run the necessary commands.
- B. Create a custom AMI that already includes the patches and agent.
- C. Pass a shell script that performs the updates and installation into the EC2 User Data field at launch.
- D. Take a snapshot of a running instance and launch new instances from the snapshot.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** EC2 User Data is designed for exactly this purpose: running initial configuration scripts at launch. This is more flexible than creating a custom AMI (B), as it ensures the absolute latest patches are installed at the moment of launch, rather than the latest patches from when the AMI was created. Manual configuration (A) is not scalable.

</details>

---

**2. A web server is launched from a standard Amazon Linux 2 AMI with a User Data script. After launching, a developer stops the instance for the night and starts it again the next morning. Will the User Data script run again when the instance is started?**

- A. Yes, the User Data script runs every time the instance boots.
- B. No, the User Data script only runs on the very first boot of the instance.
- C. Yes, but only if the instance type is changed.
- D. It depends on the content of the script.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a critical concept for the exam. By default, the cloud-init service executes the User Data script only once during the initial launch sequence. Subsequent stops/starts or reboots will not re-trigger the script.

</details>

---

**3. A bootstrapping script needs to retrieve the instance ID of the EC2 instance it is running on to tag the instance appropriately. How can the script obtain this information?**

- A. By querying the AWS IAM service.
- B. By making a call to the EC2 Instance Metadata Service endpoint at `169.254.169.254`.
- C. The instance ID must be manually passed into the User Data script as a variable.
- D. By querying the AWS Organizations service.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The EC2 Instance Metadata Service is a special service available from within any EC2 instance at the link-local address 169.254.169.254. The script can make a simple HTTP GET request (e.g., using curl) to http://169.254.169.254/latest/meta-data/instance-id to retrieve its own instance ID without needing any special credentials.

</details>

---

**4. A developer has written a complex bootstrapping script that is 25 KB in size. When they try to add it to the User Data field in the EC2 launch wizard, they receive an error. What is the most likely cause?**

- A. The script contains a syntax error.
- B. The User Data field has a size limit of 16 KB.
- C. The IAM role attached to the instance does not have permission to run scripts.
- D. The script must be base64 encoded first.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The EC2 User Data field has a hard limit of 16 KB. For scripts larger than this, the recommended best practice is to place the full script in a location like an S3 bucket and use a small User Data script to download and execute it.

</details>

---

**5. What permissions does a User Data script run with on a standard Amazon Linux 2 instance?**

- A. As the `ec2-user`.
- B. As a special `cloud-init` user with limited privileges.
- C. As the `root` user.
- D. The permissions are inherited from the IAM role attached to the instance.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** By default, User Data scripts are executed with root privileges. This is necessary so they can perform system-level tasks like installing software (yum install), modifying system configuration files, and starting services.

</details>

---

**6. A solutions architect needs to automate the deployment of a fleet of web servers. The configuration is complex and involves pulling application code from a private S3 bucket. What is the recommended approach for the bootstrapping process?**

- A. Paste the entire multi-thousand-line script directly into the User Data field.
- B. Host the configuration script in an S3 bucket, and use a small User Data script to download and execute the main script.
- C. Create a custom AMI and manually update it every time the application code changes.
- D. Embed the S3 access keys directly in the User Data script to access the private bucket.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the standard best practice for complex configurations. It keeps the User Data field clean and avoids the 16 KB limit. The instance should have an IAM role attached that grants it permission to read from the S3 bucket. The User Data script then uses the AWS CLI (aws s3 cp ...) to securely download the main script and execute it. Embedding keys (D) is a major security anti-pattern.

</details>

---

**7. After launching an EC2 instance, a developer notices that the web server software specified in the User Data script was not installed. Where should they look first to troubleshoot the issue?**

- A. The AWS CloudTrail logs for the `RunInstances` API call.
- B. The instance's system log in the EC2 console.
- C. The `/var/log/cloud-init-output.log` file on the instance itself.
- D. The S3 access logs for the bucket containing the AMI.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The cloud-init-output.log file captures the standard output and standard error of the User Data script execution. This is the primary place to look for error messages, such as a typo in a command, a package not being found, or a permissions issue, which would explain why the script failed. The instance's system log (B) can also contain clues but the cloud-init log is more specific.

</details>

---

**8. An EC2 instance is launched in a private subnet with no internet access. Its User Data script needs to install a package from an online repository. What will happen?**

- A. The instance will automatically use a NAT Gateway if one exists in the VPC.
- B. The `cloud-init` service will fail to run the script because it cannot reach the internet.
- C. The instance will connect to the internet via the Instance Metadata Service.
- D. The package installation command (e.g., `yum install`) will fail due to a lack of network connectivity to the repository.

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The User Data script itself will run, but the commands within it that require internet access will fail. A command like yum install needs to connect to remote package repositories over the internet. Without a NAT Gateway or other form of egress, this connection will time out, and the installation will fail. The script will likely continue to execute subsequent lines that don't require internet access.

</details>

---

**9. When is bootstrapping initiated on an EC2 instance?**

- A. Every time the instance is rebooted.
- B. Only on the first boot after the instance is launched.
- C. When an Elastic IP is attached to the instance.
- D. When the instance's IAM role is modified.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Bootstrapping via User Data is an initialization process. It's designed to configure the instance from a clean state into a desired state one time, right after it has been provisioned for the first time.

</details>

---

**10. You want to launch an EC2 instance and have it automatically join an existing Amazon EFS file system. Which of the following steps would likely be part of the User Data script?**

- A. Create a new EFS file system.
- B. Install the `amazon-efs-utils` package and run the `mount` command.
- C. Assign an IAM role with EFS permissions to the script.
- D. Configure the VPC route table to point to the EFS mount target.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** To mount an EFS file system, the EC2 instance needs the necessary client software (amazon-efs-utils) installed. The User Data script is the perfect place to run the installation command (e.g., yum install -y amazon-efs-utils) followed by the mount command to attach the file system to a local directory.

</details>

---

**11. A User Data script for a Windows EC2 instance should be formatted using what syntax?**

- A. It must be a JSON document.
- B. It must be enclosed in `<powershell>` tags for PowerShell scripts.
- C. It must start with `#!/bin/bash`.
- D. It must be a YAML document.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** On Windows, the EC2Launch/EC2Config service processes the User Data. To tell it to execute the content as a PowerShell script, you enclose the entire script within <powershell>...</powershell> tags. For a standard command prompt batch script, you would use <script>...</script>.

</details>

---

**12. What is the most secure way for a User Data script to get credentials for accessing other AWS services like S3 or DynamoDB?**

- A. Hard-code the IAM user access keys directly in the script.
- B. Store the credentials in a text file on the root volume of the AMI.
- C. Attach an IAM role to the EC2 instance, which provides temporary, automatically rotated credentials.
- D. Pass the credentials as command-line arguments to the script.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Hard-coding credentials (A, B, D) is a major security anti-pattern. The correct and secure method is to assign an IAM Role to the EC2 instance. The AWS CLI and SDKs running within the User Data script (and the instance in general) will automatically use the temporary credentials provided by this role, eliminating the need to manage static keys.

</details>

---

**13. A fleet of instances needs to be configured with a dynamic value, such as a database endpoint, that is only known after other infrastructure has been created. How can this value be passed to the instances' User Data scripts at launch?**

- A. This is not possible; User Data scripts must be static.
- B. Use a launch template and a tool like Terraform or CloudFormation to dynamically insert the endpoint's value into the User Data before launching the instance.
- C. Manually SSH into each instance and set an environment variable.
- D. The instance will automatically discover the database via the Instance Metadata Service.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Infrastructure as Code (IaC) tools are designed for this. A CloudFormation or Terraform template can first create the database, retrieve its endpoint address as an output, and then use that output as an input variable when defining the User Data for the EC2 instances that need to connect to it. This creates a fully automated and dynamic dependency chain.

## </details>

**14. An instance is part of an Auto Scaling group. The launch configuration includes a User Data script. What happens when the Auto Scaling group scales out and launches a new instance?**

- A. The User Data script from the launch configuration is executed on the new instance.
- B. The new instance is a perfect clone of a running instance, so the script does not run.
- C. The administrator receives an alert to manually configure the new instance.
- D. The User Data script from a randomly chosen existing instance is used.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Auto Scaling relies on launch templates or launch configurations to define how new instances should be created. The User Data script is a key part of this configuration. Every new instance launched by the Auto Scaling group will be a fresh instance created from the specified AMI, and it will execute the specified User Data script on its first boot.

## </details>

**15. To view the User Data script that was used to launch a running instance from within that instance, which command would you use?**

- A. `cat /etc/user-data.sh`
- B. `aws ec2 get-user-data --instance-id [id]`
- C. `curl http://169.254.169.254/latest/user-data`
- D. `cat /var/log/cloud-init.log`

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The EC2 Instance Metadata service provides a specific endpoint at http://169.254.169.254/latest/user-data that returns the exact, unmodified User Data script that was provided at launch. This can be useful for debugging or self-configuration.

## </details>

**16. What is the purpose of the `cloud-init` package on Linux AMIs provided by AWS?**

- A. To provide a firewall for the instance.
- B. To handle the execution of User Data scripts, set the hostname, create SSH keys, and perform other initialization tasks.
- C. To automatically back up the instance data to Amazon S3.
- D. To provide remote desktop access to the instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** cloud-init is the industry-standard multi-distribution package that handles early initialization of cloud instances. AWS uses it extensively in its official AMIs to process User Data, configure networking, set hostnames, and prepare the instance for use.

## </details>

**17. What happens to the User Data script if you create a custom AMI from an instance that has already run its User Data?**

- A. The User Data script is stored in the new AMI and will run again on instances launched from it.
- B. The User Data script is not included in the new AMI.
- C. The User Data script is included but disabled by `cloud-init`, and a new User Data script can be provided when launching from the new AMI.
- D. The original User Data script is encrypted and cannot be changed.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The results of the User Data execution (e.g., installed software) are baked into the new AMI. The cloud-init service on the new AMI knows that the original script has already been run. When you launch a new instance from this custom AMI, cloud-init is ready to accept and execute a new User Data script if you provide one.

## </details>

**18. A User Data script is failing with a "command not found" error. The command is for the AWS CLI. The instance is launched from a very minimal, non-AWS provided Linux AMI. What is the most likely reason for the failure?**

- A. The IAM role attached to the instance is missing permissions.
- B. The AWS CLI is not pre-installed on this minimal AMI.
- C. The instance does not have internet connectivity.
- D. The script is not running as root.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** While AWS's own AMIs (like Amazon Linux 2) come with the AWS CLI pre-installed, many other minimal or community AMIs do not. A common bootstrapping error is assuming a tool is present when it is not. The User Data script would first need to include steps to install the AWS CLI before it could use it.

## </details>

**19. You need to pass sensitive information, like a database password, to an instance at launch time for configuration. What is the most secure method?**

- A. Place the password directly in plain text in the User Data script.
- B. Use the User Data script to retrieve the password from a secure, encrypted location like AWS Secrets Manager or AWS Systems Manager Parameter Store.
- C. Email the password to the instance administrator after launch.
- D. Store the password in a file on a public S3 bucket and download it in the User Data script.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Placing secrets in plain text in User Data is a major security risk, as it can be viewed by anyone with permission to describe the instance configuration. The best practice is to store the secret in a managed service like AWS Secrets Manager. The instance's IAM role should be granted permission to read that specific secret, and the User Data script can then securely retrieve it at launch time.

## </details>

**20. The primary goal of bootstrapping is to:**

- A. Provide a hardware specification for an instance.
- B. Automate the initial configuration and software installation on a new instance.
- C. Create a secure network connection to an instance.
- D. Manage the billing for an instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Bootstrapping is all about automation. It's the process of taking a generic base image (the AMI) and automatically transforming it into a fully configured and operational server without manual intervention.

## </details>

**21. An EC2 instance's User Data script runs successfully, but an administrator needs to make a configuration change. They modify the User Data in the launch template and restart the running EC2 instance. Does the change take effect?**

- A. Yes, `cloud-init` detects the change and re-runs the script.
- B. No, User Data only runs on the first launch, so the running instance is unaffected. A new instance must be launched from the template.
- C. Yes, but the instance must be stopped and started, not just rebooted.
- D. No, launch templates cannot be modified after creation.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This emphasizes the "first boot only" rule. Modifying the launch template's User Data only affects new instances launched from that template in the future. It has no effect on already running instances, even if they were launched from that same template. To apply the change, the existing instance would need to be terminated and replaced.

## </details>

**22. An instance needs to download a configuration file from an S3 bucket that requires SSE-KMS encryption. What must be included in the instance's IAM role permissions?**

- A. Only `s3:GetObject` permission for the S3 bucket.
- B. Only `kms:Decrypt` permission for the KMS key.
- C. Both `s3:GetObject` for the bucket and `kms:Decrypt` for the KMS key used to encrypt the object.
- D. Only `s3:ListBucket` permission.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** When an object is encrypted with SSE-KMS, two permissions are needed to retrieve it. The instance needs s3:GetObject permission to access the object in S3, and it also needs kms:Decrypt permission on the specific KMS key that was used to encrypt the object. Without the KMS permission, S3 will not be able to decrypt the object for the instance, and the download will fail.

</details>
