# Step 16: AMI & EBS Snapshots

## 🎯 Objective

- [x] Master: **AMI & EBS Snapshots**

## 📘 Notes

### **Deep Dive: AMI & EBS Snapshots**

Understanding the relationship between Amazon Machine Images (AMIs) and Elastic Block Store (EBS) Snapshots is fundamental to managing EC2 instances effectively. An AMI is the template for your instance's software, while a snapshot is the backup of its data. They are intrinsically linked.

---

### **1. EBS Snapshots** 📸

An EBS Snapshot is a point-in-time backup of an EBS volume. It is the primary way to back up your instance data, protect it against failure, and replicate it across different regions or accounts.

- **How They Work (Incremental Backups):**
  - The first snapshot you take of a volume is a **full copy** of all the data on the volume at that moment.
  - Subsequent snapshots of the same volume are **incremental**. This means only the blocks on the volume that have changed since your last snapshot are saved.
  - This makes subsequent snapshots very fast and cost-effective, as you are not charged for the unchanged data. When you delete a snapshot, only the data unique to that snapshot is removed. The integrity of the full backup chain is always maintained by AWS.
- **Key Features:**
  - **Storage:** Snapshots are stored durably and cost-effectively in Amazon S3, though you don't see them in your S3 buckets; they are managed through the EC2 console.
  - **Regionality:** Like AMIs, snapshots are constrained to the region where they were created. To use a snapshot in a different region, you must **copy** it to that target region.
  - **Restoration:** You can create a new EBS volume from a snapshot at any time. You can also change the volume type, size, or Availability Zone when creating a volume from a snapshot.
  - **Sharing:** You can share snapshots with other specific AWS accounts or make them public. This is a common way to share data with partners or other business units.

---

### **2. Amazon Machine Images (AMIs)** 💿

We've learned that an AMI is a template for an EC2 instance. For EBS-backed instances, an AMI is, at its core, a collection of metadata and a set of pointers to **EBS snapshots**.

- **The Relationship:**
  - When you create an AMI from a running EC2 instance, AWS initiates the following process:
    1. It creates snapshots of the instance's root volume and any other attached EBS volumes.
    2. It creates an AMI object that essentially acts as a wrapper. This AMI object contains metadata (like the architecture, kernel, etc.) and pointers to the EBS snapshots that represent the state of the volumes at that time.
  - When you launch an instance from this AMI, AWS uses those underlying snapshots as the blueprint to create new, identical EBS volumes for your new instance.
- **Creating a Custom AMI:**
  - This is the standard process for creating a "golden image."
  1. Launch an instance from a trusted base AMI (like Amazon Linux 2).
  2. Connect to the instance and install/configure your applications, apply patches, and set up monitoring agents.
  3. From the EC2 console, select the configured instance and choose "Create Image (AMI)." AWS will handle the process of creating the necessary snapshots and registering the new AMI.
  4. Once the AMI is in an "available" state, you can use it to launch new, pre-configured instances.
- **Deregistering an AMI:** When you no longer need a custom AMI, you can **deregister** it. This removes the AMI itself, but the underlying **EBS snapshots are not automatically deleted**. You must manually find and delete the associated snapshots if you want to stop paying for their storage.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A solutions architect takes an initial snapshot of a 100 GB EBS volume. The next day, 5 GB of data on the volume changes, and a second snapshot is taken. How much storage will the second snapshot consume?**

- A. 105 GB
- B. 100 GB
- C. 5 GB
- D. 0 GB, as it references the first snapshot.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** EBS snapshots are incremental. The first snapshot is a full copy of the data (100 GB). Subsequent snapshots only store the blocks that have changed since the last snapshot. In this case, only 5 GB of data changed, so the second snapshot will only consume 5 GB of storage space in S3.

</details>

---

**2. What is an Amazon Machine Image (AMI) for an EBS-backed instance fundamentally composed of?**

