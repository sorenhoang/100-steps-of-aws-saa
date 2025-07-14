# Step 18: Auto Scaling Groups

## 🎯 Objective

- [x] Master: **Auto Scaling Groups**

## 📘 Notes

### **Deep Dive: Auto Scaling Groups (ASG)**

An Auto Scaling Group (ASG) is a collection of EC2 instances managed as a logical grouping. Its primary purpose is to automatically adjust the number of instances in the group to meet the current level of demand. This ensures your application has the right amount of compute capacity at all times, providing both high availability and cost optimization.

---

### **1. Core Components of an Auto Scaling Group**

An ASG is defined by three main components:

- **Launch Template (or Launch Configuration):**
  - This is the blueprint for the EC2 instances that the ASG will launch. It specifies the AMI, instance type, key pair, security groups, IAM role, and any User Data scripts.
  - **Launch Templates** are the newer, recommended standard. They support versioning and can include a wider range of parameters (like placement groups, EBS volume types, etc.).
  - _Launch Configurations_ are the older, legacy version. You should know they exist, but always prefer Launch Templates for new deployments.
- **The Auto Scaling Group (ASG) Itself:**
  - This is the core configuration where you define the logic for scaling.
  - **Min/Max/Desired Capacity:**
    - **Min Size:** The smallest number of instances the ASG will maintain. It will never go below this number.
    - **Max Size:** The largest number of instances the ASG can scale out to. It will never exceed this number.
    - **Desired Capacity:** The number of instances the ASG should have at any given time. If you don't set up scaling policies, the ASG will simply maintain this number of instances.
- **Scaling Policies:**
  - These are the rules that tell the ASG when to launch new instances (scale out) and when to terminate existing instances (scale in).

---

### **2. Scaling Policies: How to Scale**

- **Manual Scaling:** You manually change the `Desired Capacity` of the ASG. This is useful for predictable, one-time events.
- **Scheduled Scaling:** You schedule scaling actions to occur at a specific time. For example, you can set the `Desired Capacity` to 10 every weekday at 9 AM and scale it down to 2 at 6 PM. This is perfect for predictable traffic patterns.
- **Dynamic Scaling:** The ASG automatically adjusts capacity in response to changing demand, based on Amazon CloudWatch metrics.
  - **Target Tracking Scaling (Simplest & Recommended):** This is the easiest to configure. You pick a metric (like "Average CPU Utilization") and set a target value (e.g., 50%). The ASG then automatically calculates the number of instances needed to keep the metric at or near the target value. If CPU goes to 70%, it will scale out. If it drops to 30%, it will scale in.
  - **Step Scaling:** You define a set of scaling adjustments (steps) that vary based on the size of the alarm breach. For example, if CPU is between 60-75%, add 1 instance. If CPU is above 75%, add 3 instances. This gives more granular control than target tracking.
  - **Simple Scaling:** The legacy policy. You define a CloudWatch alarm, and when it's breached, a single scaling action is triggered (e.g., "add 2 instances"). It then waits for a cooldown period to complete before it will respond to any new alarms. It's less responsive than Step or Target Tracking.

---

### **3. Health Checks and Cooldowns**

- **Health Checks:** The ASG periodically checks the health of its instances.
  - **EC2 Status Checks:** By default, if an instance fails its EC2 system or instance status check, the ASG will consider it unhealthy, terminate it, and launch a replacement to maintain the desired capacity.
  - **ELB Health Checks:** If you attach a load balancer to your ASG, it can use the ELB's health checks. If the ELB marks an instance as unhealthy, the ASG will terminate it and launch a replacement. This is a best practice for web applications.
- **Cooldown Period:** A cooldown period is a configurable setting that prevents your ASG from launching or terminating additional instances before the effects of a previous scaling activity are visible. After a scale-out event, the default cooldown is 300 seconds (5 minutes). This gives the newly launched instances time to boot, install software (via User Data), and start handling traffic, preventing the ASG from scaling out again prematurely based on temporarily high metrics.
- **Termination Policies:** This policy determines which instance the ASG will terminate during a scale-in event. The default policy is designed to be balanced and safe, generally terminating the instance with the oldest launch configuration or the instance closest to the next billing hour.

---

### **4. Lifecycle Hooks**

Lifecycle hooks are a powerful feature that allows you to pause an instance as it is being launched or terminated by the ASG and perform custom actions.

- **Launch Hook (`Pending:Wait`):** Pauses a new instance _before_ it is put into service (`InService` state).
  - **Use Case:** Download and install software, run complex configuration scripts, or perform validation checks before the instance starts receiving traffic from the load balancer.
