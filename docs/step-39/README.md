# Step 39: CloudFront Basics & Edge Caching

## 🎯 Objective

- [x] Master: **CloudFront Basics & Edge Caching**

## 📘 Notes

### **Deep Dive: CloudFront Basics & Edge Caching**

Amazon CloudFront is a fast, highly secure, and programmable Content Delivery Network (CDN). Its primary purpose is to deliver your content (websites, videos, APIs) to end-users with lower latency and higher transfer speeds by caching it at locations physically closer to them.

---

### **1. How CloudFront Works: The Global Network**

CloudFront's performance comes from its massive global network of physical locations.

- **Edge Locations:** These are hundreds of data centers located in major cities around the world. When a user requests your content, CloudFront routes the request to the Edge Location that can best serve the user, typically the one with the lowest network latency.
- **Origin:** This is the source of the definitive version of your files. The origin can be an **S3 bucket**, an **EC2 instance**, an **Elastic Load Balancer**, or any custom HTTP server (even on-premises).

**The Basic Workflow:**

1. A user in Hanoi, Vietnam, requests an image from your website, `www.example.com`.
2. DNS routes the request not to your origin server in the US, but to the nearest CloudFront Edge Location (e.g., in Singapore).
3. CloudFront checks its cache at the Singapore Edge Location.
   - **Cache Hit:** If the image is already in the cache, CloudFront immediately delivers it to the user from the edge location. This is extremely fast.
   - **Cache Miss:** If the image is not in the cache, CloudFront forwards the request back to your origin server (e.g., your S3 bucket in `us-east-1`).
4. The origin server sends the image back to the Singapore Edge Location.
5. CloudFront caches the image at the edge location for future requests and forwards it to the end-user.
6. The next user in that geographic area who requests the same image will now get a cache hit, benefiting from the low latency.

---

### **2. Edge Caching & Time to Live (TTL)**

Caching is the core concept of a CDN. CloudFront stores a copy of your content at its edge locations to serve it directly to users.

- **Time to Live (TTL):** This is the amount of time, in seconds, that an object stays in the CloudFront cache before CloudFront checks with the origin again to see if an updated version is available.
  - **Long TTL (e.g., days or weeks):** Good for static content that rarely changes, like images, CSS, and JavaScript files. This maximizes cache hits and reduces load on your origin.
  - **Short TTL (e.g., seconds or minutes):** Good for dynamic content that changes frequently.
  - **Zero TTL:** You can set a TTL of 0 to prevent caching, forcing CloudFront to go back to the origin for every request. This is useful for highly dynamic API calls but misses the performance benefit of caching.
- **Cache Control Headers:** The TTL is controlled by cache-related HTTP headers, such as `Cache-Control max-age` or `Expires`, which are sent by your origin server. You can also override these and set TTL values directly in the CloudFront distribution settings.
- **Invalidation:** If you update a file at the origin (e.g., replace `logo.png`), you don't have to wait for the TTL to expire. You can perform an **invalidation** request to CloudFront, which tells it to evict the file from all edge caches globally. This forces CloudFront to fetch the new version on the next request. Invalidation requests incur a small cost.

---

### **3. Key CloudFront Concepts**

- **Distribution:** This is the main CloudFront configuration that defines how your content is delivered. You create a distribution and get a unique domain name (e.g., `d12345.cloudfront.net`).
- **Origin Access Identity (OAI):** A special CloudFront virtual user identity. Its purpose is to allow you to secure your S3 bucket origin. You can create a bucket policy that only allows read access to the OAI, effectively making your S3 bucket private while still allowing CloudFront to serve its content. This is a critical security best practice.
- **Geo-Restriction (Geoblocking):** You can configure your CloudFront distribution to prevent users in specific geographic locations (countries) from accessing your content.

---

### **AWS SAA-C03 Style Questions & Explanations**

**1. A company based in Ireland hosts its main website on an S3 bucket in the `eu-west-1` region. They are launching a marketing campaign in Australia and expect a large number of new users. They want to ensure the Australian users have a fast, low-latency experience when loading the website's images and videos. What is the BEST service to use?**