- A. A copy of the instance's RAM state.
- B. Pointers to one or more EBS snapshots, plus instance metadata.
- C. A zip file containing the operating system and applications.
- D. A direct, block-for-block copy of the original EBS volume.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An AMI for an EBS-backed instance is essentially a blueprint. It doesn't contain the full data itself but rather acts as a wrapper around the EBS snapshots of the instance's volumes. It includes metadata (like kernel ID, architecture) and pointers to these snapshots, which are then used to create new volumes when an instance is launched.

</details>

---

**3. An administrator needs to back up a critical EBS volume. What is the standard AWS procedure for this?**

- A. Use the AWS CLI to copy the volume's data to an S3 bucket.
- B. Detach the volume and attach it to a special "backup" EC2 instance.
- C. Create an EBS Snapshot of the volume.
- D. Configure AWS Backup to use an instance store as the target.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Creating an EBS Snapshot is the native, recommended, and most common method for backing up an EBS volume. It creates a durable, point-in-time copy of the volume that is stored in Amazon S3.

</details>

---

**4. You have created a custom AMI in the `us-west-2` region. You now need to launch an identical server in the `ap-southeast-1` region. What must you do first?**

- A. Launch an instance in `us-west-2` and then migrate the running instance to `ap-southeast-1`.
- B. Nothing, AMIs are global resources and can be used in any region.
- C. Use the "Copy AMI" feature from the EC2 console to copy the AMI from `us-west-2` to `ap-southeast-1`.
- D. Share the AMI with the `ap-southeast-1` region.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** AMIs are a regional resource. To use an AMI in a different region, you must first create a copy of it in the destination region. The "Copy AMI" action handles the underlying process of copying the associated snapshots and registering a new AMI in the target region.

</details>

---

**5. An administrator deregisters a custom AMI that is no longer needed. What happens to the EBS snapshot associated with that AMI?**

- A. The snapshot is automatically deleted along with the AMI.
- B. The snapshot is retained, and you must manually delete it to avoid storage costs.
- C. The snapshot is moved to Amazon S3 Glacier Deep Archive.
- D. The snapshot becomes public and is moved to the community snapshots.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Deregistering an AMI only removes the AMI pointer object. It does not delete the underlying EBS snapshots. This is a safety feature to prevent accidental data loss, but it means you are responsible for manually deleting the snapshot to stop incurring storage charges.

</details>

---

**6. A company wants to share a custom AMI containing proprietary software with a partner company that has a different AWS account. What is the most secure way to do this?**

- A. Make the AMI public so the partner can find it.
- B. Modify the AMI's launch permissions and explicitly share it with the partner's AWS account ID.
- C. Copy the AMI's underlying snapshot and share the snapshot with the partner's account.
- D. Launch an instance from the AMI and give the partner SSH access to it.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** AMIs can be shared privately and securely. By modifying the AMI's launch permissions, you can add the specific 12-digit AWS account ID of the partner. This will make the AMI visible and usable only to that account, preventing public exposure. While sharing the snapshot (C) is possible, sharing the AMI directly is the more common workflow.

</details>

---

**7. When you restore an EBS volume from a snapshot, which of the following attributes can you change?**

- A. The Availability Zone, volume type, and size.
- B. Only the size of the volume.
- C. Only the Availability Zone.
- D. None, the new volume must be identical to the original.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** Restoring from a snapshot is very flexible. It allows you to create a new volume in any Availability Zone within the same region. You can also change the size (it must be equal to or larger than the snapshot size) and change the volume type (e.g., from gp2 to io1).

</details>

---

**8. For an application that requires a pre-warmed, fully configured environment to launch quickly and consistently, what is the best practice?**

- A. Use a stock AWS AMI and a long User Data script to configure everything on boot.
- B. Create a custom "golden" AMI that has all applications and configurations pre-installed.
- C. Use Docker containers exclusively.
- D. Use EBS snapshots to restore data after the instance boots.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Creating a custom "golden" AMI is the standard solution for this requirement. By pre-installing and pre-configuring everything, you significantly reduce the instance's boot time and eliminate potential failures that could occur in a long bootstrapping script. This leads to faster and more reliable deployments, especially in Auto Scaling groups.

</details>

---