- **Terminate Hook (`Terminating:Wait`):** Pauses an instance that has been selected for termination _before_ it is actually shut down.
  - **Use Case:** Safely drain connections, upload log files to S3, or back up stateful data before the instance is permanently deleted.

After your custom action is complete, you send a `complete-lifecycle-action` command to the ASG to tell it to continue with the launch or termination. If you don't, the instance will be terminated after a timeout period (default is 1 hour).

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. An e-commerce website experiences a massive, unpredictable surge in traffic every time a new product is announced. The company wants to automatically add EC2 instances to handle the load based on CPU utilization. Which Auto Scaling Group feature should they use?**

- A. Scheduled Scaling
- B. Manual Scaling
- C. Dynamic Scaling with a Target Tracking policy
- D. Lifecycle Hooks

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The traffic is unpredictable, so scheduled or manual scaling would not be effective. A Dynamic Scaling policy is required. Target Tracking is the simplest and most effective way to manage scaling based on a metric like "Average CPU Utilization." You set a target (e.g., 60%), and the ASG handles adding and removing instances automatically to maintain that target.

</details>

---

**2. An Auto Scaling Group has a Min Size of 2, a Max Size of 10, and a Desired Capacity of 4. A scaling policy is configured to add two instances if CPU utilization exceeds 80%. If one of the four running instances fails its EC2 status check, what will the ASG do?**

- A. Nothing, as the number of instances (3) is still above the minimum.
- B. Terminate the failed instance and launch one new instance to replace it, returning the count to 4.
- C. Terminate the failed instance and launch two new instances due to the scaling policy.
- D. Terminate the failed instance and reduce the desired capacity to 3.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A core function of an Auto Scaling Group is to maintain the desired capacity. If an instance becomes unhealthy, the ASG will automatically terminate it and launch a replacement to bring the instance count back to the Desired Capacity, which is 4 in this case. The scaling policy is not triggered because the health check failure is not related to CPU utilization.

</details>

---

**3. A company has a predictable workload where they need 10 instances during business hours (9 AM - 5 PM) and only 2 instances overnight and on weekends. What is the most cost-effective and appropriate way to manage this scaling?**

- A. A Target Tracking policy based on Network I/O.
- B. Manually changing the desired capacity twice every day.
- C. Scheduled Scaling actions.
- D. A Simple Scaling policy.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Since the traffic pattern is completely predictable and time-based, Scheduled Scaling is the perfect solution. You can create one action to set the desired capacity to 10 at 9 AM and another action to set it back to 2 at 5 PM. This is fully automated and more reliable than manual changes (B).

</details>

---

**4. When an Auto Scaling Group scales in, it needs to terminate an instance. Which instance will it terminate by default?**

- A. The instance with the lowest CPU utilization.
- B. The newest instance that was launched most recently.
- C. The instance with the most open network connections.
- D. The instance launched from the oldest launch template or configuration.

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** The default termination policy is complex, but its primary goal is to create balance across Availability Zones. If a choice has to be made within an AZ, it will prioritize terminating the instance launched from the oldest launch template or launch configuration. This helps you phase out old configurations as you update your templates.

</details>

---

**5. What is the purpose of an Auto Scaling Group Cooldown Period?**

- A. To prevent the ASG from terminating instances too quickly during a scale-in event.
- B. To give an instance time to cool down its CPU after a period of high utilization.
- C. To ensure that after a scaling activity, the ASG does not launch or terminate more instances until the impact of the first activity is realized.
- D. To set a time limit on how long a lifecycle hook can pause an instance.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The cooldown period is a safety mechanism. After a scale-out, for example, it can take a few minutes for a new instance to boot, get configured via User Data, and start affecting the group's average metrics. The cooldown period prevents the ASG from seeing the still-high metric and scaling out again unnecessarily.

</details>

---

**6. Before an EC2 instance in an ASG is terminated during a scale-in event, a custom script must be run to upload log files to an S3 bucket. Which ASG feature should be used to achieve this?**

- A. A custom termination policy.
- B. A `Terminating:Wait` lifecycle hook.
- C. A User Data script.
- D. A scheduled scaling action.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This is the exact use case for a Terminating:Wait lifecycle hook. It pauses the termination process, keeping the instance running in a Terminating:Wait state. This gives your custom script time to execute (e.g., copy logs to S3). Once the script is done, it signals the hook to complete, and the ASG proceeds with the termination.

</details>

---

**7. An Auto Scaling Group is configured with a Launch Template that specifies an `m5.large` instance type. The ASG's desired capacity is 3. What is the function of the Launch Template?**

