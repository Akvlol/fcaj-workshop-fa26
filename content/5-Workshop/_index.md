---
title: "Workshop"
date: 2026-20-19
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building the SecureTable Restaurant Management System with AWS Serverless

#### Overview

**AWS Serverless** enables developers to build and deploy applications without managing servers directly. Services such as AWS Lambda, Amazon API Gateway, and Amazon DynamoDB handle requests, store data, and scale according to demand, reducing operational overhead.

In this workshop, we will learn how to build and deploy **SecureTable**, a restaurant management system on AWS that focuses on ingredient inventory management, QR code ordering, order processing, bill management, and revenue tracking.

The system uses a Serverless architecture, combining AWS services to deliver the web interface, process business logic, authenticate users, store data, and monitor system operations.

The main components we will deploy include:

- **Frontend – Amazon S3 & Amazon CloudFront:** Host and deliver web interfaces for customers and staff over HTTPS.
- **Authentication – Amazon Cognito & AWS IAM:** Authenticate staff, manage permissions, and control access to AWS resources.
- **Backend – AWS Lambda & Amazon API Gateway:** Build APIs for ordering, table and bill management, kitchen operations, inventory management, and reporting.
- **Database – Amazon DynamoDB:** Store restaurant data and use DynamoDB Transactions for operations that require data consistency.
- **Automation – Amazon EventBridge Scheduler & Amazon SNS:** Schedule reports and data backups, and send email notifications.
- **Monitoring & Security – Amazon CloudWatch, AWS CloudTrail & AWS WAF:** Monitor the system, record AWS activity, and provide additional web protection when WAF is enabled.

By the end of this workshop, we will have a working SecureTable system deployed on AWS: customers can scan QR codes to place orders, staff can manage restaurant operations, and administrators can view operational data.

The workshop also covers testing, monitoring, and resource optimization based on the principles of the **AWS Well-Architected Framework**.


#### Content

1. [System Overview & Architecture](5.1-System-overview-architecture)
2. [Prerequiste](5.2-Prerequiste/)
3. [Access S3 from VPC](5.3-S3-vpc/)
4. [Access S3 from On-premises](5.4-S3-onprem/)
5. [VPC Endpoint Policies (Bonus)](5.5-Policy/)
6. [Clean up](5.6-Cleanup/)