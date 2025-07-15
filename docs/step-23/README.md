# Step 23: S3 Encryption, Static Website Hosting

## 🎯 Objective

- [x] Master: **S3 Encryption, Static Website Hosting**

## 📘 Notes

Of course. This lecture covers how to secure your S3 data at rest and how to use S3 for a very common and cost-effective use case: hosting a static website. Let's begin.

### **Deep Dive: S3 Encryption & Static Website Hosting**

Securing data is a top priority, and S3 provides multiple robust options for encryption. Separately, S3's ability to serve content directly over HTTP makes it an ideal platform for hosting simple, serverless websites.

---

### **1. S3 Encryption** 🔐

S3 provides multiple layers of encryption to protect your data both in transit and at rest.

- **Encryption in Transit:** This is achieved by using SSL/TLS. Any time you upload or download data from S3 using HTTPS (the standard for API calls and console access), the connection is encrypted. You can enforce this by creating a bucket policy that denies any request where `aws:SecureTransport` is `false`.
- **Encryption at Rest (Server-Side Encryption):** This refers to the encryption of data while it is stored on disks in AWS data centers. S3 offers three main options for server-side encryption (SSE).
  1. **SSE-S3 (Server-Side Encryption with S3-Managed Keys):**
     - **How it works:** S3 manages the entire encryption process for you. Each object is encrypted with a unique key, and those keys are themselves encrypted by a master key that is regularly rotated by S3. AWS manages all the keys.
     - **Control:** This is the simplest option with the least overhead. You just have to enable it.
     - **Header:** `x-amz-server-side-encryption: AES256`
  2. **SSE-KMS (Server-Side Encryption with AWS Key Management Service Keys):**
     - **How it works:** Encryption is handled by S3, but it uses keys that are managed within your **AWS Key Management Service (KMS)**.
     - **Control:** This option gives you much more control. You can use an AWS Managed Key (`aws/s3`) or a Customer Managed Key that you create. With a Customer Managed Key, you can manage its lifecycle, define its access policy (key policy), and audit its usage in CloudTrail. This provides a clear audit trail of who used your key to access which object.
     - **Benefit:** Provides centralized control over keys and an audit trail, which is often a requirement for compliance.
     - **Header:** `x-amz-server-side-encryption: aws:kms`
  3. **SSE-C (Server-Side Encryption with Customer-Provided Keys):**
     - **How it works:** You provide your own encryption key as part of the upload request. S3 uses this key to encrypt the object, and then **immediately discards the key**.
     - **Control:** You are fully responsible for managing and securing your encryption keys. To retrieve the object, you must provide the **exact same key** with the GET request. If you lose the key, you lose the object forever.
     - **Use Case:** For users who have a strict requirement to manage their own encryption keys outside of AWS.

---

### **2. S3 Static Website Hosting** 🌐

S3 can be configured to serve static web content (HTML, CSS, JavaScript, images) directly from a bucket. This is an extremely cost-effective and scalable way to host websites that do not require server-side processing (like PHP, Java, or .NET).

- **Configuration Steps:**
  1. **Create a Bucket:** Create a bucket with a name that often matches the domain name (e.g., `www.example.com`).
  2. **Enable Static Website Hosting:** In the bucket properties, enable this feature and specify the index document (e.g., `index.html`) and an optional error document (e.g., `error.html`).
  3. **Make Objects Public:** For a public website, the objects must be accessible. This is typically done by creating a **bucket policy** that allows public `s3:GetObject` access for all objects in the bucket.
  4. **Disable "Block Public Access":** You must turn off the "Block all public access" settings for the bucket to allow the bucket policy to take effect.
  5. **DNS:** Use the unique website endpoint URL provided by S3 (e.g., `http://my-bucket.s3-website-us-east-1.amazonaws.com`) and point your custom domain to it using a CNAME or Alias record in Route 53.
- **Securing with CloudFront and OAI (Origin Access Identity):**
  - Hosting a public website directly from S3 requires the bucket itself to be public. A more secure and performant best practice is to use **Amazon CloudFront** as a Content Delivery Network (CDN) in front of your S3 bucket.
  - **How it works:**
    1. You keep the S3 bucket **private** (Block Public Access is ON).
    2. You create a CloudFront distribution and set the S3 bucket as its origin.
    3. You create a special CloudFront user called an **Origin Access Identity (OAI)**.
    4. You modify your S3 bucket policy to grant `s3:GetObject` permission **only** to this specific OAI.
  - **Result:** Users access your website via the CloudFront URL. CloudFront fetches the content from the private S3 bucket using its special OAI permissions. This means users never access the S3 bucket directly, providing a secure perimeter and the performance benefits of a global CDN.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to encrypt its data at rest in S3. They have a strict compliance requirement to control access to the encryption keys and to have a full audit trail of when each key is used. Which S3 encryption method should they use?**