- A. It defines the rules for when to scale in and scale out.
- B. It specifies the configuration (AMI, instance type, etc.) for any new instance launched by the ASG.
- C. It monitors the health of the instances in the group.
- D. It defines the minimum and maximum size of the group.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The Launch Template (or older Launch Configuration) is the blueprint for the instances. The ASG itself handles the scaling rules (A), health checks (C), and group size (D), but it refers to the Launch Template to know what kind of instance to create when it needs to scale out.

</details>

---

**8. Which type of scaling policy should be used if you want to set a target for "Average Network In" on your fleet of EC2 instances and have the ASG automatically manage capacity to keep the metric at that target?**

- A. Simple Scaling
- B. Step Scaling
- C. Scheduled Scaling
- D. Target Tracking Scaling

<details>
<summary>View Answer</summary>

**Answer: D**

**Explanation:** Target Tracking Scaling is designed for this "set it and forget it" approach. You choose a metric (like Average CPU, Network In/Out, or even a custom metric) and a target value. The ASG handles all the underlying alarms and calculations to add or remove instances to stay as close to your target as possible.

</details>

---

**9. An Auto Scaling Group is attached to an Application Load Balancer. The ASG is configured to use ELB health checks. What happens if an instance in the group fails the ELB's health check?**

- A. The ELB will try to reboot the instance.
- B. The ASG will mark the instance as unhealthy, terminate it, and launch a replacement.
- C. The instance will be detached from the ASG but will continue running.
- D. An email will be sent to the root account administrator.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** When you configure an ASG to use ELB health checks, you are telling the ASG to trust the ELB's judgment. If the ELB determines an instance is unhealthy (e.g., the application is not responding), it reports this to the ASG. The ASG's primary function is to maintain a fleet of healthy instances at the desired capacity, so it will terminate the failed instance and launch a new one.

</details>

---

**10. What is the key difference between a Launch Template and a Launch Configuration?**

- A. Launch Configurations are more flexible and support versioning.
- B. Launch Templates can only specify the AMI and instance type.
- C. Launch Templates are the newer, recommended standard that supports versioning and more EC2 features.
- D. Launch Templates are required for Spot Instances, while Launch Configurations are for On-Demand.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Launch Templates are the successor to Launch Configurations. They are superior because they support multiple versions, allowing you to iterate on your configuration. They also support a wider range of EC2 parameters, such as mixing instance types, placement groups, and advanced storage options, which are not available in the older Launch Configurations.

</details>

---

**11. An instance is being launched by an ASG that has a launch lifecycle hook configured. In what state will the instance be paused?**

- A. `InService`
- B. `Terminating:Wait`
- C. `Pending:Wait`
- D. `Stopped`

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A launch lifecycle hook intercepts the instance after it has launched but before it is fully registered and put into service. This intermediate state is called Pending:Wait. The Terminating:Wait state is for termination lifecycle hooks.

</details>

---

**12. An Auto Scaling group has a desired capacity of 5. All 5 instances are healthy. A new scaling policy is added to set the desired capacity to 7. What will happen?**

- A. The ASG will terminate two instances.
- B. The ASG will launch two new instances.
- C. The ASG will do nothing until an instance fails a health check.
- D. The desired capacity will be updated, but no instances will be launched until the next scheduled action.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The ASG's job is to always match the current number of running instances to the desired capacity. If the desired capacity is changed from 5 to 7, the ASG will immediately begin launching 2 new instances to meet the new target.

</details>

---

**13. A company wants to maintain a minimum of 2 instances running at all times for high availability, but allow the application to scale up to 20 instances during peak load. How should the ASG be configured?**

- A. Min Size: 2, Max Size: 20, Desired Capacity: 2
- B. Min Size: 2, Max Size: 2, Desired Capacity: 20
- C. Min Size: 20, Max Size: 20, Desired Capacity: 2
- D. Min Size: 2, Max Size: 10, Desired Capacity: 10

<details>
<summary>View Answer</summary>

**Answer: A**

**Explanation:** The Min Size sets the floor for the number of instances, ensuring high availability. The Max Size sets the ceiling, controlling costs. The Desired Capacity will be the starting point and will be adjusted by any dynamic scaling policies between the min and max values.

</details>

---

**14. Which scaling policy is considered the legacy option and is less responsive because it must wait for a cooldown period to complete before initiating another scaling action?**

- A. Target Tracking Scaling
- B. Step Scaling
- C. Simple Scaling
- D. Predictive Scaling

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Simple Scaling is the original dynamic scaling policy. Its major drawback is that after it triggers a scaling activity, it must wait for the entire cooldown period to expire before it can respond to any other alarms, even if the metric continues to get worse. Step and Target Tracking policies are more intelligent and can respond more quickly.

