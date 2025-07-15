# Step 22: S3 Versioning & Lifecycle Policies

## 🎯 Objective

- [x] Master: **S3 Versioning & Lifecycle Policies**

## 📘 Notes

### **Deep Dive: S3 Versioning & Lifecycle Policies**

While S3 is incredibly durable by default, protecting your data from accidental user deletion or logical corruption requires additional configuration. S3 Versioning provides a safety net against these errors, while Lifecycle Policies automate data management and cost optimization over time.

---

### **1. S3 Versioning** 🔄

S3 Versioning is a bucket-level feature that allows you to keep multiple variants of an object in the same bucket. When you enable versioning for a bucket, S3 automatically creates a unique version ID for every object, every time it is modified.

- **How it Works:**
  - Once enabled on a bucket, it cannot be disabled, only **suspended**.
  - **Overwriting an object:** Instead of replacing the existing object, S3 creates a new version with a new version ID. The old version is still preserved.
  - **Deleting an object:** When you perform a standard `DELETE` operation on an object in a versioned bucket, the object is not actually deleted. Instead, S3 inserts a **delete marker**, which becomes the new "current" version of the object. To the user, the object appears to be gone.
- **Key Benefits:**
  1. **Undelete / Recovery:** To "undelete" an object, you simply delete the delete marker. The previous version of the object will then become the current version again, making it visible and accessible.
  2. **Rollback:** You can easily recover from application failures or human errors by restoring a previous version of an object. If a bad file is uploaded, you can simply make an older, good version the current one.
  3. **Data Retention:** Provides a simple way to archive data, as all historical versions are preserved.
- **Managing Versions:**
  - By default, a `GET` request retrieves the most current version of an object. To retrieve a specific older version, you must specify its version ID in the request.
  - Each version of an object is a full object and is billed for its storage. A 1 GB file with 3 versions will consume 3 GB of storage space.

---

### **2. S3 Lifecycle Policies** 🗓️

A lifecycle policy is a set of rules you configure on a bucket to automatically manage your objects throughout their lifecycle. This is the primary tool for automating cost savings and enforcing data retention rules.

- **How it Works:** You define rules that trigger actions on objects based on their age or, if versioning is enabled, on the number of noncurrent versions.
- **Two Types of Actions:**
  1. **Transition Actions:** These actions move objects from one storage class to a cheaper one.
     - **Common Transitions:**
       - S3 Standard ➡️ S3 Standard-IA (after 30 days)
       - S3 Standard-IA ➡️ S3 Glacier Instant Retrieval (after 60 days)
       - S3 Glacier Instant Retrieval ➡️ S3 Glacier Flexible Retrieval (after 90 days)
       - S3 Glacier Flexible Retrieval ➡️ S3 Glacier Deep Archive (after 180 days)
  2. **Expiration Actions:** These actions permanently delete objects.
     - **`Expire current object version`:** Deletes the current version of an object after a specified number of days.
     - **`Permanently delete noncurrent versions`:** Deletes old, noncurrent versions of objects after they have been noncurrent for a specified number of days. This is a crucial cost-saving measure for versioned buckets.
     - **`Delete expired object delete markers`:** Cleans up leftover delete markers in a versioned bucket to keep listings clean.
- **Lifecycle Policies with Versioning:**
  - When versioning is enabled, lifecycle rules can be configured to act specifically on **current** and **noncurrent (old)** versions.
  - A very common and important use case is to create a rule to **permanently delete noncurrent versions** after a certain period (e.g., 90 days). This prevents your bucket from accumulating a huge number of old, hidden versions and running up storage costs.

Example Use Case:

An organization wants to keep daily log files. They need to keep the logs for one year for analysis and then archive them for 7 years for compliance before deleting them. A lifecycle policy would be perfect:

1. **Rule 1:** After 365 days, transition objects to S3 Glacier Deep Archive.
2. **Rule 2:** After 2555 days (approx. 7 years), expire (delete) the objects.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A developer accidentally deletes a critical configuration file from an S3 bucket. S3 Versioning was enabled on the bucket. How can the developer recover the file?**