- A. SSE-S3
- B. SSE-KMS with a Customer Managed Key
- C. SSE-C
- D. Client-Side Encryption

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** SSE-KMS is designed for this use case. Using a Customer Managed Key allows you to define a granular key policy to control who can use the key. Crucially, all API calls to KMS (like kms:Decrypt which S3 calls on your behalf) are logged in AWS CloudTrail, providing the required audit trail. SSE-S3 (A) does not provide this level of control or auditability.

</details>


---

**2. A company is hosting a static website in an S3 bucket. They want to improve performance for global users and secure the bucket so it is not publicly accessible. What is the best practice to achieve this?**

- A. Make the S3 bucket public and enable Cross-Region Replication.
- B. Use an Application Load Balancer in front of the S3 bucket.
- C. Use Amazon CloudFront with an Origin Access Identity (OAI) to access a private S3 bucket.
- D. Enable S3 Transfer Acceleration on the bucket.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This describes the standard best-practice architecture. CloudFront caches the static content at edge locations around the world, providing low latency for global users. The Origin Access Identity (OAI) is a special CloudFront principal that can be granted permission to read from your S3 bucket. This allows you to keep the bucket itself private while allowing CloudFront to serve its content.

</details>


---

**3. When using Server-Side Encryption with Customer-Provided Keys (SSE-C), who is responsible for managing the encryption keys?**

- A. AWS S3
- B. AWS KMS
- C. The customer
- D. AWS IAM

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** With SSE-C, the customer is fully responsible for creating, managing, rotating, and securing the encryption keys. The customer must provide the key with every upload and download request. AWS never stores the key; it uses it only for the encryption/decryption operation and then immediately discards it.

</details>


---

**4. An administrator has enabled static website hosting on an S3 bucket and uploaded an `index.html` file. When they try to access the website endpoint URL, they get a 403 Forbidden error. The bucket has "Block all public access" enabled. What is the most likely cause?**

- A. The `index.html` file is corrupted.
- B. A bucket policy that allows public `s3:GetObject` action is missing or being blocked.
- C. The bucket name contains a period (`.`).
- D. The IAM user does not have permission to view the website.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For S3 to serve objects publicly as a website, two things are needed: 1) The "Block Public Access" setting must be turned OFF, and 2) A bucket policy must be in place that explicitly grants Allow to the s3:GetObject action for anonymous users ("Principal": "\*"). A 403 Forbidden error is the classic symptom of a permissions issue.

</details>


---

**5. What is the simplest way to enforce encryption for all new objects uploaded to an S3 bucket without managing any keys?**

- A. Enable default encryption on the bucket using SSE-S3 (AES-256).
- B. Create a bucket policy that denies puts if the `aws:kms` header is not present.
- C. Use SSE-C and provide a key with every upload.
- D. Manually encrypt each object before uploading.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** SSE-S3 is the most straightforward encryption method. By enabling it as the default encryption setting on the bucket, every new object uploaded will be automatically encrypted at rest by S3, with AWS managing all the keys. This requires zero client-side changes or key management overhead.

</details>


---

**6. What is an Origin Access Identity (OAI)?**

- A. An IAM user that has permission to access CloudFront.
- B. An IAM role that CloudFront assumes to access S3.
- C. A special CloudFront user identity that can be associated with a distribution to allow it to read objects from a private S3 bucket.
- D. The DNS name for the S3 bucket origin.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** An Origin Access Identity (OAI) is a virtual user identity created for use with CloudFront. Its sole purpose is to be used in an S3 bucket policy as a Principal. This allows you to lock down your bucket so that only CloudFront can access it, securing your origin.

</details>


---

**7. An EC2 instance needs to download an object from S3 that was encrypted using SSE-KMS with a specific Customer Managed Key. What permissions does the instance's IAM role need?**

