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
  - [Security Hub](#security-hub)
  - [DataSync](#datasync)
  - [Storage Gateway](#storage-gateway)
  - [FSx](#fsx)
  - [DMS](#dms)
  - [CloudFront](#cloudfront)
  - [GuardDuty](#guardduty)
  - [Inspector](#inspector)
  - [WAF](#waf)
  - [Secrets Manager](#secrets-manager)

## VPC

DX + s2s VPN: **Dedicated and encrypted connection**
!["DX + VPN"](dx-vpn.png)

VPC Interface endpoint policies:
!["vpce policies"](vpce-policy.jpg)

**PrivateLink**: PrivateLink uses ENIs within the client VPC so that there are no IP address conflicts with the service provider. You can access PrivateLink endpoints over VPC peering, VPN, and AWS Direct Connect connections. PrivateLink supports VPC peering, which makes it possible for customers to privately connect to a service, even if that service's endpoint resides in a different Amazon VPC that is connected using VPC peering.

NAT GW unsolicited inbound connections, you can run the query using only the first 2 octets (dstAddr=VPC CIDR, srcAddr=Inbound IP):
!["NAT GW"](nat.jpg)

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

Set a budget alert and move the exceeding budget account to a restricted OU
!["Budgets"](budgets.jpg)

GuardDuty: When you use GuardDuty with an AWS Organizations organization, you can designate any account within the organization to be the GuardDuty delegated administrator. **Only** the organization **management account can designate GuardDuty delegated administrators**. An account that is designated as a delegated administrator becomes a **GuardDuty administrator account**, has GuardDuty automatically enabled in the designated Region and is granted permission to enable and **manage GuardDuty for all accounts in the organization within that Region**. The other accounts in the organization can be viewed and added as GuardDuty member accounts associated with the delegated administrator account.

## EC2

Debugging EC2 instances before they are terminated by ASG
!["ASG processes"](ASG-processes.jpg)
!["EC2 termination protection"](termination-protection.jpg)

## KMS

`GenerateDataKey` API call for client side encryption:
!["KMS encryption"](client-encryption.jpg)

## Security Hub

You cannot use AWS Security Hub to manage AWS WAF rules across accounts in the organization, rather you need to use **AWS Firewall Manager** to accomplish this.

## DataSync

DataSync copies data over the internet or AWS Direct Connect.

In most situations, **you deploy the agent as a virtual machine in the same local network as your source storage**. This approach minimizes network overhead associated with transferring data by using network protocols such as Network File System (NFS) and Server Message Block (SMB) or when accessing your object storage that's compatible with the Amazon S3 API. This setup is common regardless of the endpoint type you use to connect your agent to AWS.

## Storage Gateway

S3 File GW: Storage Gateway updates the file share cache automatically when you write files to the cache locally using the file share. However, **Storage Gateway doesn't automatically update the cache when you upload a file directly to Amazon S3**. When you do this, you must perform a `RefreshCache` operation to see the changes on the file share. If you have more than one file share, then you must run the `RefreshCache` operation on each file share.

## FSx

In a Multi-AZ deployment, Amazon FSx automatically provisions and maintains a standby file server in a different Availability Zone. Any changes written to disk in your file system are synchronously replicated across Availability Zones to the standby. If there is planned file system maintenance or unplanned service disruption, Amazon FSx automatically fails over to the secondary file server, allowing you to continue accessing your data without manual intervention.

You can test the **failover** of your Multi-AZ file system by **modifying its throughput capacity**. When you modify your file system's throughput capacity, Amazon FSx switches out the file system's file server.

!["fsx"](fsx.jpg)

## DMS

RDS to Redshift: The Amazon Redshift cluster must be in the same AWS account and the same AWS Region as the replication instance. During a database migration to Amazon Redshift, AWS DMS first moves data to an Amazon S3 bucket. When the files reside in an Amazon S3 bucket, AWS DMS then transfers them to the proper tables in the Amazon Redshift data warehouse. AWS DMS creates the S3 bucket in the same AWS Region as the Amazon Redshift database

## CloudFront

To avoid the **307 Temporary Redirect response**, send requests only to the Regional endpoint in the same Region as your S3 bucket. CloudFront forwards requests to the default S3 endpoint ( s3.amazonaws.com). The default S3 endpoint is in the us-east-1 Region. If you must access Amazon S3 within the first 24 hours of creating the bucket, you can change the origin domain name of the distribution. The domain name must include the Regional endpoint of the bucket. For example, if the bucket is in us-west-2, you can change the origin domain name from `awsexamplebucketname.s3.amazonaws.com` to `awsexamplebucket.s3.us-west-2.amazonaws.com`.

## GuardDuty

GuardDuty is an intelligent threat detection service that continuously monitors your **AWS accounts, Amazon Elastic Compute Cloud (EC2) instances, Amazon Elastic Kubernetes Service (EKS) clusters, and data stored in Amazon Simple Storage Service (S3)** for malicious activity without the use of security software or agents. GuardDuty generates detailed security findings that can be used for security visibility and assisting in remediation. GuardDuty can monitor reconnaissance activities by an attacker such as unusual API activity, intra-VPC port scanning, unusual patterns of failed login requests, or unblocked port probing from a known bad IP.

## Inspector

Amazon Inspector is an automated vulnerability management service that continually scans Amazon Elastic Compute Cloud (EC2) and container workloads for software vulnerabilities and unintended network exposure.

## WAF

- Amazon CloudFront distribution
- Amazon API Gateway REST API
- Application Load Balancer
- AWS AppSync GraphQL API
- Amazon Cognito user pool

## Secrets Manager

Secrets Manager can't rotate secrets for AWS services running in Amazon VPC private subnets because these subnets don't have internet access. To rotate the keys successfully you need to configure an Amazon VPC interface endpoint to access your Secrets Manager Lambda function and private Amazon Relational Database Service (Amazon RDS) instance.

Steps that need to be followed:

1. Create security groups for the Secrets Manager VPC endpoint, Amazon RDS instance, and the Lambda rotation function
2. Add rules to Amazon VPC endpoint and Amazon RDS instance security groups
3. Attach security groups to AWS resources
4. Create an Amazon VPC interface endpoint for the **Secrets Manager service** and associate it with a security group
5. Verify that the Secrets Manager can rotate the secret