</details>

---

**15. What are the two types of health checks an Auto Scaling group can use? (Select TWO)**

- A. IAM Status Checks
- B. EC2 Status Checks
- C. VPC Flow Log Checks
- D. ELB Health Checks
- E. S3 Bucket Health Checks

<details>
<summary>View Answer</summary>

**Answers: B and D**

**Explanation:** An ASG can natively use the built-in EC2 Status Checks (both system and instance status). Additionally, if an Elastic Load Balancer is attached, the ASG can be configured to use the application-level ELB Health Checks to determine if an instance is healthy.

</details>

---

**16. What is the primary benefit of using an Auto Scaling Group?**

- A. It automatically provides static IP addresses for your instances.
- B. It provides elasticity and high availability for your application by automatically adjusting capacity.
- C. It encrypts all data on the EC2 instances.
- D. It provides a way to logically group instances for billing purposes.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The core value proposition of Auto Scaling is elasticity—the ability to scale capacity up or down to match demand—and high availability—the ability to automatically replace failed instances. This leads to better performance for users and improved cost-efficiency.

</details>

---

**17. A lifecycle hook is triggered, and the instance enters a wait state. The custom script on the instance finishes its task successfully. What must the script do to allow the ASG to proceed?**

- A. The script must reboot the instance.
- B. The script must call the `CompleteLifecycleAction` API action.
- C. The script must delete itself.
- D. Nothing, the ASG will proceed automatically after 1 minute.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A lifecycle hook requires an explicit signal to continue. After the custom action is finished, the script must make an API call to CompleteLifecycleAction (typically via the AWS CLI), telling the ASG that it's okay to move the instance to the next state (e.g., from Pending:Wait to InService).

</details>

---

**18. Can an Auto Scaling group launch instances across multiple AWS Regions?**

- A. Yes, by specifying multiple regions in the launch template.
- B. No, an Auto Scaling group is a regional resource and can only launch instances in subnets within its own region.
- C. Yes, if it is connected to a Global Accelerator.
- D. No, it can only launch instances in a single Availability Zone.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** An Auto Scaling group operates within a single AWS Region. You can (and should) configure it to launch instances across multiple Availability Zones within that region for high availability, but it cannot span regions. To achieve multi-region deployment, you would need to create a separate ASG in each target region.

</details>

---

**19. You want to create an Auto Scaling policy that adds 1 instance if CPU is between 70% and 80%, but adds 3 instances if CPU is greater than 80%. Which policy type offers this level of granular control?**

- A. Target Tracking Scaling
- B. Simple Scaling
- C. Step Scaling
- D. Manual Scaling

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Step Scaling is designed for this scenario. It allows you to define multiple "steps" for a CloudWatch alarm. You can specify different scaling adjustments (e.g., add 1 instance, add 3 instances) based on how far the metric has breached the alarm threshold.

</details>

---

**20. An ASG is configured with a Min size of 1 and a Max size of 1. What is the primary function of this ASG?**

- A. To provide rapid scaling for a high-traffic website.
- B. To ensure that there is always exactly one instance running, automatically replacing it if it fails.
- C. To test scaling policies without launching instances.
- D. This is an invalid configuration.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** This configuration is often called a "self-healing" setup. With Min=Max=Desired=1, the ASG's only job is to ensure that one healthy instance is always running. If the running instance fails its health check for any reason, the ASG will terminate it and launch an identical replacement, providing automated recovery.

</details>

---

**21. A new version of an application has been deployed, and the Launch Template for an ASG has been updated to use a new AMI. How can the existing instances in the ASG be updated to the new version with minimal disruption?**

- A. Delete the ASG and create a new one.
- B. Manually terminate each old instance one by one.
- C. Use the "Instance Refresh" feature of the ASG.
- D. Update the Desired Capacity to zero and then back to the original value.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Instance Refresh feature is designed for this exact purpose. It allows you to perform a rolling update of the instances in your ASG. It will gradually terminate old instances and launch new ones based on the updated launch template, respecting health checks and cooldowns to ensure the application remains available throughout the process.

</details>

---

**22. If you do not define any scaling policies for an Auto Scaling Group, how will it behave?**

- A. It will not launch any instances.
- B. It will launch the "Max Size" number of instances.
- C. It will simply maintain the "Desired Capacity" number of healthy instances.
- D. It will scale based on default CPU utilization metrics.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Without any dynamic or scheduled scaling policies, the ASG acts as a self-healing mechanism. It will launch the number of instances specified in the Desired Capacity field and then work to maintain that exact number, replacing any instances that fail their health checks.

</details>