- A. S3 Transfer Acceleration
- B. An Application Load Balancer
- C. Amazon CloudFront
- D. S3 Cross-Region Replication to the `ap-southeast-2` (Sydney) region.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This is the primary use case for Amazon CloudFront. As a Content Delivery Network (CDN), it will cache the website's static assets (images, videos) at an AWS Edge Location in Australia. This means Australian users will be served the content from a nearby location with low latency, instead of having to fetch it all the way from Ireland.

</details>

---

**2. What is the main purpose of an Amazon CloudFront Edge Location?**

- A. To host the origin EC2 instances for an application.
- B. To cache content closer to end-users to reduce latency.
- C. To run regional data processing jobs.
- D. To store the master copy of all S3 objects.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Edge Locations are the heart of a CDN. They are a global network of data centers whose primary function is to store a cache of your content. By serving content from a location that is geographically close to the user, CloudFront dramatically reduces the round-trip time and improves performance.

</details>

---

**3. A company hosts a static website in a private S3 bucket. They are using CloudFront to serve the content to the public. How can they configure access so that only CloudFront can read the objects in the S3 bucket?**

- A. By creating an IAM user for CloudFront and placing its credentials in the distribution settings.
- B. By making the S3 bucket public.
- C. By creating an Origin Access Identity (OAI) and modifying the S3 bucket policy to grant it access.
- D. By using a VPC Endpoint for S3.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** This describes the standard secure pattern for serving S3 content. An Origin Access Identity (OAI) is a special CloudFront principal. You create an OAI and then create a bucket policy that grants s3:GetObject permission specifically to that OAI. This allows you to keep your S3 bucket completely private from the public while allowing CloudFront to fetch and serve the content.

</details>

---

**4. When a user requests an object from CloudFront for the first time and the object is not in the edge cache, what is this event called?**

- A. A Cache Hit
- B. A Cache Miss
- C. A Cache Invalidation
- D. An Origin Fetch

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** A Cache Miss occurs when the requested content is not found in the cache of the Edge Location that receives the request. This triggers CloudFront to forward the request to the origin server to fetch the content.

</details>

---

**5. How does CloudFront determine how long to keep an object in its cache before checking the origin for an updated version?**

- A. Based on the object's size.
- B. Based on the Time to Live (TTL) value, which is often set by `Cache-Control` headers from the origin.
- C. All objects are cached for exactly 24 hours by default.
- D. Based on the number of times the object has been requested.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The duration an object is cached is controlled by its Time to Live (TTL). This value can be set in the CloudFront distribution's cache behavior settings, or more commonly, it is controlled by caching headers like Cache-Control: max-age=<seconds> or Expires that are sent by the origin server along with the object.

</details>

---

**6. A developer updates an important image file (`logo.png`) in their S3 origin bucket. The CloudFront distribution has a TTL of 24 hours for images. The developer needs the change to be visible to all users immediately. What action should they take?**

- A. Wait 24 hours for the cache to naturally expire.
- B. Create a new CloudFront distribution.
- C. Perform an Invalidation for the file path (`/images/logo.png`).
- D. Disable and then re-enable caching on the distribution.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A CloudFront Invalidation is the correct mechanism for force-expiring content from the cache. By creating an invalidation for /images/logo.png, you tell CloudFront to evict that object from all edge caches worldwide. The next time a user requests it, CloudFront will be forced to go back to the origin to fetch the newly updated version.

</details>

---

**7. A company wants to prevent users from specific countries from accessing their video content served through CloudFront. What CloudFront feature should they use?**

- A. AWS WAF integration.
- B. Geo-restriction (Geoblocking).
- C. Signed URLs.
- D. A custom Lambda@Edge function.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** CloudFront has a built-in Geo-restriction feature. You can configure your distribution with either a "whitelist" (allow only specified countries) or a "blacklist" (block specified countries) to control content access based on the user's geographic location.

</details>

---

**8. Which of the following can be configured as an origin for a CloudFront distribution? (Select TWO)**

