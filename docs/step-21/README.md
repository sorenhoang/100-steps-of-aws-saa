# Step 21: S3 Storage Classes & Basics

## 🎯 Objective

- [x] Master: **S3 Storage Classes & Basics**

## 📘 Notes

### **Deep Dive: S3 Storage Classes & Basics**

Amazon Simple Storage Service (S3) is a secure, durable, and highly-scalable object storage service. Instead of thinking of it as a traditional file system with folders, think of it as a key-value store for large objects. It's one of the oldest and most fundamental AWS services.

---

### **1. S3 Basics** 🪣

- **Objects:** These are the files you store in S3. An object consists of the data itself, a key, and metadata.
- **Key:** The "key" is simply the unique name of the object within a bucket (e.g., `images/puppy.jpg`).
- **Bucket:** A bucket is a container for objects. Bucket names must be **globally unique** across all of AWS, as they are part of the DNS namespace (e.g., `my-unique-bucket-name.s3.amazonaws.com`).
- **Durability and Availability:** S3 is designed for **99.999999999% (11 nines) of durability**. This means if you store 10,000,000 objects, you can on average expect to lose a single object once every 10,000 years. It achieves this by redundantly storing your data across multiple devices and at least three Availability Zones within a region. Different storage classes have different _availability_ SLAs.
- **Data Consistency:** S3 provides **strong read-after-write consistency** for new objects (PUTs) and **strong consistency** for overwrites (PUTs) and deletes (DELETEs). After a successful write or delete, any subsequent read request will immediately receive the latest version of the object.

---

### **2. S3 Storage Classes**

S3 offers a range of storage classes designed for different use cases. You choose a class based on your data's access patterns and how long you plan to store it.

- **S3 Standard:**
  - **Use Case:** The default, all-purpose storage class. Perfect for frequently accessed data that requires high performance and low latency, such as websites, content distribution, and big data analytics.
  - **Availability:** 99.99% SLA.
- **S3 Intelligent-Tiering:**
  - **Use Case:** For data with unknown, changing, or unpredictable access patterns. This is the "smart" storage class.
  - **How it works:** It automatically moves your data between two access tiers (a frequent access tier and an infrequent access tier) based on your usage patterns, optimizing costs for you. There is a small monthly monitoring and automation fee per object. It can also be configured to automatically archive data to Glacier tiers.
- **S3 Standard-Infrequent Access (S3 Standard-IA):**
  - **Use Case:** For data that is accessed less frequently but requires rapid access when needed. Good for long-term storage, backups, and disaster recovery files.
  - **Cost Model:** Lower storage cost than S3 Standard, but you are charged a **per-GB retrieval fee**.
- **S3 One Zone-Infrequent Access (S3 One Zone-IA):**
  - **Use Case:** Same as Standard-IA, but for non-critical, easily reproducible data.
  - **Key Difference:** Data is stored in only a **single Availability Zone** instead of three. This makes it cheaper than Standard-IA but less resilient. If that AZ is destroyed, the data is lost.
  - **Availability:** 99.5% SLA.

### **3. Archive Storage Classes** 🧊

- **S3 Glacier Instant Retrieval:**
  - **Use Case:** Long-term archival where you still need immediate, millisecond access. Think medical images or news media assets.
  - **Retrieval Time:** Milliseconds.
  - **Cost Model:** Very low storage cost, but a higher per-GB retrieval fee than S3-IA.
- **S3 Glacier Flexible Retrieval (Formerly S3 Glacier):**
  - **Use Case:** The classic archive solution. Good for backups and data archives where a retrieval time of a few minutes to several hours is acceptable.
  - **Retrieval Time:** Configurable. Expedited (1-5 minutes), Standard (3-5 hours), or Bulk (5-12 hours).
- **S3 Glacier Deep Archive:**
  - **Use Case:** The lowest-cost storage class in all of AWS. Designed for long-term retention (7-10 years or more) of data that is rarely, if ever, accessed. Think compliance archives or financial records.
  - **Retrieval Time:** Standard retrieval is within **12 hours**.

### **4. S3 Lifecycle Policies** 🔄