**9. What is the primary cost-saving benefit of EBS snapshots being incremental?**

- A. It reduces the time it takes to create a snapshot.
- B. You are only billed for the storage of the changed blocks since the last snapshot.
- C. It reduces the latency when restoring a volume.
- D. It allows you to store snapshots in a cheaper AWS region.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The incremental nature directly impacts cost. After the first full backup, you are not re-billed for storing the full data set with each subsequent snapshot. You only pay for the storage of the new, changed blocks, which makes frequent backups highly cost-effective.

</details>

---

**10. When you copy an encrypted EBS snapshot to another region, what is required?**

- A. The data must be decrypted before being copied and re-encrypted in the destination region.
- B. You must use a KMS key in the destination region to re-encrypt the snapshot copy.
- C. You must first share the snapshot publicly.
- D. The destination region must be in the same country as the source region.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When you copy an encrypted snapshot, AWS handles the decryption and re-encryption process transparently. However, since KMS keys are regional, you must specify a KMS key in the destination region that AWS will use to encrypt the new copy of the snapshot. You cannot use a KMS key from the source region.

</details>

---

**11. A user creates an AMI from an instance that has an encrypted EBS root volume. What is the encryption status of the new AMI's underlying snapshot?**

- A. The snapshot is unencrypted by default.
- B. The snapshot is automatically encrypted using the same KMS key as the original volume.
- C. The user is prompted to choose a new encryption key during AMI creation.
- D. The AMI cannot be created from an instance with an encrypted root volume.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Encryption status is inherited. When you create a snapshot (either manually or via AMI creation) from an encrypted EBS volume, the resulting snapshot is always encrypted. By default, it uses the same KMS key that was used to encrypt the source volume.

</details>

---

**12. To automate the creation of EBS snapshots for backup and retention purposes, which service should be used?**

- A. AWS CloudTrail
- B. Amazon CloudWatch Events (or EventBridge)
- C. Amazon Data Lifecycle Manager (DLM)
- D. AWS Config

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Amazon Data Lifecycle Manager (DLM) is the dedicated service for automating the creation, retention, and deletion of EBS snapshots. You can create policies that run on a schedule (e.g., every 24 hours), target volumes based on tags, and define retention rules (e.g., keep daily snapshots for 7 days, weekly for 4 weeks).

</details>

---

**13. A snapshot of a 200 GB volume is taken. Later, the volume is resized to 300 GB, and another snapshot is taken. What is the size of the second snapshot?**

- A. 300 GB
- B. 100 GB plus any changed blocks from the original 200 GB.
- C. 200 GB
- D. The snapshot will fail because the volume size changed.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The second snapshot will be incremental. It will store all the new blocks from the resized portion of the volume (100 GB) plus any blocks that changed within the original 200 GB portion since the first snapshot was taken.

</details>

---

**14. An AMI has been shared with your AWS account. Where would you find and use this shared AMI?**

- A. It will automatically appear in the "My AMIs" section of the EC2 console.
- B. In the EC2 launch wizard, under the "AMIs shared with me" or "Private images" filter.
- C. It will be delivered to your S3 buckets.
- D. You must import it using the AWS CLI.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When another account shares an AMI with you, it doesn't appear in your primary AMI list. You need to go to the launch instance wizard and specifically filter for "Private images" or look in the "Shared with me" section to see and select the AMI that was shared.

</details>

---

**15. If you delete a snapshot that is part of an incremental backup chain (e.g., you delete Snapshot B from a chain of A -> B -> C), what happens?**

- A. Snapshot C becomes corrupted because its dependency is gone.
- B. All snapshots in the chain are deleted automatically.
- C. AWS preserves the full backup chain by moving the data blocks that were unique to Snapshot B into Snapshot C.
- D. You cannot delete a snapshot if it is referenced by another snapshot.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The snapshot deletion process is designed to be safe. When you delete a snapshot, AWS analyzes which data blocks are still needed by other snapshots in the chain. Any data that was referenced by the deleted snapshot but is still needed by a later snapshot is "moved" to that later snapshot. You are only ever removing data that is no longer referenced by any snapshot, and you are only billed for the total unique blocks stored across all your snapshots.