- A. The file is permanently gone and must be restored from a backup.
- B. By deleting the "delete marker" for the object.
- C. By creating a new object with the same name.
- D. By contacting AWS Support to restore the object.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When you delete an object in a versioned bucket, S3 doesn't actually remove the data. It places a delete marker on top of the object stack, making it the "current" version, which makes the object appear deleted. To recover the file, you simply need to find and delete this specific delete marker. The previous version of the file will then become the current, visible version again.

</details>

---

**2. A company enables versioning on an S3 bucket to protect against accidental overwrites. An engineer uploads a 10 MB file, then uploads a new 10 MB version of the same file, and finally deletes the file. How much storage is consumed in S3?**

- A. 0 MB, because the file was deleted.
- B. 10 MB, for the most recent version.
- C. 20 MB, for the two historical versions of the object.
- D. 30 MB, because the delete marker also consumes storage.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Each version of an object is a complete object and is billed for its full size. Uploading the first file creates one 10 MB version. Uploading the second file creates another 10 MB version (the first one becomes noncurrent). The delete marker itself is very small and does not consume significant storage like a full object version does. Therefore, the total storage consumed is 20 MB from the two object versions.

</details>

---

**3. What is the primary purpose of an S3 Lifecycle Policy?**

- A. To automatically enable versioning on a bucket after a certain period.
- B. To define rules to automatically transition objects to different storage classes or delete them based on their age.
- C. To grant users access to specific versions of an object.
- D. To replicate objects to another AWS Region for disaster recovery.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** S3 Lifecycle Policies are an automation tool for cost management and data retention. Their entire purpose is to define time-based rules that automatically move data to cheaper storage tiers as it ages or to permanently delete it when it's no longer needed.

</details>

---

**4. A company stores log files in a versioned S3 bucket. To save costs, they want to automatically delete old, noncurrent versions of the log files 90 days after they become noncurrent. What should they configure?**

- A. An S3 bucket policy.
- B. An S3 Lifecycle Policy rule to expire noncurrent versions.
- C. An S3 Event Notification that triggers a Lambda function to delete old versions.
- D. A scheduled job on an EC2 instance.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a classic and critical use case for S3 Lifecycle Policies on versioned buckets. You can create a specific rule that targets only noncurrent versions and sets an expiration action (permanent deletion) after a specified number of days. This is the native, most efficient way to prevent storage costs from growing indefinitely due to old versions.

</details>

---

**5. Once S3 Versioning is enabled for a bucket, what state can it be put into?**

- A. It cannot be changed in any way.
- B. It can be disabled, which deletes all old versions.
- C. It can be suspended, which stops creating new versions but preserves existing ones.
- D. It can only be disabled by contacting AWS Support.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** You can never truly "disable" versioning once it has been enabled. You can, however, suspend it. When suspended, the bucket stops creating unique version IDs for new objects (overwrites will actually replace the object), but all the versions that were created while versioning was active are preserved.

</details>

---

**6. A lifecycle policy is configured to transition objects from S3 Standard to S3 Glacier Flexible Retrieval after 30 days. An object is 45 days old. What will happen to it?**

- A. The object will be moved to S3 Glacier Flexible Retrieval.
- B. The object will be deleted.
- C. Nothing will happen, as the transition period has passed.
- D. The object will be moved to S3 Standard-IA first.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** S3 Lifecycle policies evaluate objects based on their creation date. Since the 45-day-old object is older than the 30-day threshold, it is eligible for the transition action. AWS will queue the object to be moved to S3 Glacier Flexible Retrieval.

</details>

---

**7. How do you access a specific, noncurrent (older) version of an object in a versioned S3 bucket?**

- A. It's not possible; only the current version is accessible.
- B. By sending a standard GET request for the object key.
- C. By specifying the object's unique Version ID in the GET request.
- D. By first promoting the older version to be the current version.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A standard GET request without a version ID will always retrieve the current version. To access any other version, you must include its specific version ID as part of the API call or request.

</details>

---

**8. Which S3 feature provides protection against accidental data deletion and overwrites?**