- A. An Amazon S3 bucket.
- B. An AWS Lambda function.
- C. An Application Load Balancer.
- D. An Amazon EFS file system.
- E. An EBS Snapshot.

<details>
<summary>View Answer</summary>

**Answers: A and C**

**Explanation:** A CloudFront origin must be an HTTP endpoint. The most common origins are Amazon S3 buckets (for static content) and HTTP servers, which can be EC2 instances or, more commonly, an Application Load Balancer that sits in front of a fleet of EC2 instances.

</details>

---

**9. What is the primary benefit of using Amazon CloudFront for API acceleration?**

- A. It automatically encrypts all API payloads.
- B. It provides a static IP address for the API endpoint.
- C. It terminates the TCP and TLS connection at an edge location close to the user, reducing round-trip time and latency.
- D. It caches all POST requests by default.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** For dynamic content like APIs, CloudFront improves performance by optimizing the network path. It establishes a persistent, optimized connection from the edge location back to your origin. A user's request only has to travel the "bad" public internet to the nearest edge location. This termination of the connection close to the user significantly reduces latency for dynamic requests.

</details>

---

**10. What is the function of the `Cache-Control: max-age=3600` HTTP header?**

- A. It tells the user's browser to cache the object for 3600 seconds.
- B. It tells the CloudFront edge location to cache the object for 3600 seconds.
- C. Both A and B. CloudFront and the browser will both respect this header.
- D. It invalidates the object from the cache after 3600 seconds.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Cache-Control header is a standard HTTP header that provides caching instructions to all caches in the request chain. This includes intermediate caches like CloudFront and the end-user's browser cache. Both will honor the max-age directive.

</details>

---

**11. Which service is a Content Delivery Network (CDN)?**

- A. AWS Direct Connect
- B. Amazon Route 53
- C. Amazon CloudFront
- D. Elastic Load Balancer

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** Amazon CloudFront is AWS's global CDN service. Its purpose is to cache and deliver content from a network of edge locations to provide lower latency to end-users.

</details>

---

**12. When you create a new CloudFront distribution, what are you given to access your content?**

- A. An Elastic IP address.
- B. A globally unique domain name (e.g., `d12345.cloudfront.net`).
- C. A pair of access keys.
- D. An IAM role ARN.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Each CloudFront distribution is assigned its own unique domain name. You then typically create a CNAME or Alias record in your own DNS to point your friendly domain (e.g., www.example.com) to this CloudFront domain name.

</details>

---

**13. A company serves static content from an S3 bucket via CloudFront. Data transfer from S3 to CloudFront incurs what charge?**

- A. Standard S3 data transfer out fees.
- B. A reduced data transfer fee.
- C. It is free of charge.
- D. It is billed as CloudFront data transfer in.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** To encourage the use of the CDN, data transfer from an AWS origin (like S3 or EC2) to CloudFront edge locations is free. You only pay for the data transfer from CloudFront out to the internet (your users).

</details>

---

**14. A user reports that they are seeing an old version of a CSS file on a website served by CloudFront. What is the LEAST disruptive way for a developer to fix this for all users?**

- A. Delete and recreate the CloudFront distribution.
- B. Create an invalidation for the CSS file's path.
- C. Tell the user to clear their browser cache.
- D. Reduce the TTL for all objects in the distribution to 0.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** While telling the user to clear their cache (C) might fix it for them, it doesn't fix it for anyone else. The correct, global solution is to create an invalidation for the specific file. This forces all CloudFront edge locations to discard their old cached copy, ensuring everyone gets the new version on their next request.

</details>

---

**15. What is an Origin in a CloudFront distribution?**

- A. The geographic location of the end-user.
- B. A CloudFront Edge Location.
- C. The source of the content, such as an S3 bucket or an HTTP server.
- D. The DNS provider for the domain name.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The Origin is where CloudFront goes to get the definitive, master copy of your files when they are not in the cache. This can be an S3 bucket, an ELB, an EC2 instance, or any other publicly accessible HTTP endpoint.

</details>

---

**16. What is the purpose of using Signed URLs or Signed Cookies with CloudFront?**

