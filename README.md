# AWS Certified Solutions Architect Professional (SAP-C02) EXAM NOTES - David Galera, October 2025

- [AWS Certified Solutions Architect Professional (SAP-C02) EXAM NOTES - David Galera, October 2025](#aws-certified-solutions-architect-professional-sap-c02-exam-notes---david-galera-october-2025)
  - [VPC](#vpc)
  - [Cloudwatch Logs](#cloudwatch-logs)
  - [IAM](#iam)
  - [CloudTrail](#cloudtrail)
  - [RDS](#rds)
  - [Organizations](#organizations)
  - [EC2](#ec2)
  - [KMS](#kms)

## VPC

VPC Interface endpoint policies:
!["vpce policies"](vpce-policy.jpg)

**PrivateLink**: PrivateLink uses ENIs within the client VPC so that there are no IP address conflicts with the service provider. You can access PrivateLink endpoints over VPC peering, VPN, and AWS Direct Connect connections. PrivateLink supports VPC peering, which makes it possible for customers to privately connect to a service, even if that service's endpoint resides in a different Amazon VPC that is connected using VPC peering.

## Cloudwatch Logs

**CWLogs Subscription Filter**: You can use a subscription filter with **Kinesis Streams**, **Lambda**, or **Kinesis Data Firehose**. Logs that are sent to a receiving service through a subscription filter are **Base64 encoded and compressed with the gzip format**.

## IAM

**IAM Access Analyzer** analyzes CloudTrail logs to determine whether external access is granted. Both IAM Access Analyzer and Macie findings can be reported in Security Hub. Security Hub has integration with Organizations, so you can use one single Security Hub dashboard to monitor for both security issues in one place.

**Permission Boundaries**: You can attach permissions boundaries only to a **user** or **role**, **not a group**.

## CloudTrail

**AWS CloudTrail data events** capture the last 90 days of bucket-level events (for example, `PutBucketPolicy` and `DeleteBucketPolicy`), and you can enable object-level logging. These logs use a JSON format. After you enable object-level logging with data events, review the logs to find the IP addresses used with each upload to your bucket. It might take a few hours for AWS CloudTrail to start creating logs.

## RDS

**Disaster Recovery**: In addition to using Read Replicas to reduce the load on your source DB instance, you can also use Read Replicas to implement a DR solution for your production DB environment. If the source DB instance fails, you can promote your Read Replica to a standalone source server. Read Replicas can also be created in a different Region than the source database. Using a **cross-Region Read Replica** can help ensure that you get back up and running if you experience a regional availability issue.

**Automated Backups**: The automated backup feature of Amazon RDS enables point-in-time recovery for your database instance. Amazon RDS will backup your database and transaction logs and store both for a user-specified retention period. If it’s a Multi-AZ configuration, backups occur on the standby to reduce I/O impact on the primary. Amazon RDS supports **single Region** or **cross-Region** automated backups.

**Database cloning** is only available for Aurora and not for RDS.

## Organizations

An **SCP** cannot be overridden, even when users within the account have **administrative rights**.

## EC2

Debugging EC2 instances before they are terminated by ASG
!["ASG processes"](ASG-processes.jpg)
!["EC2 termination protection"](termination-protection.jpg)

## KMS

`GenerateDataKey` API call for client side encryption:
!["KMS encryption"](client-encryption.jpg)
