# CS471 Cloud Computing — Lab 05: Amazon S3 (Object Storage) + Static Website

**Student:** Ashraf Ali  
**Course:** CS471 Cloud Computing Lab — Fall 2026  
**Institution:** National University of Technology (NUTECH), Department of Computer Science  
**Lab Title:** Amazon S3 (Object Storage) + Static Website Hosting  

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Deployment Steps](#deployment-steps)
3. [Bucket Configuration](#bucket-configuration)
4. [Security Notes](#security-notes)
5. [Cost Considerations](#cost-considerations)
6. [Troubleshooting](#troubleshooting)
7. [EC2 vs. S3 Comparison](#ec2-vs-s3-comparison)
8. [Post-Lab Answers (Questions 60–67)](#post-lab-answers)
9. [Individual Reflection](#individual-reflection)

---

## Architecture Overview

```
Website Files (HTML, CSS, Images)
              │
              ▼
    Amazon S3 Bucket (Storage)
              │
              ▼
  S3 Static Website Endpoint (HTTP)
              │
              ▼
     Browser (End User)
```

The portfolio is deployed as a **static website** hosted directly on Amazon S3. The HTML, CSS, JavaScript, and image assets are stored as objects within an S3 bucket configured for static website hosting. S3 provides high availability, scalability, and security without requiring any underlying server instances (like EC2) or web server management (like Apache).

### Repository & File Structure

```
Lab-05/
├── images/
│   ├── profile.jpg           # Student profile photo
│   └── aws-architecture.jpg  # Cloud architecture diagram
├── index.html                # Home page (Index document)
├── about.html                # About page (Education, skills, interests)
├── projects.html             # Projects page (Academic & personal projects)
├── style.css                 # Shared responsive stylesheet
├── error.html                # Custom 404 error page (Error document)
└── README.md                 # Lab documentation and post-lab analysis
```

---

## Deployment Steps

1. **Bucket Creation:**
   - Navigated to the Amazon S3 Console.
   - Created a globally unique bucket (e.g., `cs471-aa-portfolio-2026`) in the designated AWS region.
   - Maintained default Object Ownership (**ACLs disabled**).
   - Adjusted bucket-level **Block Public Access** settings to allow public read policies as permitted for this lab demonstration.

2. **Uploading Assets:**
   - Uploaded all web assets: `index.html`, `about.html`, `projects.html`, `style.css`, and `error.html` at the bucket root.
   - Uploaded image files under the `images/` prefix (`images/profile.jpg`, `images/aws-architecture.jpg`).

3. **Enabling Static Website Hosting:**
   - In **Properties** > **Static website hosting**, selected **Enable**.
   - Designated `index.html` as the **Index document** and `error.html` as the **Error document**.
   - Recorded the generated **Bucket website endpoint URL**.

4. **Configuring Public Read Bucket Policy:**
   - Under **Permissions** > **Bucket policy**, configured an approved JSON policy granting anonymous `s3:GetObject` permission:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadForStaticWebsiteLab",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

5. **Testing & Verification:**
   - Verified functionality across desktop and incognito browser sessions.
   - Confirmed styling, inter-page navigation, error fallback, and media asset loading.

6. **Versioning Enablement:**
   - Enabled **Bucket Versioning** under bucket Properties.
   - Updated and re-uploaded `index.html` to verify version preservation and seamless live updates.

---

## Bucket Configuration

| Configuration Field | Applied Setting |
|---|---|
| **Bucket Name** | `cs471-aa-portfolio-2026` (unique lab identifier) |
| **AWS Region** | `us-east-1` (US East, N. Virginia) |
| **Object Ownership** | ACLs disabled (Recommended) |
| **Block Public Access** | Bucket-level public read allowed; account safeguards preserved |
| **Static Website Hosting** | Enabled |
| **Index Document** | `index.html` |
| **Error Document** | `error.html` |
| **Bucket Versioning** | Enabled |

---

## Security Notes

1. **Least-Privilege Resource Access:**  
   The bucket policy is restricted strictly to `"Action": "s3:GetObject"`. It does **not** allow `s3:PutObject`, `s3:DeleteObject`, or administrative modifications by unauthenticated users.
2. **Safeguarding Sensitive Data:**  
   Public buckets must only store non-sensitive public assets. No API keys, secret credentials, `.env` files, or proprietary data are ever stored in website buckets.
3. **Account Safeguards Preservation:**  
   Account-level Block Public Access controls were maintained; only the specific lab bucket was granted public read permissions.

---

## Cost Considerations

1. **Storage Class Selection:**  
   **S3 Standard** is used for static website hosting because it offers high throughput and low-latency millisecond retrieval. Archive classes (S3 Glacier / Deep Archive) cannot be used for websites because data retrieval requires minutes to hours of delay.
2. **Versioning & Cost Management:**  
   Every revision of an object in a versioned bucket consumes storage. Without management, storage costs grow continuously.
3. **Cost-Control Actions:**  
   - Monitored storage usage and regional pricing.
   - Proposed lifecycle rule to transition older versions to cheaper storage or expire them after 90 days.
   - Cleanup plan to remove unused objects and non-current versions after lab evaluation.

---

## Troubleshooting

### Issue 1: HTTP 403 Forbidden / AccessDenied
- **Symptom:** Opening the bucket website endpoint returned `403 AccessDenied`.
- **Diagnosis:** Discovered that although the bucket policy was created, bucket-level **Block Public Access** was still active, blocking the public policy from taking effect.
- **Resolution:** Updated the bucket-level Block Public Access settings to permit policies that grant public read access, then verified the resource ARN in the bucket policy matched the bucket name exactly.

### Issue 2: Broken Images / Missing CSS Styles
- **Symptom:** Webpage loaded as unstyled text, or images displayed broken icons.
- **Diagnosis:** S3 object keys are case-sensitive (`style.css` vs `Style.css`) and include their prefixes (`images/profile.jpg`).
- **Resolution:** Maintained consistent lowercase relative paths in HTML tags (`<link rel="stylesheet" href="style.css">` and `<img src="images/profile.jpg">`) and uploaded files matching exact key hierarchies.

---

## EC2 vs. S3 Comparison

| Dimension | EC2 + Apache (Lab 04) | S3 Static Website (Lab 05) |
|---|---|---|
| **What you manage** | OS, Apache web server, patches, security configurations, runtime | Bucket properties, object keys, bucket policies, and hosting settings |
| **Content Supported** | Dynamic applications, server-side scripts (PHP/Node/Python), databases, and static files | Static files only (HTML, CSS, client-side JS, images); no server execution |
| **Operations** | Server patching, monitoring, uptime management, capacity scaling | Fully managed by AWS; no instances to monitor or patch |
| **Security Controls** | EC2 Security Groups, Network ACLs, OS firewall, Linux file permissions | S3 Bucket policies, Block Public Access, IAM identity policies |
| **Cost Factors** | Charged for instance runtime (24/7), EBS volumes, public IPv4 addresses | Charged for storage (GB/month), GET requests, and outbound data transfer |
| **Scalability** | Requires Auto Scaling Groups, load balancers, and multi-AZ planning | Natively distributed, virtually infinite elasticity and auto-scaling |
| **HTTPS Support** | Configured directly on Apache via SSL/TLS certs (e.g., Let's Encrypt) | S3 website endpoints are HTTP-only; production HTTPS requires CloudFront + ACM |

---

## Post-Lab Answers

### Q60. Differentiate a bucket, object, object key, and prefix.
- **Bucket:** A top-level container in Amazon S3 for storing data. Bucket names are globally unique across all AWS accounts.
- **Object:** The fundamental entity stored in S3, consisting of the file data and its metadata.
- **Object Key:** The unique string identifier for an object within a bucket (e.g., `index.html` or `images/profile.jpg`).
- **Prefix:** A string at the beginning of an object key up to a delimiter (like `/`). S3 uses prefixes to simulate a hierarchical folder structure in the console, although S3 itself uses a flat namespace.

### Q61. Why does an S3 static website need object-read access for visitors?
By default, S3 buckets and their objects are private. When visitors enter the website URL, their web browsers initiate anonymous HTTP `GET` requests to retrieve HTML, CSS, and images. Without granting public object-read (`s3:GetObject`) permission via a bucket policy, S3 blocks these unauthenticated requests with `403 AccessDenied`.

### Q62. How does an IAM identity policy differ from a bucket policy?
- **IAM Identity Policy:** Attached to an IAM user, group, or role (an identity). It defines what that specific authenticated identity can do across AWS resources.
- **Bucket Policy:** Attached directly to an S3 bucket (a resource). It defines who (any principal, including anonymous visitors or other AWS accounts) can access the bucket and what actions they can perform.

### Q63. What does Block Public Access protect against?
Block Public Access provides centralized controls to prevent accidental exposure of S3 data. It overrides permissive bucket policies and ACLs, preventing unauthenticated public read/write access and blocking administrators from inadvertently publishing confidential data.

### Q64. Why can S3 static website hosting not replace EC2 for every application?
S3 static website hosting only serves pre-rendered client-side files (HTML, CSS, JS). It cannot execute server-side code (PHP, Python, Java, Node.js), host databases, handle server-side sessions, or process backend computations. Applications requiring server-side rendering or backend data processing still require compute services like EC2 or AWS Lambda.

### Q65. How does versioning help recovery, and what cost consideration does it introduce?
- **Recovery:** When an object is overwritten or deleted, S3 preserves previous versions. A delete operation merely adds a "delete marker," allowing previous revisions to be easily restored.
- **Cost Consideration:** Every version of an object is retained and billed for storage. Repeated updates to large files rapidly multiply storage consumption. Lifecycle rules must be configured to expire or transition non-current versions.

### Q66. Which approach would you choose for a simple static portfolio, and why?
I would choose **Amazon S3 Static Website Hosting** (with Amazon CloudFront). It is significantly cheaper than running an EC2 instance 24/7, requires zero operating system or web server maintenance, offers 99.999999999% (11 9's) data durability, and scales automatically to handle traffic fluctuations without intervention.

### Q67. Name two steps you took to improve security and control cost.
1. **Security:** Restricted public access strictly to the `s3:GetObject` action on objects only, ensuring no unauthenticated user can upload, alter, or delete files.
2. **Cost Control:** Selected the cost-efficient S3 Standard storage class for immediate static retrieval and established a plan to clean up non-current versions and demo objects following lab grading.

---

## Individual Reflection

Working with Amazon S3 static website hosting provided valuable insight into cloud storage and serverless architectures. Coming from Lab 04 where an Apache web server was configured on an EC2 virtual machine, deploying via S3 highlighted the operational benefits of cloud-managed services. Eliminating OS patching, instance lifecycle management, and server uptime monitoring makes static hosting both cost-efficient and maintenance-free. Understanding the distinction between IAM policies and resource-based bucket policies, as well as the safety net provided by Block Public Access, reinforced the critical importance of least-privilege security in cloud computing.
