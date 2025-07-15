# Step 29: Glacier & Archive Patterns

## 🎯 Objective
- [x] Master: **Glacier & Archive Patterns**

## 📘 Notes

### **Deep Dive: Glacier & Archive Patterns**

Amazon S3 Glacier is a secure, durable, and extremely low-cost storage service for data archiving and long-term backup. It's not designed for data that you need to access frequently or in real-time. Instead, it's for data you need to keep for months or years to meet compliance requirements or for future analysis. It's important to understand the different retrieval options and data protection features.

---

### **1. S3 Glacier Storage Classes** 🧊

These are the storage classes specifically designed for data archiving. They offer the lowest storage costs in exchange for longer retrieval times.

- **S3 Glacier Instant Retrieval:**
    - **Use Case:** For long-term data that is rarely accessed but requires immediate, millisecond retrieval when needed. Ideal for medical images, news media assets, or user-generated content archives.
    - **Retrieval Time:** Milliseconds (same performance as S3 Standard).
    - **Cost:** Higher retrieval cost than other Glacier tiers, but lower storage cost than S3 Standard-IA.
- **S3 Glacier Flexible Retrieval (The "classic" Glacier):**
    - **Use Case:** The standard archive solution for data that does not require immediate access. Perfect for backups, disaster recovery archives, and data where a retrieval time of minutes or hours is acceptable.
    - **Retrieval Time:** Offers flexible retrieval options:
        - **Expedited:** 1-5 minutes (for objects up to 250MB).
        - **Standard:** 3-5 hours.
        - **Bulk:** 5-12 hours (lowest cost retrieval option).
- **S3 Glacier Deep Archive:**
    - **Use Case:** The lowest-cost storage in all of AWS. Designed for very long-term retention (7-10+ years) of data that is almost never accessed. Think compliance archives (e.g., financial or healthcare records) or preserving raw data from scientific projects.
    - **Retrieval Time:**
        - **Standard:** Within **12 hours**.
        - **Bulk:** Within **48 hours**.
    - **Note:** There is no "Expedited" option for Deep Archive.

---

### **2. Data Protection & Compliance Features**

- **S3 Object Lock:**
    - **Purpose:** To enforce WORM (Write-Once, Read-Many) policies, preventing objects from being deleted or overwritten for a fixed amount of time or indefinitely. This is a key feature for meeting regulatory compliance.
    - **How it works:** You enable it on a bucket at the time of creation. You can then apply retention settings to individual objects.
    - **Modes:**
        1. **Governance Mode:** Protects objects from being deleted by most users, but users with special IAM permissions can still override the lock settings or delete the object.
        2. **Compliance Mode:** The highest level of protection. The object version **cannot be overwritten or deleted by any user, including the root user**, until its retention period expires. Its retention mode and period cannot be changed.
- **S3 Glacier Vault Lock:**
    - **Purpose:** The original compliance feature for S3 Glacier. It allows you to deploy an immutable compliance policy on a Glacier Vault.
    - **How it works:** You create a lockable policy (e.g., "deny all deletes" or "enforce MFA for deletes"), test it, and then lock it. Once locked, the policy **cannot be changed or removed by anyone**, including the root user.
    - **Object Lock vs. Vault Lock:** S3 Object Lock is the newer, more flexible, and recommended feature for WORM compliance, as it works across all S3 storage classes and is configured on the S3 bucket itself. Vault Lock applies only to direct-to-Glacier vaults.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A financial services company is required by law to store trade records for 10 years. The data will almost never be accessed. If access is needed, a 24-hour turnaround time is acceptable. To meet this requirement at the lowest possible cost, where should the data be stored?**

- A. S3 Standard-IA
- B. S3 Glacier Instant Retrieval
- C. S3 Glacier Flexible Retrieval
- D. S3 Glacier Deep Archive

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** This is the ideal use case for S3 Glacier Deep Archive. It's designed for long-term (7-10+ years) retention, offers the absolute lowest storage cost, and its standard retrieval time of up to 12 hours easily meets the 24-hour requirement.
</details>

---

