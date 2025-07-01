# Step 01: IAM – Users, Groups, Policies

## 🎯 Objective
- [x] Master: **IAM – Users, Groups, Policies**

## 📘 Notes
> Knowledge summary, documentation links, videos...
Here's a concise summary of the **Identity and Access Management (IAM)** report you provided:

---

## **IAM: Concise Summary**

### **Executive Summary**

Identity and Access Management (IAM) is a core pillar of cloud security. It ensures only authorized users and systems have access to resources, replacing traditional network perimeters with identity-based control. IAM is essential for enforcing the **Principle of Least Privilege (PoLP)**, maintaining compliance, and mitigating security risks.

---

## **1. What is IAM and Why It Matters**

* **Definition**: IAM governs digital identities, authenticates users, authorizes actions, and monitors access.
* **Cloud Importance**:

  * Prevents unauthorized access and breaches.
  * Supports compliance (e.g., GDPR, HIPAA).
  * Enhances operational efficiency via automation and SSO.
  * Enables granular access control and centralized management.

---

## **2. IAM Core Components**

### **Users**

* **Human**: Employees, contractors.
* **Non-Human (NHIs)**: Services, scripts. Require automated management and short-lived credentials.
* **Root User vs. IAM User (AWS)**:

  * Root has full, unrestricted access (used only for critical tasks).
  * IAM users are recommended for daily use with policy-defined permissions.

### **Groups**

* Grouping users simplifies access management by assigning policies at scale.
* Benefits: scalability, consistency, and easier onboarding/offboarding.

### **Policies**

* Define what actions identities can perform on resources.
* Written in JSON with key elements:

  * **Effect**, **Action**, **Resource**, **Condition**, **NotAction**, **NotResource**, **SID**.
* **Types**:

  * **AWS Managed**: Created by AWS, not customizable.
  * **Customer Managed**: User-created, customizable, reusable.
  * **Inline**: Directly attached to a user/group/role, deleted with the entity.

---

## **3. How IAM Works Together**

* **Authentication**: Confirms identity (e.g., password, MFA).
* **Authorization**: Policies are evaluated in order: SCPs → Resource → Identity-based → Permissions Boundaries → Session Policies.
* **Policy Evaluation Rules**:

  * All access is denied by default.
  * Explicit denies override any allows.
* **Cloud Hierarchies**: Inherited permissions in platforms like Google Cloud can introduce risks if not managed carefully.

---

## **4. Best Practices**

### **4.1. Principle of Least Privilege (PoLP)**

* Grant only the minimum necessary access.
* Methods:

  * **RBAC**, **JIT Access**, **Automated Deprovisioning**, **Access Reviews**.
* Benefits: Reduces risk of breaches, insider threats, and compliance failures.

### **4.2. Other Key Practices**

* **MFA** and **SSO** for secure, simplified login.
* **Regular Auditing** and **Monitoring** to detect threats.
* **Automate IAM Tasks** to reduce human error.
* **RBAC/ABAC** for scalable, dynamic access control.
* **Strong Password Policies**.
* **Zero Trust Model**: Always authenticate and authorize.

---

## **5. IAM Risks and Misconfigurations**

### **Common Risks**

* Excessive permissions (privilege creep).
* Weak password management.
* Poor provisioning/deprovisioning.
* Lack of visibility.
* Privileged account abuse.
* Identity misconfigurations.
* Insecure external sharing.

### **Examples**

* Overly broad policies (e.g., `s3:*`).
* Hardcoded credentials in code.
* Unrevoked access for former employees.
* Misconfigured trust policies.
* Role misuse in EC2 deployments.

**Mitigation**: Use automation, audits, MFA, policy templates, and security tools.

---

## **6. Conclusion & Recommendations**

IAM is essential to secure cloud operations. Key recommendations:

* Enforce PoLP with RBAC and ABAC.
* Automate identity lifecycle management.
* Mandate MFA and use SSO.
* Regularly audit permissions and logs.
* Secure NHIs with ephemeral access and rotation.
* Adopt a Zero Trust model.
* Invest in IAM tools and staff training.

## 🧪 Practice