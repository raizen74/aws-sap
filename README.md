# AWS Certified Solutions Architect Professional (SAP-C02) EXAM NOTES - David Galera, October 2025

- [AWS Certified Solutions Architect Professional (SAP-C02) EXAM NOTES - David Galera, October 2025](#aws-certified-solutions-architect-professional-sap-c02-exam-notes---david-galera-october-2025)
  - [VPC](#vpc)
  - [S3](#s3)
  - [API Gateway](#api-gateway)
  - [Cloudwatch Logs](#cloudwatch-logs)
  - [Identity Center](#identity-center)
  - [IAM](#iam)
  - [SSM Patch Manager](#ssm-patch-manager)
  - [OpsWorks](#opsworks)
  - [CloudTrail](#cloudtrail)
  - [Data Migration Service DMS](#data-migration-service-dms)
  - [Lambda](#lambda)
  - [Aurora](#aurora)
  - [RDS](#rds)
  - [KDS](#kds)
  - [Redshift](#redshift)
  - [Kinesis Data Firehose](#kinesis-data-firehose)
  - [Kinesis Data Streams](#kinesis-data-streams)
  - [Organizations](#organizations)
  - [Trusted Advisor](#trusted-advisor)
  - [EC2](#ec2)
  - [EBS](#ebs)
  - [ELB](#elb)
  - [ASG](#asg)
  - [EFS](#efs)
  - [SQS](#sqs)
  - [KMS](#kms)
  - [Security Hub](#security-hub)
  - [DataSync](#datasync)
  - [Storage Gateway](#storage-gateway)
  - [FSx](#fsx)
  - [Route 53](#route-53)
    - [Route 53 Health Checks](#route-53-health-checks)
    - [Route 53 Resolvers](#route-53-resolvers)
  - [CloudFront](#cloudfront)
  - [Global Accelerator](#global-accelerator)
  - [GuardDuty](#guardduty)
  - [Inspector](#inspector)
  - [WAF](#waf)
  - [Secrets Manager](#secrets-manager)
  - [SnowMobile](#snowmobile)
  - [Snowball](#snowball)
  - [Storage](#storage)
  - [Application Discovery Service](#application-discovery-service)
  - [Application Migration Service (MGM)](#application-migration-service-mgm)
  - [Migration Strategies](#migration-strategies)
  - [QuickSight enterprise edition](#quicksight-enterprise-edition)

## VPC

AWS allows **CIDR range only between /28 and /16**

Gateway vs Interface Endpoints: **Gateway endpoint uses Amazon S3 public IP addresses and does not allow access from on-premises systems**. A gateway endpoint is a gateway that you specify in your route table to access Amazon S3 from your VPC over the AWS network. **Interface endpoints** extend the functionality of gateway endpoints by using **private IP addresses** to route requests to Amazon S3 from within your **VPC**, **on-premises**, or from a **VPC in another AWS Region using VPC peering or AWS Transit Gateway**. Interface endpoints are compatible with gateway endpoints. If you have an existing gateway endpoint in the VPC, you can use both types of endpoints in the same VPC.

!["Interface vs Gateway Endpoints"](interface-gateway.jpg)

**VPC Flow log data** can be published to **Amazon CloudWatch Logs** or **Amazon S3**. You can create a flow log for a **VPC**, a **subnet**, or a **network interface**. If you create a flow log for a subnet or VPC, each network interface in that subnet or VPC is monitored. VPC Flow Logs **cannot be used to inspect the network packets**.

!["VPC Flow Logs example"](vpc-logs.jpg)

Good to know: Amazon EC2 instances running with an Amazon VPC have built-in protection against packet sniffing. It is not possible for a virtual instance running in promiscuous mode to receive or “sniff” traffic that is intended for a different virtual instance. While customers can place their interfaces into promiscuous mode, the hypervisor will not deliver any traffic to them that is not addressed to them. Even two virtual instances that are owned by the same customer located on the same physical host cannot listen to each other’s traffic

**Virtual Private Gateway**: For AWS Site-to-Site VPN connections, a virtual private gateway is the VPN concentrator on the Amazon side of the Site-to-Site VPN connection. You use a virtual private gateway or a transit gateway as the gateway for the Amazon side of the Site-to-Site VPN connection. For Direct Connect connections, you can use an **AWS Direct Connect gateway** to **connect your AWS Direct Connect connection over a private VIF to one or more VPCs** in any account that are located in the **same or different Regions**. **You associate a Direct Connect gateway with the virtual private gateway for the VPC**. The virtual private gateway must be attached to the VPC to which you want to connect.

DX + s2s VPN: **Dedicated and encrypted connection**.
Only AWS Direct Connect **public VIF** can be used to establish a dedicated network connection between your network to public AWS resources, such as an Amazon virtual private gateway IPsec endpoint.
!["DX + VPN"](dx-vpn.png)

VPC Interface endpoint policies:
!["vpce policies"](vpce-policy.jpg)

S3 Bucket policies for GW endpoints:
!["gateway policies"](gateway-endpoints.jpg)

SGs:
!["Security groups"](sg.jpg)

DX connection VIFs:
!["VIFs"](VIF.jpg)
!["VIF Types"](VIF-types.jpg)

Shared Services VPC:
!["Shared Services VPC"](shared-services-VPC.jpg)

VPN 3 flavours: hardware only, software only and a mix of hardware/software. Hardware only VPNs include both the AWS managed **AWS Site-to-Site VPN** solution and the **AWS VPN CloudHub**. You can also create a VPN connection to your remote network by using an **Amazon EC2 instance in your VPC that's running a third party software VPN appliance**.
!["VPNs"](VPN.jpg)

Transit VPC:
!["Transit VPC"](transit-vpc.jpg)

The **VPN CloudHub** operates on a simple hub-and-spoke model that you can use with or without a VPC.
!["AWS VPN CloudHub"](AWS_VPN_CloudHub.png)

**PrivateLink**: PrivateLink uses ENIs within the client VPC so that there are no IP address conflicts with the service provider. You can access PrivateLink endpoints over VPC peering, VPN, and AWS Direct Connect connections. PrivateLink supports VPC peering, which makes it possible for customers to privately connect to a service, even if that service's endpoint resides in a different Amazon VPC that is connected using VPC peering.

NAT GW unsolicited inbound connections, you can run the query using only the first 2 octets (dstAddr=VPC CIDR, srcAddr=Inbound IP):
!["NAT GW"](nat.jpg)

**Private hosted zones** accept DNS queries only from a VPC DNS server. The IP address of the VPC DNS server is the reserved IP address at the base of the VPC IPv4 network range plus two. Enabling DNS resolution allows you to use the VPC DNS server as a Resolver for performing DNS resolution. Keep this option disabled if you're using a custom DNS server in the DHCP Options set, and you're not using a private hosted zone.

**VPC sharing** (part of Resource Access Manager) allows multiple AWS accounts to create their application resources such as EC2 instances, RDS databases, Redshift clusters, and Lambda functions, into shared and centrally-managed Amazon Virtual Private Clouds (VPCs). To set this up, the account that owns the VPC (owner) shares one or more subnets with other accounts (participants) that belong to the same organization from AWS Organizations. After a subnet is shared, the participants can view, create, modify, and delete their application resources in the subnets shared with them. Participants cannot view, modify, or delete resources that belong to other participants or the VPC owner.

Connect on-premises instances to s3 privately: To access S3 using a private IP address over Direct Connect, you can create a private virtual interface for your connection and then create an interface VPC endpoint for S3 in your VPC that is associated with the virtual private gateway. The VGW must connect to a Direct Connect private virtual interface. This interface VPC endpoint resolves to a private IP address even if you enable a VPC endpoint for S3. When you access Amazon S3, you need to use the same DNS name provided under the details of the VPC endpoint.

## S3

You can create 100 buckets per account

**Byte Range Fetch**:
You can use concurrent connections to Amazon S3 to fetch different byte ranges from within the same object. This helps you achieve higher aggregate throughput versus a single whole-object request.

S3 Select ScanRange.

**S3 Select**: **Cheaper than Athena**. S3 Select simplifies and improves the performance of scanning and filtering the contents of objects into a smaller, targeted dataset by up to 400%. You can use S3 Select to retrieve a subset of data using SQL clauses, like SELECT and WHERE, from objects stored in `CSV`, `JSON`, or `Apache Parquet` format. It also works with **objects that are compressed** with `GZIP` or `BZIP2` **(for CSV and JSON objects only)** and server-side encrypted objects.

**S3 Glacier Select**: You can use SQL commands to query S3 Glacier archive objects that are in the **UNCOMPRESSED** CSV format

When data is transfered out from S3 to Cloudfront, its **free**.

**S3 standard IA has a minimum storage duration charge of 30 days** thereby making it costlier than using S3 Standard storage class for the given scenario because the data would be moved to Glacier via a Lifecycle policy immediately.
!["S3 transitions"](S3-transitions.jpg)
!["S3 Classes"](s3-classes.jpg)

Object logging:
!["S3 Object Logging"](s3-object-logging.jpg)

S3 notifications:
!["S3 Notifications"](s3-notifications.jpg)

Access Analyzer:
!["S3 Access Analyzer"](s3-access-analyzer.jpg)

RTC:
!["RTC"](rtc.jpg)

## API Gateway

!["API Gateway"](api.jpg)

Error codes:
!["API Gateway Error Codes"](code-errors.jpg)

`HTTPs APIs` only support **access logs**, they do **NOT** support **execution logs**.

**Usage Plans** can be defined at the API or API method level. When request submissions exceed the steady-state request rate and **burst limits**, API Gateway begins to throttle requests.

!["API Gateway Throttling"](api-throttling.jpg)

## Cloudwatch Logs

**CWLogs Subscription Filter**: You can use a subscription filter with **Kinesis Streams**, **Lambda**, or **Kinesis Data Firehose**. Logs that are sent to a receiving service through a subscription filter are **Base64 encoded and compressed with the gzip format**.

## Identity Center

!["SAML"](SAML.jpg)

## IAM

Grant Access to a 3rd party with cross-account Roles and `ExternalID`: Create an IAM role in your AWS account with a trust policy that trusts the Partner (Example Corp). Take a **unique external ID value from Example Corp** (trust relationship) and include this external ID condition in the role’s trust policy. At a high level, the **external ID** is a piece of data that can be passed to the `AssumeRole` API of the **Security Token Service (STS)**. You can then use the external ID in the **condition** element in a role’s **trust policy**, allowing the role to be assumed only when a certain value is present in the external ID.

**IAM Access Analyzer** analyzes CloudTrail logs to determine whether external access is granted. Both IAM Access Analyzer and Macie findings can be reported in Security Hub. Security Hub has integration with Organizations, so you can use one single Security Hub dashboard to monitor for both security issues in one place.

**Permission Boundaries**: You can attach permissions boundaries only to a **user** or **role**, **not a group**.

The `NotAction` element enables you to list services whose operations (or individual operations) are **EXEMPTED** from this restriction. `Action` takes precedence over `NotAction` this is the difference in comparison with `Allow/Deny`

Session-tags:
!["SAML session tags"](session-tags.jpg)

## SSM Patch Manager

You can use Patch Manager to apply patches for both operating systems and applications. (On Windows Server, application support is limited to updates for Microsoft applications). Patch Manager uses patch baselines, which include rules for auto-approving patches within days of their release, as well as a list of approved and rejected patches. You can install patches individually or to large groups of instances by using Amazon EC2 tags.

Systems Manager supports an SSM document for Patch Manager, `AWS-RunPatchBaseline`, which performs patching operations on instances for both security-related and other types of updates. When the document is run, it uses the patch baseline currently specified as the "default" for an operating system type.

The `AWS-ApplyPatchBaseline` SSM document supports patching on **Windows instances only and doesn't support Linux instances**. For applying patch baselines to both Windows Server and Linux instances, the recommended SSM document is `AWS-RunPatchBaseline`.

!["SSM Patch Manager"](patch-manager.jpg)

## OpsWorks

AWS OpsWorks is a configuration management service that provides managed instances of Chef and Puppet. Chef and Puppet are automation platforms that allow you to use code to automate the configurations of your servers. OpsWorks lets you use Chef and Puppet to automate how servers are configured, deployed, and managed across your Amazon EC2 instances or on-premises compute environments. **You cannot use OpsWorks for automatic patch management**.

## CloudTrail

When you enable **log file integrity validation**, CloudTrail creates a hash for every log file that it delivers. Every hour, CloudTrail also creates and delivers a file that references the log files for the last hour and contains a hash of each. This file is called a digest file. **CloudTrail signs each digest file using the private key of a public and private key pair**. After delivery, you can use the public key to validate the digest file. **CloudTrail uses different key pairs for each AWS region**.

The **digest files** are delivered to the **same Amazon S3 bucket** associated with your trail as your **CloudTrail log files**

**AWS CloudTrail data events** capture the last 90 days of bucket-level events (for example, `PutBucketPolicy` and `DeleteBucketPolicy`), and you can enable object-level logging. These logs use a JSON format. After you enable object-level logging with data events, review the logs to find the IP addresses used with each upload to your bucket. It might take a few hours for AWS CloudTrail to start creating logs.

!["CloudTrail events"](cloudtrail-events.jpg)

## Data Migration Service DMS

If you use the AWS DMS **console to create the endpoint, then DMS creates the required IAM roles and policies automatically**. If you use the AWS Command Line Interface (AWS CLI) or the AWS DMS API, you must create the IAM roles and policies manually.

RDS to Redshift: The Amazon Redshift cluster must be in the same AWS account and the same AWS Region as the DMS replication instance. During a database migration to Amazon Redshift, AWS DMS first moves data to an Amazon S3 bucket. When the files reside in an Amazon S3 bucket, AWS DMS then transfers them to the proper tables in the Amazon Redshift data warehouse. AWS DMS creates the S3 bucket in the same AWS Region as the Amazon Redshift database.

!["DMS Redshift"](dms-redshift.jpg)

You can use **AWS DMS data validation** to ensure that your data has migrated accurately from the source to the target. DMS compares the source and target records and then reports any mismatches. In addition, for a CDC-enabled task, AWS DMS compares the incremental changes and reports any mismatches. As part of data validation, DMS compares each row in the source with its corresponding row at the target and verifies that those rows contain the same data. **For this comparison, DMS issues appropriate queries to retrieve the data**. These queries consume additional resources at the source and the target as well as additional network resources.

!["DMS Validation"](dms-validation.jpg)

Premigration assessment: A premigration assessment evaluates specified components of a database migration task to help identify any problems that might prevent a migration task from running as expected. This assessment gives you a chance to identify issues before you run a new or modified task. You can then fix problems before they occur while running the migration task itself. This can avoid delays in completing a given database migration needed to repair data and your database environment.

## Lambda

!["Lambda Limits"](lambda-limits.jpg)

## Aurora

!["Aurora Global"](aurora.jpg)

**Aurora Auto Scaling** dynamically adjusts the number of Aurora **Replicas** provisioned for an Aurora DB cluster using **single-master replication**.

You define and apply a **scaling policy** to an Aurora DB cluster. The scaling policy defines the minimum and maximum number of Aurora Replicas that Aurora Auto Scaling can manage. Based on the policy, Aurora Auto Scaling adjusts the number of Aurora Replicas up or down in response to actual workloads, determined by using Amazon CloudWatch metrics and target values.

**Aurora Auto Scaling is possible for Aurora replicas and not for Aurora writer instances**. **Multi-master Aurora Cluster** architecture is needed if multiple writers are needed for any use case.

## RDS

**Disaster Recovery**: In addition to using Read Replicas to reduce the load on your source DB instance, you can also use Read Replicas to implement a DR solution for your production DB environment. If the source DB instance fails, you can promote your Read Replica to a standalone source server. Read Replicas can also be created in a different Region than the source database. Using a **cross-Region Read Replica** can help ensure that you get back up and running if you experience a regional availability issue.

**Automated Backups**: The automated backup feature of Amazon RDS enables point-in-time recovery for your database instance. Amazon RDS will backup your database and transaction logs and store both for a user-specified retention period. If it’s a Multi-AZ configuration, backups occur on the standby to reduce I/O impact on the primary. Amazon RDS supports **single Region** or **cross-Region** automated backups.

**Database cloning** is only available for Aurora and not for RDS.

!["Replication"](replication.jpg)

## KDS

!["KDS"](KDS.jpg)

## Redshift

Using Amazon **Redshift Spectrum**, you can efficiently query and retrieve structured and semistructured data from files in Amazon S3 without having to load the data into Amazon Redshift tables. Amazon Redshift Spectrum resides on dedicated Amazon Redshift servers that are independent of your cluster. Redshift Spectrum pushes many compute-intensive tasks, such as predicate filtering and aggregation, down to the Redshift Spectrum layer. Thus, Redshift Spectrum queries use less of your cluster's processing capacity than other queries.

!["Redshift"](redshift-1.jpg)

**Audit logging** is not turned on by default in Amazon Redshift. When you turn on logging on your cluster, Amazon Redshift creates and uploads logs to Amazon S3 that capture data from the time audit logging is enabled to the present time. Each logging update is a continuation of the information that was already logged.

Audit logging to Amazon S3 is an optional, manual process. When you enable logging on your cluster, you are enabling logging to Amazon S3 only. Currently, **you can only use Amazon S3-managed keys (SSE-S3) encryption (AES-256) for audit logging**.

## Kinesis Data Firehose

When a Kinesis data stream is configured as the source of a Firehose delivery stream, Firehose’s PutRecord and PutRecordBatch operations are disabled and Kinesis Agent cannot write to Firehose delivery stream directly. Data needs to be added to the Kinesis data stream through the Kinesis Data Streams PutRecord and PutRecords operations instead.

!["Firehose"](firehose.png)

## Kinesis Data Streams

Kinesis Client Library:

- You can **only use DynamoDB for checkpointing KCL**
- Each KCL application must use **its own DynamoDB table**

!["Data Streams"](data-streams.jpg)
!["Data Streams"](data-streams-2.jpg)

## Organizations

SCPs:

- An **SCP** cannot be overridden, even when users within the account have **administrative rights**
- SCPs **do not affect service-linked roles**

!["SCPs"](SCP.jpg)

Set a budget alert and move the exceeding budget account to a restricted OU
!["Budgets"](budgets.jpg)

GuardDuty: When you use GuardDuty with an AWS Organizations organization, you can designate any account within the organization to be the GuardDuty delegated administrator. **Only** the organization **management account can designate GuardDuty delegated administrators**. An account that is designated as a delegated administrator becomes a **GuardDuty administrator account**, has GuardDuty automatically enabled in the designated Region and is granted permission to enable and **manage GuardDuty for all accounts in the organization within that Region**. The other accounts in the organization can be viewed and added as GuardDuty member accounts associated with the delegated administrator account.

## Trusted Advisor

!["Trusted advisor"](trusted-advisor.jpg)

## EC2

Debugging EC2 instances before they are terminated by ASG
!["ASG processes"](ASG-processes.jpg)
!["EC2 termination protection"](termination-protection.jpg)
!["EC2 flavors"](ec2-flavors.jpg)

## EBS

General purpose: Baseline performance scales linearly at **3 IOPS per GiB of volume size**

!["EBS volumes"](volumes.jpg)

Provisioned IOPS: You can provision from 100 IOPS up to 64,000 IOPS per volume on Instances built on the Nitro System and up to 32,000 on other instances. The maximum ratio of provisioned IOPS to requested volume size (in GiB) is 50:1 for io1 volumes, and 500:1 for io2 volumes.
!["Provisioned IOPS"](provisioned-iops.jpg)

## ELB

If you need a **static IP for ALB** -> Front it with NLB, ALB does not support Elastic IP.

**Access Logs** are stored in **S3** buckets and it is **not possible** to directly write the logs to **Kinesis Data Firehose**.

**NLB does not support Least Outstanding Requests routing algorithm**. AWS suggests using the Least Outstanding Requests with an ALB when the requests for your application vary in complexity or your targets vary in processing capability.

For TCP traffic, the NLB selects a target using a **flow hash algorithm** based on the protocol, source IP address, source port, destination IP address, destination port, and TCP sequence number. The TCP connections from a client have different source ports and sequence numbers and can be routed to different targets. Each TCP connection is routed to a single target for the life of the connection.

For UDP traffic, the NLB selects a target using a **flow hash algorithm** based on the protocol, source IP address, source port, destination IP address, and destination port. A UDP flow has the same source and destination, so it is consistently routed to a single target throughout its lifetime. Different UDP flows have different source IP addresses and ports, so they can be routed to different targets.

**Load Balance traffic from a centralized VPC to 2 private Subnets**: you need to configure each NLB as an `AWS PrivateLink endpoint service` with associated VPC endpoints in the centralized VPC and then have the Application Load Balancer (ALB) in the centralized VPC point their target groups to the private IP addresses of each VPC endpoint. Finally, you will configure host-based routing on the ALB to route each application's traffic to the corresponding target group.

!["AWS privatelink"](privatelink.jpg)

## ASG

!["ASG Termination Policy"](asg-termination.jpg)

## EFS

!["EFS"](efs.jpg)

## SQS

You can use **message timers** to set an initial invisibility period for a message added to a queue. So, if you send a message with a 60-second timer, the message isn't visible to consumers for its first 60 seconds in the queue. **The default (minimum) delay for a message is 0 seconds. The maximum is 15 minutes.**

**Delay queues** let you postpone the delivery of **all** new messages to a queue for several seconds. 0 - 15 minutes.

## KMS

`GenerateDataKey` API call for client side encryption:
!["KMS encryption"](client-encryption.jpg)

## Security Hub

You cannot use AWS Security Hub to manage AWS WAF rules across accounts in the organization, rather you need to use **AWS Firewall Manager** to accomplish this.

## DataSync

!["DataSync"](datasync.png)
!["DataSync"](datasync.jpg)

DataSync copies data over the internet or AWS Direct Connect.

In most situations, **you deploy the agent as a virtual machine in the same local network as your source storage**. This approach minimizes network overhead associated with transferring data by using network protocols such as Network File System (NFS) and Server Message Block (SMB) or when accessing your object storage that's compatible with the Amazon S3 API. This setup is common regardless of the endpoint type you use to connect your agent to AWS.

!["DataSync Pros and Cons"](datasync-pros.jpg)

## Storage Gateway

S3 File GW: Storage Gateway updates the file share cache automatically when you write files to the cache locally using the file share. However, **Storage Gateway doesn't automatically update the cache when you upload a file directly to Amazon S3**. When you do this, you must perform a `RefreshCache` operation to see the changes on the file share. If you have more than one file share, then you must run the `RefreshCache` operation on each file share.

Volume Gateway:
!["Volume GW"](volume-gw.jpg)

## FSx

In a Multi-AZ deployment, Amazon FSx automatically provisions and maintains a standby file server in a different Availability Zone. Any changes written to disk in your file system are **synchronously** replicated across Availability Zones to the standby. If there is planned file system maintenance or unplanned service disruption, Amazon FSx automatically fails over to the secondary file server, allowing you to continue accessing your data without manual intervention.

You can test the **failover** of your Multi-AZ file system by **modifying its throughput capacity**. When you modify your file system's throughput capacity, Amazon FSx switches out the file system's file server.

!["fsx"](fsx.jpg)

## Route 53

Route 53 DNS requests and subsequent application traffic routed through CloudFront are inspected inline. Always-on monitoring, anomaly detection, and mitigation against common infrastructure DDoS attacks such as SYN/ACK floods, UDP floods, and **reflection attacks are built into both Route 53 and CloudFront**.

Route 53 is also designed to withstand DNS query floods, which are real DNS requests that can continue for hours and attempt to exhaust DNS server resources. Route 53 uses shuffle sharding and anycast striping to spread DNS traffic across edge locations and help protect the availability of the service.

!["Routing policy"](routing.jpg)

### Route 53 Health Checks

- Can monitor: The health of a specified resource, such as a web server, the **status of other health checks** or the status of an **Amazon CloudWatch alarm** (use it to monitor resources in Private Hosted Zones).
- !["Health Checks"](types-healthchecks.jpg)
- Route 53 must be able to establish a TCP connection with the endpoint within 4 seconds. In addition, the endpoint must respond with an HTTP status code of 2xx or 3xx within 2 seconds after connecting. **HTTPS health checks don't validate SSL/TLS certificates**, so checks don't fail if a certificate is invalid or expired.
- If you choose **HTTPS** for the value of `Protocol` in Route53 configuration, an **additional charge applies**.
- **After you create a Health Check, you can't change the value of** `String matching`. If you choose Yes for the value of `String matching`, **an additional charge applies**. The search string must appear entirely **within the first 5,120 bytes of the response body**.
- If you specify the endpoint by **domain name**, Route 53 uses **only IPv4** to send health checks to the endpoint.
- If you specify a **non-AWS endpoint**, **an additional charge applies**.
- Charges for a health check apply even when the health check is **disabled**.
- Route 53 **aggregates** the data from the health checkers, and if more than **18% of health checkers** report that an endpoint is healthy, Route 53 considers it healthy
- If you're routing traffic to any AWS resources that you can create **alias records** for, don't create health checks for those resources. When you create the alias records, you set `Evaluate Target Health` to `Yes` instead.
- If no records are healthy, Route 53 considers all the records in the group to be healthy and selects one based on the routing policy and on the values that you specify for each record

If you're creating failover records in a **private hosted zone**, note the following:

1. **Route 53 health checkers are outside the VPC**. To check the health of an endpoint within a VPC by IP address, you must assign a public IP address to an instance in the VPC.
2. You can create a CloudWatch metric, associate an alarm with the metric, and then create a health check that is based on the data stream for the alarm.

### Route 53 Resolvers

When you create a VPC, the Route 53 Resolver that is created by default, maps to a DNS server that runs on a reserved IP address for the VPC network range, plus 2. For example, the DNS Server on a 10.0.0.0/16 network is located at 10.0.0.2. For VPCs with multiple IPv4 CIDR blocks, the DNS server IP address is located in the primary CIDR block. The DNS server does not reside within a specific subnet or Availability Zone in a VPC.

The Resolver additionally contains endpoints that you configure to answer DNS queries to and from your on-premises environment. Before you start to forward queries, you create Resolver **inbound and/or outbound endpoints in the connected VPC**. These endpoints provide a path for inbound or outbound queries.

**Outbound endpoint on Route 53 Resolver**: To resolve DNS queries for any resources in the on-premises network from the AWS VPC, you can create an outbound endpoint on Route 53 Resolver and then Route 53 Resolver can conditionally forward queries to resolvers on the on-premises network via this endpoint. To **conditionally forward queries**, you need to create **Resolver rules** that specify the domain names for the DNS queries that you want to forward (such as example.com) and the **IP addresses of the DNS resolvers on the on-premises network** that you want to forward the queries to.

!["DNS Resolvers"](dns-resolvers.jpg)

## CloudFront

!["CloudFront signed-urls"](signed-urls.jpg)

For **uploads**, you can use the `POST` and `PUT` methods for your CloudFront distribution to **accelerate content uploads to the origin**, which is S3 for the given use-case.

To avoid the **307 Temporary Redirect response**, send requests only to the Regional endpoint in the same Region as your S3 bucket. CloudFront forwards requests to the default S3 endpoint ( s3.amazonaws.com). The default S3 endpoint is in the us-east-1 Region. If you must access Amazon S3 within the first 24 hours of creating the bucket, you can change the origin domain name of the distribution. The domain name must include the Regional endpoint of the bucket. For example, if the bucket is in us-west-2, you can change the origin domain name from `awsexamplebucketname.s3.amazonaws.com` to `awsexamplebucket.s3.us-west-2.amazonaws.com`.

Proxy methods **PUT/POST/PATCH/OPTIONS/DELETE** go directly to the origin from the POPs and do not proxy through the **regional edge caches**.

Origin Access Control (OAC) works only for s3, it does not support custom origins.

SSL Certificates: If the origin server returns an expired certificate, an invalid certificate, or a self-signed certificate, or if the origin server returns the certificate chain in the wrong order, CloudFront drops the TCP connection, returns **HTTP status code 502 (Bad Gateway)**, and sets the **X-Cache header to Error from CloudFront**.

!["CloudFront SSL"](ssl.jpg)

## Global Accelerator

With AWS Global Accelerator, you can shift traffic gradually or all at once between the blue and the green environment and vice-versa without being subject to DNS caching on client devices and internet resolvers, traffic dials and endpoint weights changes are effective within seconds.

AWS Global Accelerator is a service that improves the availability and performance of your applications with local or global users. It provides **2 static IP addresses that act as a fixed entry point to your application endpoints in a single or multiple AWS Regions**, such as your Application Load Balancers, Network Load Balancers or Amazon EC2 instances.

!["Global Accelerator"](AGA.jpg)

**Custom Routing Accelerator**: A new type of accelerator in Global Accelerator. It allows you to use your own application logic to deterministically route one or more users to a specific Amazon EC2 instance destination in a single or multiple AWS Regions. This is useful for use cases where you want to **control which session on an EC2 instance your user traffic is sent to**. You can direct multiple users to a unique port on your accelerator, and their traffic will be routed to a specific destination IP address and port that your application session is running on.

Custom routing accelerators **support only VPC subnet endpoints**, each containing one or more EC2 instances that are running your application. Each VPC subnet endpoint, which could be in a single or multiple Regions, contains the IP addresses of the EC2 instances that host your application. With a custom routing accelerator, you can put your accelerator in front of up to thousands of EC2 instances running in a single or multiple VPCs. Custom routing accelerators support VPC subnet endpoints with a maximum size of /17 and route traffic only to EC2 instances within each subnet.

## GuardDuty

GuardDuty is an intelligent threat detection service that continuously monitors your **AWS accounts, Amazon Elastic Compute Cloud (EC2) instances, Amazon Elastic Kubernetes Service (EKS) clusters, and data stored in Amazon Simple Storage Service (S3)** for malicious activity without the use of security software or agents. GuardDuty generates detailed security findings that can be used for security visibility and assisting in remediation. GuardDuty can monitor reconnaissance activities by an attacker such as unusual API activity, intra-VPC port scanning, unusual patterns of failed login requests, or unblocked port probing from a known bad IP.

## Inspector

You can enable Amazon Inspector for your entire organization or an individual account with a few clicks in the AWS Management Console. Once enabled, Amazon Inspector automatically discovers running Amazon EC2 instances and Amazon ECR repositories and immediately starts continually scanning workloads for software vulnerabilities and unintended network exposure.

**Amazon Inspector uses the Systems Manager (SSM) agent** to collect the software application inventory of the Amazon EC2 instances.

Network Reachability rules package in Amazon Inspector: The findings also have recommendations that include information about exactly which Security Group you can edit to remove the access. And like all Amazon Inspector findings, these can be **published to an SNS topic** for additional processing.

## WAF

- Amazon CloudFront distribution
- Amazon API Gateway REST API
- Application Load Balancer
- AWS AppSync GraphQL API
- Amazon Cognito user pool

!["WAF"](waf.jpg)
!["WAF Geomatch"](waf-geomatch.jpg)

The logging destinations that you can choose from for your AWS WAF logs are, same destinations as VPC Flow Logs:

- CloudWatch Logs
- S3
- Kinesis Data Firehose

## Secrets Manager

Secrets Manager can't rotate secrets for AWS services running in Amazon VPC private subnets because these subnets don't have internet access. To rotate the keys successfully you need to configure an Amazon VPC interface endpoint to access your Secrets Manager Lambda function and private Amazon Relational Database Service (Amazon RDS) instance.

Steps that need to be followed:

1. Create security groups for the Secrets Manager VPC endpoint, Amazon RDS instance, and the Lambda rotation function
2. Add rules to Amazon VPC endpoint and Amazon RDS instance security groups
3. Attach security groups to AWS resources
4. Create an Amazon VPC interface endpoint for the **Secrets Manager service** and associate it with a security group
5. Verify that the Secrets Manager can rotate the secret

## SnowMobile

For datasets > 10 PB

## Snowball

!["Snowball vs S3 Transfer Acceleration"](snowball.jpg)

## Storage

!["EFS pricing"](efs-pricing.jpg)
!["EBS pricing"](ebs-pricing.jpg)
!["S3 pricing"](s3-pricing.jpg)

## Application Discovery Service

AWS Application Discovery Service helps enterprise customers plan migration projects by gathering information about their on-premises data centers. ADS will discover on-premises or other hosted infrastructure. This includes details such as server hostnames, IP and MAC addresses, resource allocation, and utilization details of key resources.

Planning data center migrations can involve thousands of workloads that are often deeply interdependent. Server utilization data and dependency mapping are important early first steps in the migration process. AWS Application Discovery Service collects and presents configuration, usage, and behavior data from your servers to help you better understand your workloads.

ADS will identify server dependencies by recording inbound and outbound network activity for each server. ADS will provide details on server performance. It captures performance information about applications and processes by measuring metrics such as host CPU, memory, and disk utilization. It also will allow you to search in Amazon Athena with predefined queries.

The collected data is retained in encrypted format in an AWS Application Discovery Service data store. In addition, this data is also available in AWS Migration Hub, where you can migrate the discovered servers and track their progress as they get migrated to AWS.

!["Application Discovery Service"](ams.jpg)

## Application Migration Service (MGM)

AWS Application Migration Service (AWS MGN) is the primary migration service recommended to **lift and shift** your applications to AWS. Customers considering **CloudEndure Migration** are encouraged to use AWS Application Migration Service for future migrations.

You can use AWS Application Migration Service to perform non-disruptive tests before cutover. After testing, you can use AWS Application Migration Service to quickly **lift and shift your applications to the cloud** during a short cutover window, typically measured in **minutes**.

!["Application Migration Service"](mgn.jpg)

## Migration Strategies

**Replatforming** is a migration strategy where you don't change the core architecture but leverage some cloud optimizations.
!["Migration Strategies"](migration.png)

## QuickSight enterprise edition

Connect to RDS in a private subnet: Sign in to QuickSight as a **QuickSight admin** and create a new **QuickSight VPC connection**. When you register a VPC connection with QuickSight, you can securely connect to data that's available only in your VPC, for example: 1. Data you can reach by IP address 2. Data that isn't available on the public internet 3. Private databases 4. On-premises data