**2. A medical imaging archive contains millions of files. The files are not accessed often, but when a doctor requests a patient's imaging history, the files must be retrieved and displayed with millisecond latency. Which S3 storage class is designed for this specific requirement?**

- A. S3 Standard
- B. S3 Glacier Instant Retrieval
- C. S3 Glacier Flexible Retrieval
- D. S3 Glacier Deep Archive

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The key requirements are long-term archival cost combined with immediate access. S3 Glacier Instant Retrieval is the only archive tier that provides the low storage costs of archival with the same millisecond, low-latency retrieval performance as S3 Standard.
</details>

---

**3. A company needs to enforce a strict WORM (Write-Once, Read-Many) policy on certain documents stored in S3 to meet regulatory requirements. Once a document is stored, it must be impossible for anyone, including the root user, to delete it for five years. Which S3 feature should be used?**

- A. S3 Versioning with MFA Delete.
- B. An S3 bucket policy with a `Deny` statement for the `s3:DeleteObject` action.
- C. S3 Object Lock in Compliance Mode.
- D. S3 Object Lock in Governance Mode.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** S3 Object Lock in Compliance Mode is designed for this highest level of data immutability. Once an object is locked in compliance mode, its retention period cannot be shortened, and the object cannot be deleted by any user, including the account root user, until the retention period expires. Governance mode (D) allows designated users to bypass the lock.
</details>

---

**4. An administrator needs to retrieve a 100 MB file from S3 Glacier Flexible Retrieval as quickly as possible. Which retrieval option should they choose?**

- A. Bulk
- B. Standard
- C. Expedited
- D. Instant

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** For S3 Glacier Flexible Retrieval, Expedited is the fastest retrieval option, typically returning data within 1-5 minutes for objects of this size. "Instant" is a feature of a different storage class (S3 Glacier Instant Retrieval).
</details>

---

**5. What is the standard retrieval time for data stored in S3 Glacier Deep Archive?**

- A. 1-5 minutes
- B. 3-5 hours
- C. Within 12 hours
- D. Within 48 hours

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The standard retrieval time for S3 Glacier Deep Archive is within 12 hours. There is also a lower-priority Bulk retrieval option that can take up to 48 hours. There is no Expedited option for Deep Archive.
</details>

---

**6. A company uses a lifecycle policy to move data from S3 Standard to S3 Glacier Flexible Retrieval after 90 days. An analyst now needs to query a 1-year-old archived log file using Amazon Athena. What must be done first?**

- A. Nothing, Athena can query data directly in S3 Glacier Flexible Retrieval.
- B. The lifecycle policy must be suspended.
- C. A restore request must be initiated for the object from Glacier, and the user must wait for the restore to complete.
- D. The object must be moved to S3 Glacier Deep Archive first.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Standard query services like Amazon Athena cannot query data directly while it is in an "archived" state in S3 Glacier Flexible Retrieval or Deep Archive. You must first initiate a restore operation. This creates a temporary, accessible copy of the object in S3 for a specified number of days, during which time it can be queried by Athena.
</details>

---

**7. What is the primary difference between S3 Object Lock and S3 Glacier Vault Lock?**

- A. Object Lock is for WORM compliance, while Vault Lock is for encryption.
- B. Object Lock is configured on an S3 bucket and works with all S3 storage classes, while Vault Lock is an older feature configured on a Glacier Vault directly.
- C. Vault Lock is more secure because it cannot be bypassed by the root user.
- D. Object Lock requires a monthly fee, while Vault Lock is free.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the key distinction. S3 Object Lock is the modern, recommended approach. It's a setting on an S3 bucket and integrates with features like lifecycle policies. S3 Glacier Vault Lock is a legacy feature that only applies to vaults that you write to directly using the Glacier API, not to objects transitioned from S3.
</details>

---

**8. A user initiates a "Bulk" retrieval from S3 Glacier Flexible Retrieval. What is the main advantage of this retrieval method?**

- A. It is the fastest retrieval option.
- B. It has the lowest cost per GB retrieved.
- C. It automatically restores the data to the S3 Standard class.
- D. It guarantees retrieval within one hour.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The trade-off for the slow retrieval time of a Bulk retrieval (5-12 hours) is that it is the most cost-effective retrieval option. It's ideal for large, non-urgent data retrievals where minimizing cost is the top priority.
</details>