- A. Only `s3:GetObject` permission on the object.
- B. Only `kms:Decrypt` permission on the KMS key.
- C. Both `s3:GetObject` permission on the object AND `kms:Decrypt` permission on the KMS key.
- D. The instance needs to be in the same region as the KMS key.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** To retrieve a KMS-encrypted object, the principal needs permission for two actions. It needs s3:GetObject to request the file from S3. S3 then needs permission to call KMS on the principal's behalf. Therefore, the principal's IAM role must also have kms:Decrypt permission for the specific key that was used for encryption.

</details>


---

**8. What happens to the encryption key when using SSE-C after an object is uploaded?**

- A. S3 stores the key securely in KMS.
- B. S3 stores a hash of the key to validate future requests.
- C. S3 discards the key completely after using it to encrypt the object.
- D. S3 encrypts the key with a master key and stores it with the object.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A key characteristic of SSE-C is that S3 does not store the customer's key in any form. It uses the provided key in memory to perform the encryption and then immediately and permanently discards it. The customer is entirely responsible for retaining the key for future decryptions.

</details>


---

**9. When configuring an S3 bucket for static website hosting, what is the "Index Document"?**

- A. A JSON file that lists all the contents of the bucket.
- B. The default webpage that is returned when a user requests the root URL of the website.
- C. The webpage that is shown when a requested object is not found.
- D. A document that must be named `s3-index.html`.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The Index Document (commonly index.html) is the default page served when a request is made to the root of the website or a sub-directory. For example, when a user browses to http://www.example.com/, S3 serves the index.html file.

</details>


---

**10. Which type of encryption occurs on the server side after the data has been received by AWS?**

- A. Client-Side Encryption
- B. Encryption in Transit (SSL/TLS)
- C. Server-Side Encryption (SSE)
- D. All of the above.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Server-Side Encryption (SSE) means that AWS (specifically, the S3 service in this case) is responsible for performing the encryption on the data after it arrives at the AWS data center. Client-side encryption, by contrast, happens on the user's machine before the data is sent to AWS.

</details>


---

**11. Why might a company choose SSE-KMS over SSE-S3 for encryption?**

- A. SSE-KMS is significantly faster than SSE-S3.
- B. SSE-S3 does not support automatic key rotation.
- C. SSE-KMS provides an audit trail via CloudTrail and allows for centralized management and control of the key policy.
- D. SSE-KMS is free, whereas SSE-S3 has a monthly cost.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The primary advantages of SSE-KMS are control and auditability. It allows you to create and manage your own keys, define who can use them via key policies, and track every use of the key in CloudTrail logs. This is often required for regulatory compliance.

</details>


---

**12. A user uploads an object to S3 using the HTTPS endpoint. The bucket has default encryption enabled with SSE-S3. Which statements are true about the data's encryption? (Select TWO)**

- A. The data is encrypted in transit.
- B. The data is encrypted at rest.
- C. The data is only encrypted at rest, not in transit.
- D. The user must provide an encryption key.
- E. The data is not encrypted unless SSE-KMS is used.

<details>
<summary>View Answer</summary>

**Answers: A and B**

**Explanation:** Using the HTTPS endpoint ensures the data is encrypted in transit via SSL/TLS. Because the bucket has default encryption enabled, S3 will automatically encrypt the data at rest once it is received.

</details>


---

**13. What is the primary function of the "Error Document" in S3 static website hosting configuration?**

- A. To log all errors that occur on the website.
- B. To display a custom error page (e.g., a custom 404 Not Found page) when a user requests an object that doesn't exist.
- C. To redirect users to the index page if an error occurs.
- D. To send an email notification to the administrator on an error.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The Error Document setting allows you to specify a custom HTML page that S3 will serve when an error, most commonly a 404 Not Found, occurs. This provides a better user experience than the default XML error message.

</details>


---

**14. What is required to use your own domain name (e.g., `www.my-cool-site.com`) for an S3 static website?**

- A. You must name the S3 bucket exactly `www.my-cool-site.com`.
- B. You must configure DNS records (like a CNAME or a Route 53 Alias record) to point your domain to the S3 website endpoint.
- C. You must use an Application Load Balancer.
- D. You must enable versioning on the S3 bucket.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 provides a long, functional endpoint URL. To use your own friendly domain name, you must configure your domain's DNS settings to point to that S3 endpoint. Using an Alias record in Route 53 is the preferred method as it's more efficient and can be used with a root domain. While naming the bucket after the domain (A) is a common convention and required for some configurations, the DNS record is the essential step that directs traffic.