- A. S3 Lifecycle Policies
- B. S3 Cross-Region Replication
- C. S3 Versioning
- D. S3 Bucket Policies

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** S3 Versioning is the feature specifically designed to act as a safety net against these common user errors. By preserving a full history of all writes and deletes for an object, it allows for easy rollback and recovery.

</details>

---

**9. A bucket has versioning enabled. A lifecycle policy is configured to transition noncurrent versions to S3 Glacier after 60 days and then permanently delete them after 365 days. An object has 3 noncurrent versions that are 100 days old. What is their status?**

- A. They have been deleted.
- B. They are stored in S3 Standard.
- C. They have been transitioned to S3 Glacier.
- D. They are marked for deletion but not yet deleted.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The noncurrent versions are 100 days old. This is past the 60-day transition threshold but before the 365-day expiration threshold. Therefore, the lifecycle policy would have already moved these noncurrent versions to S3 Glacier.

</details>

---

**10. What is a "delete marker" in the context of S3 Versioning?**

- A. A flag that schedules an object for deletion by a lifecycle policy.
- B. An entry in CloudTrail that logs the deletion of an object.
- C. A special object with a version ID that is placed on top of an object's version stack when it is deleted, effectively hiding it.
- D. An object that has been moved to the S3 Glacier storage class.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The delete marker is the mechanism S3 uses to handle deletions in a versioned bucket. It's a zero-byte placeholder that becomes the "current" version of the object, making it appear as if it has been deleted while preserving all the previous data versions underneath it.

</details>

---

**11. Why is it important to configure a lifecycle policy to expire noncurrent versions on a bucket with versioning enabled?**

- A. To improve the read performance for the current version.
- B. To prevent incurring storage costs for a potentially unlimited number of old object versions.
- C. To automatically promote the latest noncurrent version if the current one is deleted.
- D. To comply with AWS security best practices for all S3 buckets.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is a critical cost management practice. Without a lifecycle rule to clean up old versions, every time a file is overwritten, a new full copy is stored. Over time, this can lead to massive and often unexpected storage costs. Expiring noncurrent versions is essential for managing these costs.

</details>

---

**12. A lifecycle rule is set to expire objects 30 days after creation. Versioning is NOT enabled on the bucket. What happens to a 35-day-old object?**

- A. It is moved to the recycle bin for 30 days.
- B. Nothing, because versioning is not enabled.
- C. The object is permanently deleted.
- D. The object is transitioned to S3 Glacier Deep Archive.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Lifecycle policies work on non-versioned buckets as well. In this case, the rule is straightforward. Since the object is older than the 30-day expiration threshold, S3 will permanently delete it.

</details>

---

**13. Can a single S3 Lifecycle Policy contain multiple rules?**

- A. No, only one rule is allowed per policy.
- B. Yes, a policy can contain multiple rules, for example, one for transitioning objects and another for expiring them.
- C. Yes, but all rules must apply to the entire bucket.
- D. No, you must create a separate policy for each rule.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Yes, a single lifecycle configuration XML file attached to a bucket can contain up to 1,000 rules. You can create different rules that apply to different prefixes (folders) or tags within the bucket, allowing for very granular and flexible automation.

</details>

---

**14. What is a benefit of using MFA Delete with S3 Versioning?**

- A. It requires users to enter an MFA code before they can upload a new version of an object.
- B. It provides an additional layer of security, requiring an MFA code to permanently delete an object version or change the bucket's versioning state.
- C. It automatically deletes old versions when an MFA device is lost.
- D. It encrypts all object versions using a key derived from an MFA token.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** MFA Delete is a security feature you can enable on a versioned bucket. Once enabled, any request to permanently delete a specific object version or to change the versioning state of the bucket (e.g., suspend it) must include a valid, temporary MFA code in the request. This provides strong protection against accidental or malicious deletions.

</details>

---

**15. To recover an object that was overwritten in a versioned bucket, what should you do?**