---

**9. In S3 Object Lock, what is the difference between Governance Mode and Compliance Mode?**

- A. Governance mode is for long-term retention, while Compliance mode is for short-term legal holds.
- B. In Governance mode, users with special IAM permissions can bypass the lock, while in Compliance mode, no one (including root) can bypass the lock.
- C. Governance mode is more expensive than Compliance mode.
- D. Compliance mode encrypts the object, while Governance mode does not.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the core difference. Governance Mode provides strong protection but allows for a "break-glass" scenario where authorized administrators can remove the lock if necessary. Compliance Mode is for absolute immutability; once locked, the object is protected from deletion by everyone until the retention period ends.
</details>

---

**10. A lifecycle policy attempts to transition a 10 KB object from S3 Standard to S3 Standard-IA after 30 days. What will happen?**

- A. The object will be transitioned successfully.
- B. The transition will fail because the object is too small.
- C. The object will be deleted instead of transitioned.
- D. The object will be moved directly to S3 Glacier Flexible Retrieval.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** There is a minimum object size for transitions to Infrequent Access storage classes. Objects smaller than 128 KB are not eligible for transition via lifecycle policies to S3 Standard-IA or S3 One Zone-IA. S3 does not transition them because the cost savings on storage would be negated by the overhead and fees.
</details>

---

**11. Which S3 Glacier storage class does NOT offer an "Expedited" retrieval option?**

- A. S3 Glacier Instant Retrieval
- B. S3 Glacier Flexible Retrieval
- C. S3 Glacier Deep Archive
- D. All Glacier classes offer Expedited retrieval.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** S3 Glacier Deep Archive is designed for the absolute lowest cost, and the trade-off is slow retrieval times. It does not have a fast retrieval option like "Expedited." Its fastest retrieval is the "Standard" 12-hour option.
</details>

---

**12. When you initiate a restore from S3 Glacier Flexible Retrieval, what are you actually doing?**

- A. Permanently moving the object from Glacier back to S3 Standard.
- B. Creating a temporary copy of the object that is accessible in S3 for a specified number of days.
- C. Decrypting the object in place within the Glacier vault.
- D. Generating a pre-signed URL to access the object directly from Glacier.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A restore operation does not move the original archived object. It creates a temporary copy of the data in the S3 Standard-IA or S3 Standard storage class within the same bucket. You specify how many days this temporary copy should exist. After that period, the temporary copy is deleted, but the original archived object remains safe in Glacier.
</details>

---

**13. A company has enabled S3 Object Lock in Governance mode on an object with a retention period of 1 year. An administrator with the `s3:BypassGovernanceRetention` IAM permission tries to delete the object. What will be the outcome?**

- A. The deletion will fail because the object is locked.
- B. The deletion will be successful because the administrator has the special bypass permission.
- C. The administrator will have to enter an MFA code to delete the object.
- D. The object's retention period will be extended automatically.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is the definition of Governance Mode. It protects against accidental deletion from most users, but it provides an escape hatch. Users who are explicitly granted the s3:BypassGovernanceRetention permission in their IAM policy can override the lock and delete the object.
</details>

---

**14. An object is placed under a "Legal Hold" using S3 Object Lock. What does this mean?**

- A. The object is automatically transitioned to S3 Glacier Deep Archive.
- B. The object is protected by WORM settings and cannot be overwritten or deleted, even if its retention period expires.
- C. All access to the object is blocked until the hold is removed.
- D. The object can only be accessed by users with specific legal IAM roles.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Legal Hold is an independent lock that you can place on an object version. It provides the same WORM protection as a retention period, but it has no expiration date. An object under a legal hold remains protected until an authorized user explicitly removes the hold. This is useful for things like e-discovery, where you don't know how long the data needs to be preserved.
</details>

---

**15. What are the three retrieval speed options for S3 Glacier Flexible Retrieval?**

