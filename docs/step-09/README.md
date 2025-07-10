# Step 09: KMS – Keys, CMKs, Encryption Workflow

## 🎯 Objective
- [x] Master: **KMS – Keys, CMKs, Encryption Workflow**

## 📘 Notes

### **Deep Dive: KMS – Keys, CMKs, & Encryption Workflow**

AWS Key Management Service (KMS) is a managed service that makes it easy for you to create and control the encryption keys used to encrypt your data. It provides a highly available and durable key management system that integrates deeply with other AWS services. At its core, KMS is about controlling who can use your keys to encrypt and decrypt data.

### **1. KMS Keys (Customer Master Keys - CMKs)**

The primary resource in KMS is the **Customer Master Key (CMK)**. A CMK is a logical representation of a key. It's not the key itself but a container for the key material that includes metadata like the key ID, creation date, and a key policy. The term "CMK" is still widely used, but newer AWS documentation often refers to them simply as **KMS keys**.

There are three main types of KMS keys you must know:

- **AWS Managed Keys:**
    - Created, managed, and used on your behalf by an AWS service (e.g., S3, EBS, RDS) that integrates with KMS.
    - You can see them in your account (they have aliases like `aws/s3` or `aws/ebs`), but you **cannot manage them directly**. You can't change their key policies or schedule them for deletion.
    - Rotation is automatic (every 3 years, though this can vary by service).
    - **Cost:** Free.
- **Customer Managed Keys:**
    - You create, own, and manage these keys directly within your AWS account. You have **full control** over them.
    - **Control:** You define their key policies, IAM policies, can enable/disable them, schedule them for deletion, and configure automatic rotation.
    - **Rotation:** You can enable automatic key rotation, which occurs **once per year**. KMS saves the old key material to decrypt data previously encrypted with it.
    - **Custom Key Material:** You can import your own key material (BYOK - Bring Your Own Key) or use a custom key store backed by your own AWS CloudHSM cluster.
    - **Cost:** There is a monthly fee for each Customer Managed Key, plus charges per API request.
- **AWS Owned Keys:**
    - These are keys that are owned and managed by an AWS service for use in multiple AWS accounts. They are **not in your AWS account**. You have no visibility or control over them. For example, AWS Shield uses AWS Owned Keys to encrypt certain data. For the exam, you just need to know they exist as a separate category you don't manage.

**Key Policy:** This is the most important control mechanism. A key policy is a **resource-based policy** attached directly to a Customer Managed Key. It is the primary way to control access to the key. It specifies who (principals) can perform which actions (e.g., `kms:Encrypt`, `kms:Decrypt`, `kms:ReEncrypt`) on that specific key. **You must give the AWS account root user access in the key policy, otherwise you risk locking yourself out.**

### **2. The Encryption Workflow: Envelope Encryption** ✉️

KMS does not typically encrypt large amounts of data directly with a CMK because it's slow and inefficient. Instead, it uses a highly secure and performant method called **Envelope Encryption**.

Here's the workflow:

**To Encrypt Data:**

1. Your application asks KMS to generate a unique data key. This request is sent to the `kms:GenerateDataKey` API, specifying which CMK to use.
2. KMS uses the specified CMK to generate a new data key. It returns two things to your application:
    - **Plaintext Data Key:** The actual key used to encrypt your data.
    - **Encrypted Data Key:** The same data key, but encrypted under the CMK you specified.
3. Your application uses the **Plaintext Data Key** in memory to encrypt your large data object (e.g., a file or a database entry). This is fast because it happens locally within your application.
4. After encryption, your application **discards the Plaintext Data Key** from memory immediately. This is a critical security step.
5. You then store the **encrypted data** alongside the **Encrypted Data Key**. This combination is the "envelope."

**To Decrypt Data:**