A lifecycle policy is a set of rules you configure on a bucket to automatically manage your objects. This is the primary way to optimize S3 costs.

- **How it works:** You create rules that transition objects from one storage class to another based on their age.
- **Example Policy:**
  1. **Transition Rule:** Move objects from S3 Standard to S3 Standard-IA 30 days after creation.
  2. **Transition Rule:** Move objects from S3 Standard-IA to S3 Glacier Flexible Retrieval 90 days after creation.
  3. **Expiration Rule:** Permanently delete the objects 7 years (2555 days) after creation.

This automates the process of moving data to cheaper storage tiers as it becomes less frequently accessed and eventually deleting it to comply with retention policies.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company needs to store critical legal documents for a required retention period of 7 years. The documents are rarely accessed, but if a request is made, a retrieval time of up to 24 hours is acceptable. The primary goal is to minimize storage costs. Which S3 storage class is the most appropriate?**

- A. S3 Standard
- B. S3 Standard-Infrequent Access (Standard-IA)
- C. S3 Glacier Flexible Retrieval
- D. S3 Glacier Deep Archive

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** This is a perfect use case for S3 Glacier Deep Archive. It offers the absolute lowest storage cost in AWS, which is ideal for long-term (7+ years) archival. The retrieval time of up to 12 hours fits well within the acceptable 24-hour window.

</details>

---

**2. What is the key difference between S3 Standard-IA and S3 One Zone-IA?**

- A. S3 One Zone-IA is designed for frequently accessed data.
- B. S3 One Zone-IA stores data in a single Availability Zone, making it less resilient but cheaper.
- C. S3 One Zone-IA has a faster retrieval time than S3 Standard-IA.
- D. S3 One Zone-IA does not have a per-GB retrieval fee.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The name gives it away. S3 One Zone-IA stores data in only one AZ, whereas S3 Standard and S3 Standard-IA store data across at least three AZs. This reduced redundancy makes One Zone-IA less expensive but also less durable; it's suitable for non-critical data that can be easily regenerated.

</details>

---

**3. A company has a large dataset with unpredictable and changing access patterns. Some data is accessed frequently for a month and then rarely touched, while other data is accessed sporadically. They want to automate cost savings without complex analysis. Which S3 storage class should they use?**

- A. S3 Standard
- B. S3 Intelligent-Tiering
- C. S3 Standard-IA
- D. S3 One Zone-IA

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 Intelligent-Tiering is designed specifically for data with unknown or changing access patterns. It automatically moves objects between a frequent access tier and an infrequent access tier based on usage, providing cost savings without operational overhead or retrieval fees.

</details>

---

**4. How does S3 achieve its high durability of 99.999999999%?**

- A. By creating snapshots of data every hour.
- B. By requiring users to enable versioning on all buckets.
- C. By transparently storing copies of objects across multiple devices and multiple Availability Zones within a region.
- D. By encrypting all data by default.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** S3's famous "11 nines" of durability comes from its architecture. When you upload an object, S3 automatically creates and stores redundant copies on different devices across a minimum of three Availability Zones. This ensures that even the loss of an entire AZ will not result in data loss.

</details>

---

**5. A hospital needs to store patient medical imagery for long-term archival. The data is rarely accessed, but when a doctor needs an image, it must be available in milliseconds. Which storage class meets these requirements?**

- A. S3 Standard-IA
- B. S3 Glacier Flexible Retrieval
- C. S3 Glacier Deep Archive
- D. S3 Glacier Instant Retrieval

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The key requirements are long-term archival (low storage cost) and "millisecond" access. S3 Glacier Instant Retrieval is the only archive tier designed for this. It provides the low cost of archival storage with the same low-latency, high-throughput performance as S3 Standard.

</details>

---

**6. A company wants to automatically move log files from S3 Standard to S3 Glacier Flexible Retrieval after 90 days and then permanently delete them after 365 days. What S3 feature should be used to automate this process?**

- A. S3 Bucket Policies
- B. S3 Cross-Region Replication
- C. S3 Lifecycle Policies
- D. S3 Event Notifications

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the primary function of S3 Lifecycle Policies. You can create rules that define actions (like transitioning to another storage class or expiring objects) to be taken on objects after a certain number of days since their creation.

