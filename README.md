# AWS Cloud Storage: S3 Bucket Versioning, Web Hosting & Lifecycle Management

## Table of Contents
- [Introduction to Cloud Object Storage](#introduction-to-cloud-object-storage)
- [Project Overview](#project-overview)
- [Objective](#objective)
- [System Specifications & Storage Baseline](#system-specifications--storage-baseline)
- [Storage & Versioning Architecture (ASCII Flowchart)](#storage--versioning-architecture)
- [Deployment Methodology](#deployment-methodology)
  - [Phase 1: MFA-Secured Root Authentication & S3 Dashboard Access](#phase-1-mfa-secured-root-authentication--s3-dashboard-access)
  - [Phase 2: S3 Bucket Architecture & Access Control Setup](#phase-2-s3-bucket-architecture--access-control-setup)
  - [Phase 3: Object Versioning & Server-Side Encryption Configuration](#phase-3-object-versioning--server-side-encryption-configuration)
  - [Phase 4: Asset Upload & Browser Endpoint Verification](#phase-4-asset-upload--browser-endpoint-verification)
  - [Phase 5: Document Iteration & Multi-Version Object Tracking](#phase-5-document-iteration--multi-version-object-tracking)
  - [Phase 6: Multi-Stage Object Purge & Safe Bucket Deletion](#phase-6-multi-stage-object-purge--safe-bucket-deletion)
- [Operational Relevance & Storage Governance](#operational-relevance--storage-governance)
- [Ethical Guidelines & Disclaimer](#ethical-guidelines--disclaimer)

---

## Introduction to Cloud Object Storage
**Amazon Simple Storage Service (Amazon S3)** is an object storage service offering industry-leading scalability, data availability, security, and performance. Unlike traditional file systems (which store files in hierarchical folder trees) or block storage (like Amazon EBS, which attaches to virtual machines), S3 stores data as discrete objects identified by unique keys inside flat containers called **buckets**.

A foundational feature of S3 is **Bucket Versioning**. Versioning keeps multiple variants of an object in the same bucket. When enabled, S3 generates a unique Version ID every time a file with the same key name is overwritten or uploaded. This provides built-in protection against accidental overwrites, inadvertent deletions, and malicious data tampering.

## Project Overview
This project documents the complete lifecycle of an AWS S3 bucket. Using the **AWS Management Console**, a general-purpose bucket is created in the `us-west-2` (Oregon) region with Object Versioning enabled and public access settings configured. 

The lab walks through uploading web and graphic assets, verifying in-browser rendering via presigned S3 URLs, committing multiple revisions of an `index.html` file to analyze the underlying version stack, and executing the mandatory multi-stage deletion process (emptying all versioned objects before terminating the container).

## Objective
To configure, test, inspect, and safely decommission an Amazon S3 storage container, demonstrating:
1. Multi-factor authenticated console access to storage control planes.
2. Bucket provisioning with default server-side encryption (SSE-S3).
3. Enabling and validating Object Versioning across repeated file commits.
4. Serving static web content and diagram assets over HTTP/HTTPS.
5. Understanding S3 deletion safeguards and executing safe resource teardown.

## System Specifications & Storage Baseline
* **Cloud Platform:** Amazon Web Services (AWS)
* **Target Region:** `us-west-2` (US West - Oregon)
* **Bucket Type:** General purpose (multi-AZ redundant object storage)
* **Storage Class:** S3 Standard
* **Object Ownership:** ACLs disabled (Bucket Owner Enforced)
* **Default Encryption:** Server-Side Encryption with Amazon S3 managed keys (SSE-S3)
* **Bucket Key:** Enabled (reduces KMS request volume)
* **Bucket Versioning:** Enabled
* **Staged Artifacts:**
  * `index.html` (Baseline webpage: 145.0 B -> Updated cyber attack infographic version: 1.9 KB)
  * `Type-of-Computer-Network.jpg` (Network topology diagram: 435.3 KB)
  * `1712250288886.png` (Threat infographic asset: 78.8 KB)

---

## Storage & Versioning Architecture

```text
+----------------------------------------------------------------------------------------------------+
|                                AWS S3 BUCKET LIFECYCLE ARCHITECTURE                                |
+----------------------------------------------------------------------------------------------------+

                          [ AWS Root / IAM Management Console ]
                                            |
                                  (1) MFA Authenticated
                                            |
                                            v
                            +-------------------------------+
                            |   Amazon S3 Service Engine    |
                            +---------------+---------------+
                                            |
                                            |-- (2) Provisions Bucket: General Purpose
                                            |-- (3) Sets SSE-S3 Encryption & Bucket Key
                                            |-- (4) Enables Bucket Versioning
                                            |
                                            v
+----------------------------------------------------------------------------------------------------+
| S3 Bucket Container: pratiksha-27 (Region: us-west-2)                                              |
|                                                                                                    |
|  +----------------------------------------------------------------------------------------------+  |
|  | OBJECT REVISION STACK (Key: index.html)                                                      |  |
|  |                                                                                              |  |
|  |  [Version 3 (Latest)] -> Version ID: RDsnqwDviB70JMosuSrmkdon9aNQ4HLe (1.9 KB)               |  |
|  |                          Contains: "Types of Cyber Attacks" Diagram                          |  |
|  |                                                                                              |  |
|  |  [Version 2 (Historic)] -> Version ID: FOPeau4W7Y1VnV9lyBdpk3jkc3oLPIWN (1.9 KB)             |  |
|  |                            Intermediate revision preserved under stack                       |  |
|  |                                                                                              |  |
|  |  [Version 1 (Initial)]  -> Version ID: tQ0kHgf3rcna4YtBYvyH6ZY2wWNQf_AB (145.0 B)            |  |
|  |                            Contains: Baseline Webpage Header                                 |  |
|  +----------------------------------------------------------------------------------------------+  |
|                                                                                                    |
|  +----------------------------------------------------------------------------------------------+  |
|  | STATIC ASSETS INVENTORY                                                                      |  |
|  | - Type-of-Computer-Network.jpg (435.3 KB)                                                    |  |
|  | - 1712250288886.png (78.8 KB)                                                                |  |
|  +----------------------------------------------------------------------------------------------+  |
+----------------------------------------------------------------------------------------------------+
                                            |
                        +-------------------+-------------------+
                        |                                       |
                        v                                       v
         [ In-Browser Public Endpoint ]             [ Resource Decommissioning ]
         - Renders Presigned Image                 - Empty: Purge 5 Objects (518 KB)
         - Renders Webpage & Infographic           - Delete: Final Bucket Termination
```

---

## 🚀 Deployment Methodology

### Phase 1: MFA-Secured Root Authentication & S3 Dashboard Access

Navigated to the AWS Management Console landing page (`aws.amazon.com/console`) and selected **Sign in**.
<br>

![AWS Console Landing](images/01-open-aws-website.png)

Selected the **Root user** radio button and entered the account root email address.
<br>

![Enter Root Email](images/02-login-as-root-user.png)

Supplied the root master password.
<br>

![Enter Root Password](images/03-enter-password.png)

Encountered the mandatory **Additional verification required** challenge and entered the dynamic 6-digit TOTP code (`189148`) from Google Authenticator.
<br>

![Root MFA Verification](images/04-enter-mfa-code.png)

Completed the visual security CAPTCHA verification (`4t4hsh`) to authenticate the session.
<br>

![Security CAPTCHA](images/05-human-verification.png)

Authenticated into **Console Home** in the `us-west-2` (Oregon) region.
<br>

![Console Home](images/06-logged-in-successfully.png)

Searched for **S3** in the global navigation bar and opened the Simple Storage Service console.
<br>

![Search S3 Service](images/07-search-for-s3.png)

Landed on the Amazon S3 overview page and clicked **Create bucket**.
<br>

![S3 Landing Create Bucket](images/08-open-s3-dashboard-click-create-bucket.png)

---

### Phase 2: S3 Bucket Architecture & Access Control Setup

Under **General configuration**, configured container parameters:
* **AWS Region:** `us-west-2` (US West - Oregon)
* **Bucket type:** `General purpose` (replicates data across a minimum of three Availability Zones)
* **Bucket namespace:** `Global namespace`
* **Bucket name:** Configured a globally unique bucket name (`arjun-21`)
<br>

![Bucket General Configuration](images/09-config-s3-bucket.png)

Under **Object Ownership**, selected **ACLs disabled (recommended)** to ensure Bucket Owner Enforced security.
<br>

![Object Ownership ACLs](images/10-select-ownership-acl.png)

Under **Block Public Access settings for this bucket**, unchecked **Block *all* public access** and acknowledged the warning box to allow static website hosting and asset delivery.
<br>

![Public Access Configuration](images/11-blocked-public-access.png)

---

### Phase 3: Object Versioning & Server-Side Encryption Configuration

Under **Bucket Versioning**, selected **Enable** to preserve, retrieve, and restore every version of every object stored in the bucket.
<br>

![Enable Bucket Versioning](images/12-enable-bucket-versioning.png)

Under **Default encryption**, verified server-side encryption was configured with Amazon S3 managed keys (**SSE-S3**) and enabled **Bucket Key** to optimize encryption cost.
<br>

![Default Encryption SSE-S3](images/13-select-default-encryption.png)

Under **Advanced settings**, verified **Object Lock** remained disabled and clicked **Create bucket**.
<br>

![Commit Bucket Creation](images/14-advance-settings-create-bucked.png)

The console confirmed instantiation with a success banner: *"Successfully created bucket"*. The bucket opened with 0 objects.
<br>

![Bucket Created Confirmation](images/15-bucket-created.png)

---

### Phase 4: Asset Upload & Browser Endpoint Verification

Clicked **Upload** to open the object staging interface.
<br>

![S3 Upload Console](images/16-add-objects-files.png)

Clicked **Add files**, selected the initial networking diagram (`Type-of-Computer-Network.jpg`), verified the target destination URI (`s3://pratiksha-27`), and clicked **Upload**.
<br>

![Add Object Files](images/17-files-selected-click-upload.png)

Received upload confirmation: *"Upload succeeded - 1 file, 435.3 KB (100.00%)"*.
<br>

![Upload Succeeded Banner](images/18-file-upload-successful.png)

Inspected the **Objects** table, confirming `Type-of-Computer-Network.jpg` was indexed under the S3 Standard storage tier.
<br>

![Object Inventory](images/19-check-uploaded-file.png)

Selected the object and clicked **Open**. AWS generated a secure presigned URL (`.s3.us-west-2.amazonaws.com`), and the browser rendered the networking diagram (LAN, WAN, MAN, CAN, HAN).
<br>

![Browser Rendering Image](images/20-verify-uploaded-file.jpg)

Uploaded the baseline `index.html` document (145.0 B) into the bucket root.
<br>

![Upload Baseline Index HTML](images/21-uploaded-index-file.png)

Opened `index.html` in the browser, verifying the initial header output:
> **Welcome To My S3 Webpage @RootCipherX**
<br>

![Initial Webpage Rendered](images/22-s3-bucket-as-static-website.png)

---

### Phase 5: Document Iteration & Multi-Version Object Tracking

Uploaded an updated revision of `index.html` (1.9 KB) referencing the image asset. Toggled the **Show versions** switch to inspect the version stack:
* **Latest Version:** Version ID `FOPeau4W7Y1VnV9lyBdpk3jkc3oLPIWN` (1.9 KB)
* **Historic Version:** Version ID `tQ0kHgf3rcna4YtBYvyH6ZY2wWNQf_AB` (145.0 B) preserved underneath
<br>

![First Versioning Verification](images/23a-s3-bucket-version-verified.png)

Uploaded a second infographic image (`1712250288886.png`, 78.8 KB) and committed a third version of `index.html` (1.9 KB). The bucket tracked all 5 distinct object iterations simultaneously:
* `index.html` (Latest: `RDsnqwDviB70JMosuSrmkdon9aNQ4HLe`)
* `index.html` (Previous: `FOPeau4W7Y1VnV9lyBdpk3jkc3oLPIWN`)
* `index.html` (Original: `tQ0kHgf3rcna4YtBYvyH6ZY2wWNQf_AB`)
* `1712250288886.png` (Asset: `gaZrsphUlB3jNhpT4h2qaWVbTsKzgOhP`)
* `Type-of-Computer-Network.jpg` (Asset: `BErTmIQPVeWmzJZHqYpXhLVjo.azFZmd`)
<br>

![All Object Versions Stack](images/23b-s3-bucket-version-verified.png)

Refreshed the browser endpoint for `index.html`, verifying the updated DOM layout displaying the **Types of Cyber Attacks** diagram.
<br>

![Updated Webpage With Infographic](images/24-index-file-updated-version-too.png)

---

### Phase 6: Multi-Stage Object Purge & Safe Bucket Deletion

To prevent accidental data loss, AWS forbids deleting a bucket that contains active or versioned objects. Navigated back to the master **Buckets** catalog, selected the bucket, and clicked **Empty**.
<br>

![Select Bucket to Empty](images/25-select-bucket-to-empty.png)

To satisfy the safety confirmation challenge, typed `"permanently delete"` into the text field and clicked **Empty**.
<br>

![Confirm Permanent Delete](images/26-verify-to-empty.png)

Received confirmation: *"Successfully emptied bucket"*. All 5 objects (totaling 518.0 KB across all versions) were permanently purged. Clicked **Exit**.
<br>

![Bucket Empty Success](images/27-s3-bucket-empty-done.png)

Selected the empty bucket in the catalog and clicked **Delete**.
<br>

![Select Bucket to Delete](images/28-select-bucket-to-delete.png)

Reviewed the final deletion prompt, entered the bucket name into the confirmation field, and clicked **Delete bucket**.
<br>

![Confirm Final Bucket Deletion](images/29-confirm-bucket-deletion.png)

Received the final confirmation banner: *"Successfully deleted bucket"*. The S3 console returned to a clean baseline showing **0 General purpose buckets**, eliminating any ongoing storage costs.
<br>

![Bucket Deleted Baseline](images/30-bucket-deleted-successfully.png)

---

## Operational Relevance & Storage Governance

This deployment illustrates critical principles of enterprise cloud storage:

* **Object Versioning Protection:** In production, versioning safeguards against ransomware, accidental mass deletions, and application deployment bugs. Even if an object is "deleted" by an application, S3 simply places a reversible *delete marker* on top of the stack rather than destroying the data.
* **Deletion Safeguards & WORM Compliance:** AWS strictly requires buckets to be completely emptied—including all noncurrent version IDs and delete markers—before a bucket can be destroyed. For compliance workloads requiring immutability (SEC Rule 17a-4), **S3 Object Lock** can be layered on top to prevent even root users from deleting data during a retention period.
* **Cost & Lifecycle Hygiene:** While versioning preserves historic data, keeping old revisions indefinitely incurs ongoing storage costs. In production, teams pair versioning with **S3 Lifecycle Rules** to automatically transition noncurrent versions to lower-cost tiers (like S3 Glacier Flexible Retrieval) or expire them after a defined retention window (e.g., 90 days).

---

## Ethical Guidelines & Disclaimer
This lab was completed within an authorized personal Amazon Web Services account strictly for cloud systems administration, storage versioning verification, and educational purposes. All objects and storage buckets were safely decommissioned and deleted following validation to maintain optimal account hygiene and avoid unexpected charges.