1. Your application retrieves the encrypted data and the associated Encrypted Data Key.
2. It sends **only the Encrypted Data Key** to KMS via the `kms:Decrypt` API call.
3. KMS uses the original CMK (which it knows based on the encrypted key's metadata) to decrypt the Encrypted Data Key, revealing the original Plaintext Data Key.
4. KMS returns the **Plaintext Data Key** to your application.
5. Your application uses this Plaintext Data Key in memory to decrypt the large data object.
6. Once decryption is complete, the application discards the Plaintext Data Key from memory.

**Why use Envelope Encryption?**

- **Performance:** Symmetric key encryption (using the data key) is much faster than asymmetric encryption (which KMS uses internally).
- **Security:** The powerful master key (CMK) never leaves KMS. You are only ever working with and storing small, encrypted data keys alongside your data.
- **Cost-Effective:** You reduce the number of API calls to KMS, as you only call it once to get/decrypt the data key, not for every byte of data.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A developer wants to encrypt a 500 GB data file before uploading it to Amazon S3. What is the most secure and efficient method using AWS KMS?**

- A. Send the entire 500 GB file to the `kms:Encrypt` API to have KMS encrypt it directly with a Customer Master Key (CMK).
- B. Use the `kms:GenerateDataKey` API to get a plaintext data key and an encrypted data key. Use the plaintext key to encrypt the file locally, and then store the encrypted file and the encrypted data key in S3.
- C. Split the 500 GB file into 1 MB chunks and send each chunk to the `kms:Encrypt` API.
- D. Use the CMK's ARN as the encryption key directly within the application's encryption library.
<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This describes the process of envelope encryption. It is the standard and best-practice method for encrypting large objects. The CMK never leaves KMS; it is only used to encrypt and decrypt the much smaller data key. Encrypting large objects directly with the kms:Encrypt API (A, C) is not supported (there's a 4 KB limit) and would be inefficient. A CMK ARN (D) is an identifier, not the key material itself.

</details>
    

---

**2. A company has a strict compliance requirement that their encryption keys must be generated and stored in a FIPS 140-2 Level 3 validated hardware security module (HSM). They also want to manage the key's lifecycle and policies within AWS. Which KMS key solution should they use?**

- A. An AWS Managed Key.
- B. A Customer Managed Key with imported key material.
- C. A Customer Managed Key using an AWS CloudHSM key store.
- D. An AWS Owned Key.
<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** An AWS CloudHSM cluster provides dedicated, FIPS 140-2 Level 3 validated HSMs that you control. By creating a KMS custom key store linked to your CloudHSM cluster, you can create Customer Managed Keys where the key material is generated and exclusively stored in your CloudHSM. This meets the stringent compliance requirement while still allowing you to manage the key through the standard KMS interface.

</details>
    

---

**3. What is the primary purpose of a KMS Key Policy?**

- A. To control which IAM users can log in to the AWS console.
- B. To define the maximum permissions for a member account in AWS Organizations.
- C. To act as the central, resource-based policy that defines who can use and manage a specific KMS key.
- D. To configure the automatic rotation schedule for a key.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The key policy is the primary access control mechanism for a KMS key. It is a resource-based policy attached directly to the key. It must specify which principals (users, roles, accounts) are allowed to perform actions like kms:Encrypt, kms:Decrypt, or administrative actions like kms:ScheduleKeyDeletion.
</details>
    

---

**4. When using envelope encryption, what two items are returned to the application after a successful `kms:GenerateDataKey` API call?**

- A. The plaintext CMK and the encrypted data key.
- B. The encrypted data and the plaintext data key.
- C. The plaintext data key and the encrypted data key.
- D. The key policy and the plaintext data key.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The GenerateDataKey operation returns a data key in two forms: one in plaintext for you to use immediately for encryption, and one encrypted under the specified CMK for you to safely store alongside your data. The CMK itself (A) never leaves KMS.
</details>
    

---

**5. A company wants to use a single key to encrypt data for Amazon S3, Amazon EBS, and Amazon RDS within their account. They want to control the key's policy but want AWS to manage its lifecycle. Which type of KMS key should they use?**

- A. An AWS Managed Key like `aws/s3`.
- B. A Customer Managed Key.
- C. An AWS Owned Key.
- D. They must use a separate key for each service.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** A Customer Managed Key is the only type that allows you to define a custom key policy and use it across multiple supported AWS services. AWS Managed Keys (A) are tied to a specific service and you cannot edit their policies. AWS Owned Keys (C) are not visible or manageable by you. A single Customer Managed Key can be used as the encryption key for S3 objects, EBS volumes, and RDS databases simultaneously.
</details>
    

---

**6. An IAM policy for a user contains the statement `{"Effect": "Allow", "Action": "kms:Decrypt", "Resource": "*"}`. What does this policy grant the user?**

- A. The user can decrypt data encrypted by any KMS key in the account.
- B. The user can decrypt data, but only if the key policy of the specific KMS key also grants them permission.
- C. The policy is invalid because `Resource` cannot be  for KMS actions.
- D. The user can manage all KMS keys in the account.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Access to a KMS key is ultimately controlled by its key policy. An IAM policy alone is not sufficient. For the user to be able to decrypt, their IAM policy must grant kms:Decrypt, AND the resource-based key policy on the key itself must also grant permission to that user (or their account).
</details>
    

---

**7. When you enable automatic key rotation for a Customer Managed Key, what happens?**

- A. AWS decrypts all your data and re-encrypts it with a new key every year.
- B. AWS generates new cryptographic material for the key every year, and the key's ARN remains the same.
- C. A new KMS key with a new ARN is created, and the old key is deleted.
- D. The key material is rotated every 90 days.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** When automatic rotation is enabled, KMS creates new backing key material once per year. Crucially, the key's metadata, including its ARN, remains unchanged. KMS transparently retains all previous versions of the key material so it can still decrypt any data that was encrypted with an older version. The user experience is seamless.
</details>
    

---

**8. What is the main security advantage of envelope encryption?**

- A. It uses a different encryption algorithm than standard KMS encryption.
- B. It allows the powerful master key (CMK) to be stored alongside the encrypted data.
- C. It limits the exposure of the master key by using it only to encrypt and decrypt small data keys, while the master key itself never leaves AWS KMS.
- D. It eliminates the need for IAM policies for decryption.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The core security principle of envelope encryption is protecting the master key. By ensuring the CMK never leaves the secure confines of the KMS HSMs and is only used briefly to wrap/unwrap data keys, the overall security posture is significantly improved.
</details>
    

---

**9. A team created a Customer Managed Key but forgot to add their own IAM user ARN to the key policy as an administrator. Now, no one can access the key to edit its policy. What is the likely outcome?**

- A. The key is now unusable and must be deleted by contacting AWS Support.
- B. The IAM user with account root user access can edit the key policy.
- C. After 24 hours, the key policy automatically resets to a default policy.
- D. The key can be managed by any user with the `AdministratorAccess` IAM policy.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** This is why it is a critical best practice to always give permissions to the AWS account root user in the key policy (e.g., arn:aws:iam::111122223333:root). This ensures that if you make a mistake and lock out all other users, you have a "break-glass" option. The account root user has the ultimate authority to fix the key policy.
</details>
    

---

**10. Which of the following statements about AWS Managed Keys is TRUE?**

- A. You have full control over their key policies.
- B. You are charged a monthly fee for each AWS Managed Key.
- C. They are automatically rotated by AWS on a schedule that you cannot change.
- D. You can use a single AWS Managed Key for multiple services like S3 and EBS.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** AWS Managed Keys are managed entirely by AWS. You cannot edit their policies (A), and they are free of charge (B). They are also specific to a service (e.g., aws/s3 can only be used by S3), so you cannot use one across multiple services (D). Their key material is automatically rotated by AWS (typically every 3 years).
</details>
    

---

**11. During the decryption process using envelope encryption, what information is sent to the `kms:Decrypt` API?**

- A. The encrypted data.
- B. The plaintext data key.
- C. The encrypted data key.
- D. The ARN of the user requesting decryption.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** To decrypt a large object, the application only needs to send the small, encrypted data key to the kms:Decrypt API. KMS uses the master key to decrypt this data key and returns the plaintext data key to the application, which then performs the actual decryption of the large object locally.
</details>
    

---

**12. A company needs to prove to auditors that their encryption key material was generated by and is exclusively stored within their on-premises HSMs. Which KMS feature allows this?**

- A. Automatic key rotation.
- B. Using an AWS CloudHSM key store.
- C. Importing key material (BYOK - Bring Your Own Key).
- D. Using an AWS Managed Key.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The Bring Your Own Key (BYOK) feature allows you to generate key material in your own secure environment (like an on-premises HSM) and then securely import it into a Customer Managed Key in KMS. This gives you direct control and proof of origin for the key material itself. A CloudHSM key store (B) keeps the key in an AWS-hosted HSM, not an on-premises one.
</details>
    

---

**13. A user wants to grant another AWS account permission to use a Customer Managed Key. How can this be achieved?**

- A. By adding the other account's ARN to the `Resource` section of an IAM policy.
- B. By adding the other account's ARN to the `Principal` section of the KMS key policy.
- C. This is not possible; KMS keys cannot be shared across accounts.
- D. By creating a VPC peering connection between the two accounts.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The key policy, being a resource-based policy, is the correct place to define cross-account access. By adding the other AWS account's root ARN (e.g., arn:aws:iam::999988887777:root) as a Principal in the key policy, you authorize that account to use the key (provided its own IAM principals also have the necessary IAM permissions).
</details>
    

---

**14. What is the maximum size of data that can be encrypted directly by the `kms:Encrypt` API call?**

- A. 1 MB
- B. 1 GB
- C. 4 KB
- D. There is no limit.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The KMS Encrypt API operation has a 4 KB (4096 bytes) limit on the size of data it can encrypt directly. This is why envelope encryption is essential for larger objects - you encrypt the data with a data key locally, not with the KMS Encrypt API directly.
</details>

    

---

**15. If you schedule a Customer Managed Key for deletion, what is the minimum waiting period before it is permanently deleted?**

- A. 24 hours
- B. 7 days
- C. 30 days
- D. 90 days

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** KMS enforces a mandatory waiting period for key deletion to prevent accidental data loss. You can schedule deletion for a period ranging from a minimum of 7 days up to a maximum of 30 days. During this time, you can cancel the deletion. Once deleted, a key and its material are irrecoverable.
</details>
    

---

**16. An application running on an EC2 instance needs to encrypt data using a Customer Managed Key. Which API call should the application make first?**

- A. `kms:Encrypt`
- B. `kms:Decrypt`
- C. `kms:GenerateDataKey`
- D. `kms:ReEncrypt`

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The first step in the envelope encryption workflow is to get a data key to work with. The application calls kms:GenerateDataKey, specifying the ARN of the Customer Managed Key it wants to use. This returns the plaintext and encrypted versions of the data key, kicking off the rest of the encryption process.
</details>
    

---

**17. What does it mean if a KMS key is "disabled"?**

- A. The key is scheduled for deletion and will be removed soon.
- B. All attempts to use the key for cryptographic operations (encrypt/decrypt) will fail.
- C. The key can be used for decryption but not for encryption.
- D. The key's policy is temporarily invalid.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Disabling a key is a temporary and immediate way to prevent it from being used in any cryptographic operation. Any call to Encrypt, Decrypt, ReEncrypt, or GenerateDataKey using this key will fail. It's a useful way to quickly "turn off" access to all data protected by the key without deleting it. You can re-enable it at any time.
</details>
    

---

**18. Which of the following is a key difference between a Customer Managed Key and an AWS Managed Key?**

- A. Customer Managed Keys are free, while AWS Managed Keys have a monthly cost.
- B. You can enable yearly automatic rotation for a Customer Managed Key, but you cannot change the rotation schedule for an AWS Managed Key.
- C. Customer Managed Keys can only be used by one AWS service.
- D. AWS Managed Keys provide more granular control via their key policies.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** Control over the key's lifecycle is a major differentiator. With a Customer Managed Key, you have a checkbox to enable automatic yearly rotation. With an AWS Managed Key, AWS controls the rotation policy (typically every 3 years), and you have no ability to change it.
</details>
    

---

**19. An organization wants to ensure that a specific Customer Managed Key can only be used to encrypt and decrypt data originating from their VPC. How can this be enforced?**

- A. By creating a VPC peering connection to the KMS service.
- B. By using the `aws:SourceVpce` condition key in the KMS key policy.
- C. This can only be controlled at the application level.
- D. By launching the KMS key within the target VPC.

<details>
<summary>View Answer</summary>
<br>

**Answer: B**

**Explanation:** The aws:SourceVpce condition key is designed for this purpose. You can add a condition to your KMS key policy that denies actions like kms:Encrypt or kms:Decrypt unless the request comes through a specific VPC endpoint (Vpce). This ensures the key can only be used from within your network perimeter.
</details>
    

---

**20. After an application encrypts a large file using a plaintext data key obtained from KMS, what is the immediate next step it should perform according to best practices?**

- A. Store the plaintext data key alongside the encrypted file.
- B. Upload the plaintext data key to a secure S3 bucket.
- C. Discard the plaintext data key from memory.
- D. Use the plaintext data key to encrypt another file.

<details>
<summary>View Answer</summary>
<br>

**Answer: C**

**Explanation:** The plaintext data key is highly sensitive. Once it has been used to perform the encryption, it should be immediately and securely discarded from memory. The only version of the data key that should be stored persistently is the one that was encrypted by the CMK.
</details>