</details>


---

**15. A developer wants to enforce that any object uploaded to a bucket MUST be encrypted. If an unencrypted object is uploaded, the request should be rejected. How can this be enforced?**

- A. By enabling default encryption on the bucket.
- B. By creating an S3 bucket policy that denies `s3:PutObject` if the `x-amz-server-side-encryption` header is not present.
- C. By using S3 inventory reports to find unencrypted objects.
- D. By enabling MFA Delete.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Default encryption (A) is useful, but it doesn't enforce encryption on the client side; it just applies it if nothing else is specified. To strictly enforce it and reject any unencrypted uploads, you must use a bucket policy. The policy can check for the presence of the encryption header on the PutObject request and deny the action if the header is missing.

</details>


---

**16. Which S3 encryption option requires the client to manage both the encryption key and the encryption/decryption process itself?**

- A. SSE-S3
- B. SSE-KMS
- C. SSE-C
- D. Client-Side Encryption

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** In Client-Side Encryption, the data is encrypted on the client's machine before it is ever sent to S3. The client application manages the keys and performs the cryptographic operations. S3 simply stores the resulting encrypted data as a meaningless blob of bytes. This provides the ultimate level of control and security but also the most management overhead.

</details>


---

**17. When using CloudFront with a private S3 bucket and an OAI, who is the `Principal` in the S3 bucket policy?**

- A. The AWS account root user.
- B. The ARN of the CloudFront distribution.
- C. The canonical user ID of the Origin Access Identity.
- D. A wildcard \*.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The bucket policy needs to grant permission to the OAI itself. The correct way to specify this in the Principal block of the policy is by using the OAI's canonical user ID, which looks something like: "Principal": {"CanonicalUser": "ID_for_the_OAI"}.

</details>


---

**18. What is the format of the S3 static website endpoint?**

- A. It is the same as the regular S3 REST API endpoint.
- B. `s3-website-<region>.amazonaws.com/<bucket-name>`
- C. `<bucket-name>.s3-website-<region>.amazonaws.com`
- D. `cloudfront.net/<bucket-name>`

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The static website endpoint has a specific format that includes the bucket name, the keyword "s3-website", the region, and the AWS domain. For example: http://my-bucket.s3-website-us-east-1.amazonaws.com. This is different from the REST API endpoint.

</details>


---

**19. A company wants to protect the sensitive data in their S3 bucket. They want AWS to handle all key management and rotation automatically with no extra configuration or cost. What is the best encryption option?**

- A. SSE-C
- B. SSE-S3
- C. SSE-KMS with a Customer Managed Key
- D. Do not encrypt the data.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** SSE-S3 perfectly matches these requirements. It provides strong encryption where AWS manages all aspects of the keys, including creation, management, and rotation, with no additional cost and minimal configuration (just a checkbox to enable it).

</details>


---

**20. A user is trying to download an object that was encrypted with SSE-C. What must they provide in their request?**

- A. Their IAM user access key and secret key.
- B. The original, unencrypted object as a hash.
- C. The exact same encryption key that was used to upload the object.
- D. A temporary token from AWS STS.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** With SSE-C, the customer holds the key. To decrypt and download the object, the client must provide the identical encryption key in the request headers. If the key is lost, the data is irrecoverable.

</details>


---

**21. Can a single S3 bucket be used to host multiple distinct static websites?**

- A. No, one bucket can only host one website.
- B. Yes, by creating different folders in the bucket and pointing different domains to them.
- C. Yes, if you use a CloudFront distribution with multiple behaviors that route different paths to different origin folders.
- D. Yes, by enabling versioning on the bucket.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** While an S3 bucket itself can only have one website configuration, you can achieve this with CloudFront. You can create a single CloudFront distribution, set up multiple CNAMEs (e.g., site1.example.com, site2.example.com), and then create different "behaviors" (routing rules) that map requests for each domain to a different folder within the single S3 bucket origin.

</details>


---

**22. Which part of the S3 service provides the "11 nines" (99.999999999%) guarantee?**

- A. Availability
- B. Durability
- C. Latency
- D. Throughput

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The "11 nines" refers to the durability of your objects. It is a measure of the likelihood of losing an object. Availability refers to the ability to access the object and has a different SLA (e.g., 99.99% for S3 Standard), which is a measure of uptime.

</details>
