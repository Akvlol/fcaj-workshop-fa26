---
title : "Frontend Deployment with Amazon S3 & CloudFront"
date : 2024-01-01 
weight : 3 
chapter : false
pre : " <b> 5.3. </b> "
---

In this section, we will deploy the **SecureTable** frontend to **Amazon S3** and deliver the website through **Amazon CloudFront**. The system uses **a single private S3 bucket** to store the Guest and Staff pages, with CloudFront providing HTTPS access.

## 1. Create an S3 Bucket

1. Open **Amazon S3** in the AWS Console and select **Create bucket**.
2. Select **AWS Region: Asia Pacific (Singapore) – `ap-southeast-1`**.
3. Enter a bucket name.
4. Keep **Object Ownership: ACLs disabled** and enable **Block all public access**.
5. Leave the remaining settings at their defaults, then select **Create bucket**.

![S3 Setup](../../../images/5-Workshop/5.3-Frontend-dev/s3-configuration.png)

**Note:** You do not need to enable **S3 Static Website Hosting** or make the bucket public. CloudFront accesses the content through **Origin Access Control (OAC)**.

## 2. Upload the Frontend

1. Open the newly created bucket and select **Upload**.
2. Upload the SecureTable frontend source code available [here](https://github.com/Akvlol/fcaj-workshop-fa26/tree/main/src_code/FrontEnd).
3. Select **Upload** to complete the process.

Place `index.html` in the **bucket root directory** so CloudFront can load the default page.

![S3 File Upload](../../../images/5-Workshop/5.3-Frontend-dev/s3-upload.png)

**Note:** The paths in the HTML files must match the actual file locations in S3. S3 hosts only static content; staff authentication and order processing will be implemented in later sections.

## 3. Create a CloudFront Distribution

1. Open **Amazon CloudFront** and select **Create distribution**.
2. Choose **Amazon S3** as the origin, then select the frontend bucket you created.
3. Use **Origin Access Control (OAC)** with **Sign requests (recommended)** to allow CloudFront to read from the private bucket.
4. Leave **Origin path** empty because the frontend files are located in the bucket root directory.
5. Select the **CloudFront Free Plan** for the distribution and enable **AWS WAF** within the plan's included features.
6. Create the distribution and verify that the **S3 bucket policy** grants read access to CloudFront. If the Console does not update the policy automatically, apply the bucket policy provided by CloudFront.
7. After creation, configure:
   - **Default root object:** `index.html`
   - **Viewer protocol policy:** `Redirect HTTP to HTTPS`
8. Save the changes and wait until the distribution status becomes **Deployed**.

![CloudFront origin and OAC configuration](../../../images/5-Workshop/5.3-Frontend-dev/OAC.png)
![CloudFront distribution settings](../../../images/5-Workshop/5.3-Frontend-dev/distribution_settings.png)

CloudFront provides a domain name in the format `dxxxxxxxxxxxxx.cloudfront.net`. This is the address used to access SecureTable in the workshop.

**Notes:**
- The **six-month AWS Free Plan** and the **CloudFront Free Plan** are separate programs. Verify that the distribution uses the CloudFront Free Plan so its included features remain within the free usage limits.
- **AWS WAF** has been enabled for CloudFront. Its configuration and monitoring will be covered in [5.8 – Security Hardening & System Monitoring](5.8-Security/).
- If you cannot create a distribution because your account requires verification, follow [5.2 – Prerequisites](5.2-Prerequisite).

## 4. Verify the Website

1. Open `https://<cloudfront-domain>/` and check the Guest interface.
2. Open `https://<cloudfront-domain>/staff.html` and check the Staff interface.

![Guest Interface](../../../images/5-Workshop/5.3-Frontend-dev/guest_interface.png)
![Staff Interface](../../../images/5-Workshop/5.3-Frontend-dev/staff_interface.png)

**Note:** If the website returns `403 AccessDenied`, check **OAC**, the **S3 bucket policy**, the **Default root object**, and the location of `index.html`. If CloudFront continues serving outdated content after a frontend update, create an **Invalidation** for the paths that need to be refreshed.

At this point, the **frontend is being delivered through CloudFront**. Cognito login and routing `/v1/*` requests to API Gateway will be configured in the corresponding sections.