- A. Instant, Standard, and Bulk
- B. Expedited, Standard, and Bulk
- C. Fast, Normal, and Slow
- D. On-Demand, Provisioned, and Spot

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The three retrieval tiers for the "classic" S3 Glacier (now called Flexible Retrieval) are Expedited (minutes), Standard (hours), and Bulk (many hours, lowest cost).
</details>

---

**16. The primary purpose of S3 Glacier Vault Lock is to:**

- A. Encrypt the contents of a Glacier vault.
- B. Enforce compliance controls via an immutable, resource-based policy on a Glacier vault.
- C. Provide faster retrieval times for archived data.
- D. Prevent unauthorized users from seeing the contents of a vault.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Vault Lock is a compliance feature. It allows you to create a policy (e.g., to enforce WORM or require MFA for deletes) and then lock that policy so it can never be changed, providing a strong, auditable control for regulatory purposes.
</details>

---

**17. A company archives data directly to an S3 Glacier vault (not via an S3 lifecycle policy). This data is subject to strict compliance rules. Which feature should they use to enforce these rules?**

- A. S3 Object Lock
- B. S3 Bucket Policy
- C. S3 Glacier Vault Lock
- D. AWS Organizations SCP

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** Since the data is being written directly to a Glacier vault (the older method), the appropriate compliance feature is S3 Glacier Vault Lock. S3 Object Lock (A) applies to objects within an S3 bucket, which is the more modern approach.
</details>

---

**18. What is the most cost-effective retrieval method for a very large, non-urgent restore from S3 Glacier Deep Archive?**

- A. Expedited
- B. Standard
- C. Bulk
- D. There is only one retrieval method for Deep Archive.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** S3 Glacier Deep Archive offers two retrieval options: Standard (within 12 hours) and Bulk (within 48 hours). The Bulk option is the lowest-cost retrieval method in all of AWS and is ideal for massive, non-time-sensitive data recovery.
</details>

---

**19. You have enabled versioning on a bucket and have an object under S3 Object Lock in Compliance mode. Can you delete the object by applying a delete marker?**

- A. Yes, applying a delete marker is always allowed.
- B. No, you cannot apply a delete marker to an object protected by Compliance mode.
- C. Yes, but the underlying locked object version will not be deleted.
- D. No, you must first suspend versioning on the bucket.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** This is a key interaction. S3 Object Lock protects a specific version of an object. You can still perform a DELETE operation, which will place a delete marker as the new "current" version, making the file appear to be gone. However, the original object version that is under the Compliance lock remains completely protected and cannot be permanently deleted until its retention period expires.
</details>

---

**20. A company needs to store backups for 3 years. The backups are rarely accessed. Which storage class offers the best balance of low storage cost and reasonably fast retrieval (a few hours)?**

- A. S3 Standard-IA
- B. S3 Glacier Instant Retrieval
- C. S3 Glacier Flexible Retrieval
- D. S3 Glacier Deep Archive

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** S3 Glacier Flexible Retrieval is the classic archive solution that fits this "backup and restore" use case perfectly. Its storage costs are very low, and the "Standard" retrieval time of 3-5 hours is acceptable for most backup recovery scenarios. Deep Archive (D) would also work but might be slower than necessary.
</details>

---

**21. When can S3 Object Lock be enabled on a bucket?**

- A. At any time after the bucket is created.
- B. Only at the time of bucket creation.
- C. Only if the bucket has versioning suspended.
- D. It is enabled by default on all new buckets.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** To ensure the integrity of the feature, S3 Object Lock can only be enabled when you create the bucket. You cannot enable it on a bucket that already exists. It also requires that versioning be enabled on the bucket at the same time.
</details>

---

**22. Which is NOT a valid retrieval speed for S3 Glacier Flexible Retrieval?**

- A. Expedited
- B. Standard
- C. Bulk
- D. Instant

<details>
<summary>View Answer</summary>
<br>

**Answer: D**

**Explanation:** "Instant" retrieval is the defining feature of the S3 Glacier Instant Retrieval storage class. The retrieval tiers for S3 Glacier Flexible Retrieval are Expedited, Standard, and Bulk.
</details>

---