- A. Find the previous version of the object and copy it back on top of the current version.
- B. Find the previous version of the object and delete the current version.
- C. Delete the delete marker.
- D. Promote the previous version using the AWS CLI.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** To "roll back" to a previous version, the simplest method is to delete the current version. Since versioning is on, this will just add a delete marker on top of the current version, making the one you wanted to restore the new current version. Alternatively, you could copy the desired older version and overwrite the current one (A), but deleting the current version is often simpler. Answer C only applies if the object was deleted, not overwritten.

</details>

---

**16. A company needs to store data for 1 year and then delete it. For the first 60 days, it will be accessed frequently. After that, it will be accessed less often. What is the MOST cost-effective lifecycle policy?**

- A. Transition to S3 Standard-IA after 60 days, then expire after 365 days.
- B. Transition to S3 Glacier Deep Archive after 60 days, then expire after 365 days.
- C. Expire after 60 days.
- D. Transition to S3 Standard-IA after 30 days, then expire after 365 days.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** This policy correctly matches the data's access pattern. It keeps the data in the high-performance S3 Standard class during its frequent access period (the first 60 days). It then moves it to the cheaper S3 Standard-IA for the remainder of its life before finally expiring it at the 365-day mark.

</details>

---

**17. Can you apply a lifecycle policy to only a subset of objects in a bucket?**

- A. No, a lifecycle policy always applies to all objects in the bucket.
- B. Yes, by filtering the rule based on object tags, object prefixes, or a combination of both.
- C. Yes, but only by filtering based on object size.
- D. No, you must move the objects to a different bucket to apply a different policy.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Lifecycle rules are very flexible. You can scope a rule to apply only to objects that have a specific tag (e.g., status:archive) or that are under a specific prefix (e.g., logs/2024/). This allows for sophisticated data management within a single bucket.

</details>

---

**18. What is the default state of versioning on a newly created S3 bucket?**

- A. Enabled
- B. Suspended
- C. Not enabled
- D. Enabled with MFA Delete

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Versioning is not enabled by default. It is an optional feature that you must explicitly enable on the bucket's properties.

</details>

---

**19. A lifecycle rule is set to expire noncurrent versions after 7 days. On Day 1, `file.txt` (v1) is uploaded. On Day 5, `file.txt` (v2) is uploaded. On Day 10, what is the status of v1?**

- A. It is the current version.
- B. It has been noncurrent for 5 days and is still retained.
- C. It has been noncurrent for 9 days and is deleted.
- D. It has been noncurrent for 5 days and is deleted.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Version v1 became noncurrent on Day 5 when v2 was uploaded. On Day 10, it has been noncurrent for 10 - 5 = 5 days. Since this is less than the 7-day expiration threshold, it is still retained in the bucket as a noncurrent version.

</details>

---

**20. A user performs a `DELETE` action on an object in a bucket where versioning is suspended. What is the result?**

- A. A delete marker is created.
- B. The object is permanently deleted.
- C. The object's version ID is changed.
- D. The `DELETE` action fails.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When versioning is suspended, the bucket behaves like a non-versioned bucket for new actions. A DELETE operation will permanently remove the object, and an overwrite (PUT) will permanently replace the object. No delete marker is created.

</details>

---

**21. Can you configure a lifecycle policy to transition objects directly from S3 Standard to S3 Glacier Deep Archive?**

- A. Yes, this is a valid transition path.
- B. No, objects must first be transitioned to S3 Glacier Flexible Retrieval.
- C. No, objects must first be transitioned to S3 Standard-IA.
- D. No, transitions to Glacier Deep Archive are not allowed by lifecycle policies.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Yes, you can create a lifecycle rule that transitions objects directly from S3 Standard to any other storage class, including S3 Glacier Deep Archive, after the minimum required storage duration (typically 30 days for this kind of transition). You do not need to step through intermediate tiers.

</details>

---

**22. Which of the following is the best use case for S3 Versioning?**

- A. Reducing S3 storage costs.
- B. Improving the performance of S3 GET requests.
- C. Protecting against unintended user deletions and maintaining a history of object modifications.
- D. Enforcing encryption on all new objects.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The core purpose of versioning is data protection and history tracking. It provides a robust safety net, allowing for easy recovery from common human errors like accidental deletions or overwrites. While it can be used for retention, its primary benefit is recovery.

</details>