</details>

---

**16. The process of creating a standardized, patched, and pre-configured AMI for use within a company is often called creating a:**

- A. "Golden AMI"
- B. "Root Snapshot"
- C. "Public Image"
- D. "Community Container"

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** "Golden AMI" or "Golden Image" is the industry-standard term for a custom AMI that has been hardened, patched, and configured to meet an organization's specific baseline requirements.

</details>

---

**17. Can you create an EBS volume from a snapshot in a different AWS Region?**

- A. Yes, directly from the snapshot in the source region.
- B. No, this is not possible under any circumstances.
- C. No, you must first copy the snapshot to the target region and then create a volume from the copy.
- D. Yes, but only if the regions are in the same geographic area (e.g., both in North America).

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Snapshots and EBS volumes are regional resources. To create a volume in Region B from a snapshot that exists in Region A, you must first perform a "Copy Snapshot" operation to create a duplicate of the snapshot in Region B. Then you can use that local copy to create the new volume.

</details>

---

**18. What happens to the underlying EBS snapshots when an EC2 instance is terminated?**

- A. The snapshots are automatically deleted.
- B. The snapshots are automatically deregistered.
- C. Nothing happens to the snapshots, as they are independent resources.
- D. The snapshots are detached from the AMI.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Snapshots are persistent backup objects stored in S3. The lifecycle of an EC2 instance has no direct impact on any snapshots that may have been created from its volumes. The EBS volumes attached to the instance might be deleted on termination (depending on their "Delete on Termination" flag), but the snapshots are completely separate and will remain.

</details>

---

**19. When creating an AMI, it is a best practice to do what to the instance first to ensure data consistency?**

- A. Detach all EBS volumes.
- B. Stop the instance.
- C. Increase the instance size.
- D. Disconnect it from the network.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** While AWS can create a snapshot from a running instance, it is a best practice to stop the instance before creating the AMI. This ensures that any data held in memory is flushed to the disk and that there are no pending I/O operations, resulting in a perfectly consistent point-in-time snapshot.

</details>

---

**20. You have an unencrypted EBS volume. What is the easiest way to create an encrypted version of this volume?**

- A. Create a snapshot of the volume. Use the "Copy Snapshot" feature, enabling the "Encrypt this snapshot" option. Then, create a new volume from the encrypted snapshot.
- B. Enable encryption on the running unencrypted volume.
- C. Detach the volume and use the "Encrypt Volume" command in the AWS CLI.
- D. This is not possible; a volume's encryption state cannot be changed.

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** You cannot directly encrypt an existing unencrypted volume (B is incorrect). The standard procedure is to use snapshots. You take a snapshot (which will be unencrypted), then make an encrypted copy of that snapshot. The copy operation is where you can apply encryption. Finally, you create a new, encrypted volume from the encrypted snapshot copy.

</details>

---

**21. Where are EBS snapshots stored?**

- A. On the same physical media as the source EBS volume.
- B. In an EC2 Instance Store for fast retrieval.
- C. In Amazon S3, managed by AWS.
- D. In Amazon EFS for shared access.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** EBS snapshots are stored in Amazon S3 for high durability and low cost. AWS manages the storage and abstraction, so you don't see the snapshots as objects in your S3 buckets, but S3 is the underlying storage service.

</details>

---

**22. If you share an encrypted AMI with another account, what must you also share with them?**

- A. The password to the instance's operating system.
- B. An IAM user's access keys.
- C. Permissions to use the KMS key that was used to encrypt the AMI's snapshots.
- D. Nothing, the permissions are transferred automatically.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** For another account to be able to launch an instance from your encrypted AMI, they need two sets of permissions: 1) permission to use the AMI itself, and 2) permission to use the KMS key that will be needed to decrypt the snapshots and create the new encrypted volumes. You must modify the KMS key's policy to grant kms:Decrypt, kms:ReEncrypt\*, and other necessary permissions to the other account.

</details>