- A. To enforce the use of HTTPS.
- B. To restrict access to your content to authenticated users or for a limited time.
- C. To cache dynamic content more effectively.
- D. To block users from certain countries.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** Signed URLs and Signed Cookies are features used to control access to private content. Your application can generate a temporary, cryptographically signed URL that grants a user access to a specific file for a limited duration. This is the standard way to serve private or paid content (e.g., video streaming subscriptions, software downloads) via CloudFront.

</details>

---

**17. What is a "Cache Hit Ratio"?**

- A. The percentage of time an object is updated at the origin.
- B. The percentage of total requests that are served directly from the CloudFront cache, without having to go to the origin.
- C. The number of edge locations where an object is cached.
- D. The TTL of an object in seconds.

<details>
<summary>View Answer</summary>

**Answer: B**

**Explanation:** The Cache Hit Ratio is a key performance metric for a CDN. A high cache hit ratio (e.g., 95%) means that 95% of requests are being served quickly from the edge, which indicates a healthy and effective caching strategy. A low ratio indicates that CloudFront is frequently having to go back to the origin, which reduces performance.

</details>

---

**18. You have a CloudFront distribution in front of an Application Load Balancer. The ALB's security group must be configured to allow traffic from where?**

- A. From the IP address `0.0.0.0/0`.
- B. From the security group of the EC2 instances.
- C. From the IP address ranges of the CloudFront edge locations.
- D. From the IP address of your office.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** If you want to lock down your origin so that it only accepts traffic from CloudFront, you must configure its security group to allow inbound traffic only from the published IP address ranges of the CloudFront edge servers. This prevents users from bypassing CloudFront and accessing your ALB directly.

</details>

---

**19. How does CloudFront handle dynamic content, like API requests?**

- A. It cannot be used for dynamic content.
- B. It caches all API requests for the default TTL.
- C. It forwards the requests to the origin over an optimized network path and can be configured to not cache the responses.
- D. It converts all dynamic requests to static content.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** CloudFront is highly effective for dynamic content. While you would typically configure it to not cache the responses (by setting TTL to 0 or respecting Cache-Control: no-cache headers), you still get a significant performance benefit. CloudFront terminates the user's connection at the edge and forwards the request to your origin over the fast and reliable AWS backbone network, reducing latency.

</details>

---

**20. What is a CloudFront "behavior"?**

- A. A health check performed on the origin.
- B. A setting that defines how CloudFront responds to a DDoS attack.
- C. A set of rules in a distribution that applies to requests matching a specific path pattern.
- D. A log of all user activity.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** A cache behavior allows you to customize how CloudFront handles requests based on the request path. For example, you can create a behavior for /images/_ with a long TTL and another behavior for /api/_ with a TTL of 0. This allows you to apply different caching, origin, and security settings for different parts of your site within a single distribution.

</details>

---

**21. An S3 bucket is configured as an origin for CloudFront. The bucket policy is updated to deny all public access. An OAI is NOT being used. What happens when a user requests content via CloudFront?**

- A. The request succeeds because CloudFront uses a special private connection.
- B. The request succeeds, but a warning is logged in CloudTrail.
- C. The request fails with a 403 Forbidden error because CloudFront can no longer access the origin object.
- D. The request is redirected to the S3 console.

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** If the S3 bucket is private and you are not using an Origin Access Identity (OAI), then CloudFront is effectively an anonymous public user. Since the bucket policy denies public access, CloudFront's request to the origin will be denied, and it will pass that 403 Forbidden error back to the end-user.

</details>

---

**22. Which HTTP header sent from an origin server would instruct CloudFront to cache an object for one hour?**

- A. `TTL: 3600`
- B. `Cache-Duration: 3600`
- C. `Cache-Control: max-age=3600`
- D. `Expires-In: 3600`

<details>
<summary>View Answer</summary>

**Answer: C**

**Explanation:** The standard HTTP header for specifying cache duration is Cache-Control. The max-age directive tells all caches (including CloudFront and browsers) the maximum time in seconds that they are allowed to serve a cached copy before re-validating with the origin.

</details>