</details>

---

**7. What is a key characteristic of an S3 bucket name?**

- A. It must be unique only within your AWS account.
- B. It must be globally unique across all AWS accounts.
- C. It can contain uppercase letters.
- D. It is case-sensitive.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 bucket names must be globally unique. This is because they can be part of a DNS-addressable URL (e.g., https://my-bucket.s3.us-east-1.amazonaws.com). No two accounts in the world can have a bucket with the same name.

</details>

---

**8. What happens immediately after you upload a new object to S3?**

- A. There is a brief period of eventual consistency before the object can be read.
- B. The object is immediately available to be read due to strong read-after-write consistency.
- C. The object is placed in S3 One Zone-IA by default.
- D. The object is automatically replicated to another region.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 offers strong read-after-write consistency for new object PUTS. This means that as soon as the upload (write) operation returns a success message, any subsequent read request for that object is guaranteed to retrieve the object's data.

</details>

---

**9. Which S3 storage class charges a per-GB retrieval fee?**

- A. S3 Standard
- B. S3 Intelligent-Tiering
- C. S3 Standard-IA
- D. All of the above.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The trade-off for the lower storage price of infrequent access classes like S3 Standard-IA is a fee for retrieving the data. S3 Standard has no retrieval fee. S3 Intelligent-Tiering has no retrieval fee for moving data between its active tiers but does if you retrieve from its optional archive tiers.

</details>

---

**10. You are creating a lifecycle policy to move objects to S3 Glacier Flexible Retrieval. What is the minimum number of days an object must be stored in its current class before it can be transitioned?**

- A. 0 days
- B. 30 days
- C. 90 days
- D. 180 days

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** For transitions to S3 Standard-IA or S3 Glacier Instant Retrieval, the minimum storage duration is 30 days. For transitions to S3 Glacier Flexible Retrieval, the minimum is also typically aligned with a 30-day minimum in the source class. The exam will test the general knowledge that you cannot transition objects immediately (0 days) to most IA/Archive tiers.

</details>

---

**11. An object is stored in S3 Glacier Flexible Retrieval. The user requests an "Expedited" retrieval. How long will it typically take to access the object?**

- A. Milliseconds
- B. 1-5 minutes
- C. 3-5 hours
- D. Within 12 hours

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 Glacier Flexible Retrieval offers three retrieval speeds. Expedited is the fastest, typically taking 1-5 minutes (for objects up to 250MB). Standard takes 3-5 hours, and Bulk takes 5-12 hours.

</details>

---

**12. When is S3 One Zone-IA a suitable choice over S3 Standard-IA?**

- A. When you need the highest possible durability for critical data.
- B. When you are storing secondary backup copies or easily reproducible data and want to save costs.
- C. When data is accessed very frequently.
- D. When you need to store data across multiple AWS Regions.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The lower cost of S3 One Zone-IA comes at the cost of reduced durability (as it's only in one AZ). Therefore, it's only appropriate for data that is not critical or that can be easily regenerated if the single Availability Zone fails. Storing a secondary backup copy is a perfect example.

</details>

---

**13. A company is hosting a static website on Amazon S3. Which storage class should be used for the website's files (HTML, CSS, images)?**

- A. S3 Standard
- B. S3 Glacier Deep Archive
- C. S3 One Zone-IA
- D. S3 Intelligent-Tiering

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** A public website requires low-latency, frequent access to its assets. S3 Standard is designed for this high-performance use case. The archive and infrequent access tiers would be too slow and would incur retrieval fees, making them unsuitable.

</details>

---

**14. What does the term "object" refer to in S3?**

- A. A folder or directory.
- B. A file and its associated metadata.
- C. A container for other objects.
- D. An EBS snapshot.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** In S3, the fundamental entity is the object. An object is composed of the data itself (the file) plus metadata, which includes system metadata (like content type, size) and optional user-defined metadata (tags).

</details>

---

**15. A lifecycle policy is configured to move objects to S3 Standard-IA after 30 days. If a 15-day-old object is deleted manually, what happens?**

- A. The object is transitioned to S3 Standard-IA immediately and then deleted.
- B. The lifecycle policy rule is ignored, and the object is deleted permanently.
- C. The deletion fails because the object is less than 30 days old.
- D. The object is marked for deletion but not removed until it is 30 days old.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A lifecycle policy rule is an automated, time-based action. It does not prevent manual operations. If a user with the correct permissions deletes the object, it is deleted immediately, and the lifecycle rule will no longer apply because the object no longer exists.

</details>

---

**16. What is the default storage class for an object when it is first uploaded to S3 without specifying a class?**

- A. S3 Intelligent-Tiering
- B. S3 Standard-IA
- C. S3 Standard
- D. S3 One Zone-IA

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Unless you specify otherwise during the upload or have a bucket policy that dictates a different class, all new objects are placed in the S3 Standard storage class by default.

</details>

---

**17. Which statement about S3 Glacier Deep Archive is TRUE?**

- A. It is designed for data that needs to be retrieved in under an hour.
- B. It offers the lowest object storage cost of any S3 storage class.
- C. It stores data in a single Availability Zone.
- D. It has no retrieval fees.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The primary purpose of S3 Glacier Deep Archive is to provide extremely low-cost storage for long-term data retention. Its retrieval time is up to 12 hours (A is false), it is regionally redundant (C is false), and it has retrieval fees (D is false).

</details>

---

**18. What is the minimum object size for an object to be eligible for S3 Intelligent-Tiering?**

- A. There is no minimum size.
- B. 1 KB
- C. 128 KB
- D. 1 MB

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Objects of any size can be stored in the S3 Intelligent-Tiering storage class. However, it's important to note that very small objects (less than 128 KB) will not be auto-tiered and will be billed at the frequent access tier rate. This is a subtle but important detail. The class accepts objects of any size, but the auto-tiering benefit only applies to objects 128 KB or larger.

</details>

---

**19. How does Amazon S3's data consistency model handle overwriting an existing object?**

- A. It offers eventual consistency, so a read might get the old or new data for a short time.
- B. It offers strong consistency, so any read request initiated after the overwrite completes is guaranteed to get the new data.
- C. It blocks all reads to the object while the overwrite is in progress.
- D. It creates a new version of the object instead of overwriting.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** As of a recent update, S3 now provides strong consistency for all operations, including overwriting existing objects (PUTs) and deleting objects (DELETEs). This simplifies application development as you no longer need to account for eventually consistent reads after an overwrite.

</details>

---

**20. A company needs to store data that is not critical and can be easily re-created. Their primary goal is to minimize storage costs for infrequently accessed data. Which is the MOST cost-effective storage class?**

- A. S3 Standard
- B. S3 Standard-IA
- C. S3 One Zone-IA
- D. S3 Intelligent-Tiering

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Given that the data is non-critical and can be re-created, the reduced durability of S3 One Zone-IA is an acceptable trade-off for its lower cost compared to S3 Standard-IA. It is specifically designed for this type of use case.

</details>

---

**21. A lifecycle policy is configured with two rules: 1) Transition to Standard-IA after 30 days. 2) Expire (delete) after 365 days. What happens to a 400-day-old object in the bucket?**

- A. Nothing, as it is too old for the transition rule.
- B. It will be transitioned to Standard-IA.
- C. It will be deleted.
- D. It will be moved to Glacier.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Lifecycle policies apply based on the object's creation date. Since the object is 400 days old, it has already passed the 365-day expiration threshold. S3 will identify it as eligible for the expiration action and will queue it for permanent deletion.

</details>

---

**22. Which of the following is a "key" in the context of Amazon S3?**

- A. The encryption key used to secure the object.
- B. The unique name or path of an object within a bucket.
- C. A tag used for cost allocation.
- D. The access key ID of an IAM user.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** In S3 terminology, the key is the full name of the object, which can include path-like prefixes (e.g., documents/2025/report.pdf). The combination of the bucket name, the object key, and optionally a version ID uniquely identifies an object.

</details>
