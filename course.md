# SAP Udemy Course

## Identity and Federation

IAM Access Advisor: See permissions granted and when last accessed

IAM Access Analyzer: Analyze resources shared with external entities. You define a Zone of trust. Everything outside of this zone will be reported as findings.

- IAM Access Analyzer Policy Validation: Provides warnings, actionable recommendations
- IAM Access Analyzer Policy Generator: Analyzes CloudTrail logs up to 90 days to generated fine-grained permission Policy for the service. Generates policies based on API calls made

IAM Policy: Effect, (Action | NotAction), Resource, Condition. **Action takes precedence over NotAction**. While Effect: Deny takes precedence over Effect: Allow. This is the difference.

IAM Policies Conditions: `"Condition": {"{condition-operator}": {"{condition-key}":"{condition-value}"}}`

Conditions e.g. `ForAllValues`: must have all keys. `ForAnyValue`: must have one of those keys

IAM Policies Variables and Tags: `"Resource": ["arn:aws:s3:::mybucket/${aws:username}/*"]`

3 types of variables:

- AWS Specific: aws:SourceIp
- Service Specific: s3:prefix
- Tag based: iam:ResourceTag/key-name

When you assume a role, you give up your original permissions and take the ones assigned to the role. This is the **fundamental difference between Roles and Resource Based Policies**

**IAM Permission Boundaries**: Supported for users and roles but not groups. Useful to restrict 1 user instead of an entire organization with SCP

STS:

- Temporary credentials are valid between 15 minutes - 12 hours. The caller impersonates the Role
- MFA protection for Roles is available
- `External ID` for 3rd party account access

AD:

- Managed AD: Forest trust (2 way trust between on premises AD and AWS), **replication (synchronization) is not supported**. To have replication: set up Microsoft AD replication on AWS on EC2 self managed replica in a VPC.
- AD Connector does not work with SQL Server
- Simple AD: Does not support MFA, SQL Server and SSO

Organizations:

- Member accounts created from the Management Account, ship with the `OrganizationAccountAccessRole` already created to be assumed (AssumeRole API) by the Management Account for Operations. If you invite an account to join to the Org, you must **manually create this OrganizationAccountAccessRole**
- Organizations All features:
  - Invited accounts must approve it
  - You can apply an SCP to prevent member accounts from leaving the Org
  - Can Not switch back to consolidated billing only
- Organization Policies:
  - Tags
  - IA
  - Backup Policies -> **Immutable (VIEW Only)** backup plans appear in member accounts

Identity Center: ABAC -> users have attributes in identity center, then you leverage a single **Permission Set** to define access based on users attributes

Control Tower Account Factory leverages Service Catalog under the hood

- **Control Tower Guardrails**: **Preventive** -> SCPs, deny access. **Detective** -> AWS Config
- Guardrails Levels: Mandatory, Strongly recommended, elective.

Resource Access Manager (RAM):

- You can **share a subnet with multiple accounts**, each account can only view its resources. You can access private IPs of EC2 from another account, you can also **reference SGs from other accounts**
- You can **share Prefix Lists and reference them in SGs**, much easy than reference the CIDR block in every SG (very mantainable)
- You can **share Route 53 outbound Resolver rules** and reference them in resolver outbound endpoint from another account

## Security

### CloudTrail

CloudTrail Events:

- **Management Events** (enabled by default in all trails): Operations performed on resources e.g. `AttachRolePolicy`, `CreateSubnet`. Can Separate **Read Events** from **Write Events**
- **Data Events** (By default data events are not logged because of volume) e.g. `S3 GetObject`, `PutObject`, lambda `Invoke` API...
- **CloudTrail Insights**: You enable it and pay for it. Continuously analyzes **management write events** to detect unusual patters, anomalies appear in **CloudTrail console**, events sent to **S3** and also has **EventBridge event generation**

Events retention: Management events, data events and insights events are **stored for 90 days in Cloudtrail**, to keep them more time you can log them to s3.

Cloudtrail can deliver notifications directly to S3. You can also configure to **send log files to S3 every 5 minutes**, these files are SSE-S3 encrypted but you can use SSE-KMS.

CloudTrail can **stream API calls to CW Logs** and then use Metric Filters to trigger Alarms

**Organizational Trail** is created in the **Management Account**, member accounts publish events to this trail and are stored in single S3 bucket with each member account ID in the prefix

**Cloudtrail may take up to 15 minutes to deliver events**, the **fastest way to react** to them is the **EventBridge Integration**. It is faster than CW logs + metric filter or S3 + event notification

### KMS

Envelope Encryption -> Symmetric Keys

Asymmetric Keys use case: Perform encryption outside of AWS for users who not have access to the KMS API

Types of Keys:

- **Customer Managed Keys**: Use these for **envelope encryption**, support **key policies** and are auditable by CloudTrail. **Rotation 90 - 2560 days (365 default)**
- **AWS Managed Keys**: Automatic rotation every year, used by service (aws/s3, aws/ebs, aws/redshift)
- **AWS Owned Keys**

Types of Key Material Origin:

- **AWS_KMS** (default): AWS creates and manages the key material
- **EXTERNAL**: You import the key material into KMS, you are responsible for managing it outside of AWS. Supports both symmetric and asymmetric KMS keys. You need to manually rotate the key since it lives outside AWS.
- **AWS_CLOUDHSM** (Custom key store): AWS KMS creates the key material inside `cloudhsm cluster`, cryptographic ops are performed INSIDE the cluster, deployed in at **least 2 AZs**. Useful when you require direct control of the HSM

KMS multi-region keys: A key can be **replicated cross-region**, replicas have the same KEY ID and key material. Replicas can be promoted into their own primary.

### Parameter Store

You can access **Secrets from Secrets Manager from Parameter store**, this is a trick. You can access **public parameters** issued by aws e.g. a parameter to access the latest amazon linux 2 AMI.

Standard vs Advanced Parameter tiers:

- 10000 vs 100000 params. Max param size 4KB vs 8KB. **Advanced parameters support parameter policies**, useful to attach TTL to params and leverage **EventBridge integration**

No automatic parameter rotation but you can use **EventBridge Rule + Lambda** to rotate parameters. You have to mantain it manually while secrets manager provides it out of the box.

### Secrets Manager

KMS encryption is mandatory

**Secret rotation** automatically seamlessly  (every 30 days) for **RDS**, **Redshift** and **DocumentDB**. You can write a **custom lambda function** for other cases.

Control Access to Secrets via Resource Based Policies

**Sharing a Secret Cross-Account**:

- Attach a **KMS policy** with `KMS:decrypt` and `KMS:ViaService` condition
- Then you create a **Secrets Manager Resource Based Policy** with `secretsmanager:GetSecretValue` allowing a User or Role

### RDS Security

- Transparent Data Encryption (TDE) for Oracle and SQL Server
- IAM authentication for **RDS PostgreSQL**, **MySQL** and **MariaDB**. Authorization happens in RDS not IAM

### SSL

Server Name Indication (SNI) works in ALB, NLB and CloudFront. Client has to indicate the **hostname of the target server**.

Route 53 supports **DNSSEC** for **domain registration** and also for DNS service using KMS. You can also run a DNS server (**Bind** is the most popular) on EC2 with DNSSEC

ACM certificates can be loaded on:

- ELB
- CloudFront
- API GW

ACM is a **REGIONAL** service

ACM Certs CANNOT be loaded onto EC2 instances but you can retrieve it from SSM Parameter store by referencing them in the **user data** script.

### CloudHSM

- Supports both **symmetric and asymmetric encryption**
- Has integration with Redshift for database encryption and key management.
- Good option to use for SSE-C, common pattern
- Uses CloudHSM client software + SSL to communicate with the hardware
- Create, Describe and Delete the cluster with IAM permissions
- You create users in the client software and manage their permissions
- Supports **SSL/TLS Acceleration** and **Oracle TDE Acceleration**
- Deployed in a VPC, can be **shared via VPC Peering**
- **MFA Support**
- Can be used to perform **SSL Offloading** from EC2 to CloudHSM thanks to **SSL Acceleration**, supported by NGINX, Apache and IIS Windows server. The SSL key NEVER leaves the HSM device -> Extra Security

### S3 Security

- SSE-KMS:

  - API Calls are logged in CloudTrial
  - Provides protection in case the bucket is public, since objects made public can never be read
  - If you use `s3:PutObject`, you also need permission `kms:GenerateDataKey`

- Glacier is AES-256 encrypted, key is under AWS control.
- HTTPS is mandatory for SSE-C
- To enforce HTTPS, use aws:SecureTransport in Bucket Policy

Event Notifications:

- Typically delivered in seconds but can take minutes
- Notification for every object if versioning enabled
- If **versioning disabled**, 2 simultaneous writes can result in just 1 notification

EventBridge integration: First you need to enable **CloudTrail object level logging** on S3

S3 Access via Bucket Policy:

- Internet GW -> use `AWS:SourceIP` **condition** for public IPs and Elastic IPs
- Endpoint GW -> use `AWS:SourceVPCe` for 1 endpoint or `AWS:SourceVPC` all VPC endpoints

**S3 Object Lock** -> Block an object version deletion from a specific amount of time

**Glacier Vault Lock** -> Lock the policy for future edits

S3 Access Points:

- Each Access Point has an Access Point Policy that grants **R, W or R/W** access to a `bucket prefix`
- Each Access Point has a DNS name connected to the internet or a VPC origin
- If using a **VPC Origin Access Point**, you have to create a **VPC GW endpoint or Interface endpoint** and this endpoint policy must **allow access to both the S3 bucket and the S3 Access Point**
- **S3 Object Lambda** works on top of Access Point, useful to redact PII data

S3 Multi-Region Access Points:

- **Single endpoint** that routes traffic to the nearest S3 bucket (lowest latency)
- **Bi-directional Cross-Region Replication** to keep data in sync
- **Failover control**, active-active or active-passive. Active-Active is for when you want to write to multiple regions at the same time.

### WebACL (layer 7)

- **ALB**
- **API GW**
- **CloudFront**
- **AppSync**

Web Access Control Lists:

- Protects against SQL injection and XSS attacks
- Rules can include **IP Addresses**, **HTTP Headers**, **HTTP body** or **URI strings**
- Size constraints, Geo Match
- Rate Based Rules

Rule Actions: ALLOW, BLOCK, COUNT, CAPTCHA, CHALLENGE

**Managed Rules**:

- **Baseline** rule groups
- **Use Case specific** rule groups: WindowsRuleSet...
- **IP Reputation** Rule Groups: Block requests based on source e.g. block malicious IPs. `AWSManagedRulesAmazonIpReputationList` (IPs trusted by AWS)
- **Bot Control Managed** Rule Group

WAF Logs:

- CW logs, up to 5 MB/s
- S3 every 5 minutes
- Kinesis Data Firehose

**Solution Architecture**: Force ELB requests from CloudFront only. Use WAF in Cloudfront to inject a header secret, then use WAF filter rule in ALB to check for this secret header. Manage the header secret with Secrets Manager.

### Firewall Manager

Manage rules in all accounts of an AWS Organization, centralized.

- WAF rules
- Shield Advanced (ALB, CLB, NLB, Elastic IP, Cloudfront)
- SG for EC2, ALB and ENIs in a VPC
- Network Firewall at VPC level
- Route 53 Resolver DNS Firewall
- Policies are applied at a **Region** Level

### Inspector

- Need to run the **SSM Agent** on **EC2**
  - Unintended network reachability
  - OS vulnerabilities
- Analyzes **Container images on ECR**
- Analyzes **lambda** function code vulnerabilities

Reports findings in **AWS Security Hub**

Sends findings to **EventBridge**

### Config

- **Regional** Service
- Integration with **SNS** for changes
- Can be aggregated across Regions and Accounts
- AWS Managed Config Rules and Custom Config Rules (defined in AWS Lambda)
- Rules are evaluated for every config change or on a schedule
- Trigger EventBridge for non-compliant rules

### AWS Managed Logs Destinations

- ELB Access Logs, S3 Access Logs, CloudFront Access Logs, Config -> S3
- Route 53 Access Logs -> CW Logs
- CloudTrail Logs -> S3 AND CW Logs
- WAF logs, VPC Flow Logs -> S3, CW Logs and Kinesis Data Firehose

### GuardDuty

Intelligent thread discovery. 1 click to enable, 30 day trial. Input data:

- CloudTrail Event Logs, both management events and S3 data events
- VPC Flow Logs
- DNS Logs
- EKS audit logs: suspicios activities in EKS cluster
- Optional Features: RDS and Aurora, EBS, Lambda network activity...

It does **not analyze WAF logs**.

Can setup EventBridge rules triggered by **GuardDuty findings**.

**Can protect against CryptoCurrency attacks**, has a dedicated finding.

**GuardDuty Delegated Administrator** for Organizations: The Management Account chooses a member account as delegated admin which has full permissions to enable and manage GuardDuty in all organization accounts

### EC2 Instance Connect

Uses `SendSSHPublicKey` API, first you create an inbound rule on port 22 SSH for the CIDR of Instance Connect which is 18.206.107.24/29. When you initiate the instance connect it pushes an SSH key valid for 60 seconds with the `SendSSHPublicKey` API. You have **60 seconds** to connect with this public key. The api call is logged in CloudTrail

### Security Hub

Central security tool to manage security accross many accounts and **automate security checks**. Integrated **dashboards** showing security and **compliance status**. **You need to enable AWS Config** to use it. Any security issue generates an event to EventBridge. You can investigate the source of the issues with `AWS Detective`

### Detective

**GuardDuty**, **Macie** and **Security Hub** generate findings, you can investigate the root source of the issues with Detective. Automatically collects events from VPC Flow Logs, CloudTrail and GuardDuty to create a **unified view** and uses **ML and graphs**

## Compute and Load Balancing

### EC2

Placement group:

- Cluster: Same rack, same AZ
- Spread: **Max 7 instances** per AZ
- Partition: 100s of instances per placement group, up to **7 partitions per AZ**, EC2 in the same partition **share the same rack**. EC2 instances can access info of the partition they are via metadata service.

You **can change the placement group of an EC2**:

- Stop
- **Use CLI** - modify-instance-placement
- Start

**Dedicated hosts**: You can define **host affinity** (instance reboots are kept on the same host)

EC2 Graviton is only available for Linux

EC2 status check metric:

- Instance Status -> Check underlying EC2 VM
- System Status -> Check underlying hardware

If any of these checks fail -> Instance recovery (action triggered by CW Alarm `StatusCheckFailed_System`) keeps:

- Metadata
- Placement group
- Private, public, elastic ip

Disk metrics (write/read bytes) only for **instance store**

`AWS ParallelCluster`:

- Open source cluster management tool to deploy HPC on AWS
- Configured with text files
- Automate creation of VPC, Subnet, cluster type and instance types.

### ASG

Update an AMI -> You need to update the launch template/configuration, then you can **terminate instances manually** or use **EC2 Instance Refresh for Auto Scaling** `StartInstanceRefresh` API, you need to specify the **min % of healthy instances** (it does a rolling refresh), you can specify warm-up time for the new instances

**All ASG processes can be suspended**: Launch, Terminate, HealthCheck, ReplaceUnhealthy, AZRebalance, AlarmNotification, ScheduledActions, AddToLoadBalancer, InstanceRefresh.

### Spot fleets

Automatically request spot instances with the lowest price

- On demand instances + spot instances
- you define `launch pools`: instance type e.g. m5.large, OS, AZ
- Spot fleets stop launching instances when **meeting capacity** or **max price**

Strategies to allocate Spot instances:

- `lowestPrice`
- `diversified`: instances launched across all pools
- `capacity optimized`: pool with optimal capacity for the number of instances
- `price capacity optimized (recommended)`: pools with the highest capacity available, then select the pool with the lowest price

### Containers

ALB Dynamic Port Mapping: multiple instances of the same task are running in the same EC2 machine, each task has a random port assigned and ALB finds the right one.

ECS tasks networking:

- none: no network connectivity
- bridge: Use Docker virtual network
- host: Bypass docker network, use the host ENI
- awsvpc: Every task gets its own ENI and private IP

ECS Auto Scaling: CPU and RAM is tracked in CW at the **Service level, not Task level**. It scales tasks not EC2 machines

Both Spot and On-demand instances are available on EC2 launch type and also `FARGATE_SPOT` (less reliable) for tasks running on Fargate

**EC2 Instance Role**: Grants permissions to the ECS container agent on the EC2 instance to perform actions like **registering with the cluster** and **pulling container images from ECR**.

**ECS Task Role**: Provides **permissions for your application code** (running inside the container) to access other AWS services, such as Amazon S3 or DynamoDB

### ECR

- Supports **Cross-Region and Cross-Account replication** 
- **Manual Scan** or **Scan on push**, results available on ECR Console
  - Basic Scanning (Common CVE), EventBridge integration for vulnerabilities
  - Enhanced Scanning (AWS Inspector), OS and application code vulnerabilities, EventBridge integration for findings

### EKS

- **Managed Node Groups**: Nodes are part of the ASG managed by EKS, supports On-Demand and Spot EC2
- **Self Managed Nodes**: You create the node and register it to the cluster, supports On-Demand and Spot EC2
- **Fargate**

EKS Data Volumes:

- Need to specify the `StorageClass` manifest on the EKS CLUSTER
- Support for EBS and **EFS (this is the only storage class that works for Fargate)**, **FSx for Lustre** and **FSx for NetApp ONTAP**

### Amazon ECS Anywhere

- Allows customers to deploy ECS tasks anywhere, on-premises
- **ECS Container Agent** and **SSM Agent** need to be deployed on the machines
- `EXTERNAL` Launch Type
- Must have a stable connection between on-premises and the target AWS Region

### Amazon EKS Anywhere

- Operate kubernetes clusters running outside of AWS
- The cluster should use **Amazon EKS distro**
- Install with the **EKS anywhere installer**
- Optionally use the **EKS Connector** to connect the cluster to AWS, 2 flavors:
  - `Fully Connected and partially Disconnected`: connect the external clusters to AWS and operate them from AWS Console
  - `Fully Disconnected`: Not connected to AWS, manage the EKS cluster on-premises

### Lambda

- Lambda cannot be deployed in Public Subnet
- If the VPC does not have a NAT GW, **CW Logs for lambda still work**

Lambda Async

- Integrations S3, SNS, EventBridge
- DLQ can be SQS and **SNS** for failed processing

### ELB

- Classic LB:
  - Cross-Zone Load balancing: disabled by default, no charges
- GW LB operates at layer 3 - IP protocol, uses `GENEVE protocol` at `port 6081`
  - Cross-Zone Load balancing: disabled by default, incurs charges
  - Target Groups: EC2 instance, **IP addresses** must be **private IPs**
- ALB
  - Cross-Zone Load balancing: Always enabled, cannot be disabled
  - target Group: EC2 instance, ECS task, lambda, **IP addresses** must be **private IPs**
- NLB (layer 4)
  - Cross-Zone Load balancing: disabled by default, incurs charges
  - Has 1 IP per AZ and supports assigning **Elastic IP**
  - Target groups: EC2 instances, private IPs and **ALB**.
  - **Regional NLB DNS**, returns the IPs of all NLB nodes (1 per AZ) e.g. `my-nlb-12345abc.nlb.us-east-1.amazon.aws.com`. **Zonal DNS Name**: Each node in the NLB has a DNS which resolves to the IP of that node e.g. **us-east-1a**.my-nlb-12345abc.nlb.us-east-1.amazon.aws.com (fronted by the zone).

Routing Algorithms:

- Least Outstanding Requests: The next instance to receive the request is the one that has the lowest number of pending/outstanding requests: CLB and ALB
- Round Robin: CLB and ALB
- Flow Hash: target based on hash of proto, target source/destin ip address and port, tcp sequence number. Each TCP/UDP is routed to the same target for the live of the connection: NLB.

### API GW

**max 10MB payload size** can be a problem for S3 proxy

**10000 req/s** (soft limit)

Supports **Resource Based Policy**

**Private endpoint type**: Only accessible from the VPC using an **VPC interface Endpoint**, a single VPCe can be used to access multiple APIs. Uses **API GW Resource Based Policy** to control access (`aws:SourceVPC` or `aws:SourceVPCe`) + VPCe policy

**Caches** are defined per **Stage** and **Method**, bypass the cache with **Cache-control: max-age=0**. Cache can be encrypted and **capacity 0.5 - 237 GB**

- **429 too many requests**: **Account level throttling** accross all APIs in a Region
- **502 Bad Gateway**: Incompatible output returned by **lambda** or out-of-order invocations because of high traffic.
- **503 Service Unavailable**
- **504 Integration Failure**: +29 seconds

Access logs can be send to CW logs or Kinesis Data Firehose

Usage Plans work at stage and method level

### AppSync

Retrieve data in **real-time with WebSocket** or **MQTT on WebSocket**

Can retrieve data from Aurora, OpenSearch, DynamoDB, Lambda, HTTP endpoints...

Cognito integration -> In the **GraphQL Schema** you specify security for Cognito Groups

### Route 53

CNAME maps a hostname to a hostname, the target of which has to be an A or AAAA record

ALIAS record targets, **not allowed for EC2 dns name (use CNAME for the DNS or A for its private IP)**:

- ELB
- Cloudfront
- API GW
- Beanstalk
- S3 website
- VPC interface endpoint
- Global Accelerator
- Other records in the hosted zone

TTL is mandatory for every record type EXCEPT ALIAS records

Routing policies:

- Simple, CANNOT be associated with health checks
- Multiple: The client randomly selects one value from the response, **no health checks**
- Weighted, can associate health checks
- Latency, can associate health checks (failover)
- Failover (active-passive), mandatory health check on the primary
- Geolocation, must create a **Default location** for non matching requests, can associate health checks
- Geoproximity, can define **bias**, you must use **Traffic Flow** feature of Route 53
- Multi-value: Up to 8 healthy records are returned, not a substitute for ELB
- IP-based routing: **You provide a list of CIDRs** e.g. route a specific ISP to a specific endpoint

Traffic Flow: Visual editor to create very complex configurations, can be saved as **Traffic Flow Policy** and have versioning

`DNSSEC` only works with **public hosted zones**

Route 53 DNS provider with 3rd party Registar: You update the **NS servers** on your domain registrar to point the ones in Route 53

**Health Checks** are only for **public resources**, pass if the endpoint responds 2xx or 3xx Status code:

- Health Checks that monitor an endpoint
- Health Checks that monitor other Health Checks (Calculated Health Checks), up to 256 child health checks.
- Health Checks that monitor a CW Alarm
- **Can be configured to fail/pass based on the value in the first 5120 bytes of the response**

To monitor a **private** resource inside a VPC (health checks can only monitor public resources): Create a CW metric and associate a CW Alarm, then monitor the alarm (not private VPC)

Health Checks can trigger CW Alarms

**Hybrid DNS**:

- Inbound Endpoint: Allow to resolve domain names for **AWS Resources** and **Private Hosted Zones**.
- Setup Inbound Endpoint: From the DNS resolver on-premises you point to the IP addresses of the **Resolver Inbound endpoint**. The resolver inbound endpoint is linked to the Route 53 resolver which queries the Private Hosted Zone
- Outbound Endpoint: Conditionally forward DNS queries to on-premises resolvers, use **Forwarding Rules**
- Setup Outbound Endpoint: A private EC2 queries Route 53 resolver which in turn queries the Outbound endpoint **forwarding rules** which point to the on-premises DNS resolver.

These endpoints can be associated with **1 or more VPC in the same Region**. Create them in 2 AZ for HA. Each endpoint supports 10000 queries/s per IP

**Resolver Rules**, can be shared Cross-Account with RAM:

- Forwarding Rules: Forward DNS queries for a specified domain and all its subdomains to target IPs
- **System Rules** overwrite Forwarding Rules (e.g. not forward a specific subdomain).
- Auto-defined system rules: Define how AWS internal domain names and Private Hosted Zones are resolved

### Global Accelerator

2 anycast IPs, Health Checks on endpoints and failover in less than 1 minute

Targets:

- Elastic IP
- EC2
- ALB, NLB, public or private

Client IP preservation **except for Elastic IP endpoints**

### Outposts

- AWS sets up and manage the rack.
- EC2, ECS, EKS, S3 are supported
- S3 Outposts storage Class, **you can create an access point and access it from your VPC** or you can use DataSync to copy it to the cloud.

## Storage

### EBS

- AZ bound -> copy snapshots to another AZ
- `st1` frequently accessed
- `sc1` less frequently accessed
- `gp2/gp3` and `io1/io2` can be used as **boot volumes**
- EBS backups consume I/O
- Fast Snapshot Restore or `fio/dd` command
- There is an **account level setting** to enable encryption in new EBS volumes, Regional setting
- `io1/io2` support **multi-attach**, must use a file system thats cluster aware, not XFS or EX4

Data LifeCycle Manager

- Use resource tags to identify resources (EC2, EBS)
- Can not be used to manage snapshots/AMIs created outside DLM
- Can not be used to manage **instance-store backed AMIs**

**Instance store**: **Data survives reboot** but data lost if you stop the EC2

### EFS

- You **pay per GB used**, scales automatically to PBs, no capacity provisioning
- Can only attached to **1 VPC**, 1 ENI (mount target) per AZ
- 1000s of concurrent NFS clients, 10GB/s throughput

**EFS Performance Mode** (set at creation time)

- **General Purpose (default)**: Latency optimized (Web serving)
- **Max I/O**: Higher latency, Higher throughput, higher parallel (big data, media processing)

**Throughput Mode**:

- Bursting: Grows in throughput when storage grows e.g. 1TB -> 50 MB/s + burst up to 100 MB/s
- Provisioned: Set your throughput regardless storage size e.g. 1TB -> 1GB/s
- Elastic: Automatically scales throughput based on workload, ideal for unpredictable workloads

**Lifecycle management feature**, moves files between storage tiers after **N days**:

- Standard -> Frequently accessed files
- EFS IA
- Archive -> 50% cheaper, access few times a year

**Availability and durability**:

- **Standard**: Multi AZ
- One Zone/One Zone IA: (90% cheaper)

External Access -> via NFS ENIs:

- VPC Peering
- DX or VPN
- To **mount NFS from on-premises servers** you must **use NFS mount target by ENI IPv4**, not DNS. EC2 instances can use DNS.
  
EFS **File System Policies**: Same as Resource Based Policies, by default grant access to all clients

Cross-Region Replication: **RPO and RTO of minutes**. Do not affect provisioned throughput

### S3

**S3 Replication time control (RTC)**: Objects are replicated in seconds and 99.99% of them within **15 minutes** or you receive an **alarm**

Limits per prefix per bucket:

- `3500 PUT/COPY/POST/DELETE`
- `5000 GET/HEAD`
- No limit of prefixes per bucket

**Multi-part upload**, recommended for > 100MB, mandatory for > 5GB

 You can use **AWS CLI** to **list** incomplete multi-part uploads. Can set up a Lifecycle policy to **delete incomplete multipart uploads** after X days, `AbortIncompleteMultipartUpload`

**S3 Byte-Range Fetch** -> Speed downloads, request different byte ranges in parallel

**S3 Analytics** also named **Storage Class Analytics**:

- Recommendations for **Standard** and **Standard IA**
- Does **NOT WORK** for **One-Zone IA** or **Glacier**
- CSV report is updated daily
- 24/48 hours to start seeing data analysis
- Visualization in `QuickSight`

**Storage Lens**:

- Understand, analyze and optimize storage accross **AWS Organization**
- Discover anomalies, cost inefficiencies, apply data protection
- Can be configured to export metrics daily to an S3 bucket (CSV, Parquet)
- Default Dashboard or create your own Dashboard
- Default Dashboard:
  - Both Free and Advanced Metrics
  - shows **multi-Region** and **multi-Account** data
  - Can't be deleted but can be disabled
- Metrics displayed:
  - `StorageBytes`, `ObjectCount` -> Identify the fastest growing or not-used Buckets or prefixes
  - Cost optimization metrics: `NonCurrentVersionStorageBytes`, `IncompleteMultipartUploadStorageBytes`
  - Data Protection Metrics: `VersionEnabledBucketCount`, `MFADeleteEnabledBucketCount`
  - Access Management Metrics: `ObjectOwnershipBucketOwnerEnforcedBucketCount`
  - Event metrics: `EventNotificationsEnabledBucketCount`
  - Performance Metrics: About Transfer Acceleration
  - Activity Metrics: About how storage is requested: `GetRequests`, `PutRequests`
  - Detailed Status Code Metrics: `200OkStatusCount`, `403ForbiddenErrorCount`
- Free Metrics ~28, data available for query for **14 days**
- Advanced Metrics and Recommendations, additional paid metrics and features
  - Activity, Advanced Cost Optimization, Advanced Data Protection, Status Code
  - CW publishing
  - Prefix aggregation
  - Data available for **15 months**

### FSx

FSx for Windows:

- **Can be mounted on Linux EC2**
- Active Directory integration
- Supports **SMB** and **Windows NTFS**
- Supports Microsoft Distributed File System Namespaces
- Storage Options:
  - SSD - small random file ops
  - HDD - large sequential file ops
- Can be accessed over DX or VPN
- Multi-AZ option
- **FSx Windows Single AZ cannot be automatically moved to Multi AZ**, leverage `DataSync` or backup and restore as Multi-AZ
- Data is backed daily in S3

Lustre -> Seamless integration with S3, can read S3 as a file system. Can write the result of computation back to S3. Accessible from Windows EC2 instances

FSx Lustre Deployment Options:

- Scratch File System
  - Temporary Storage
  - Data not replicated
  - High burst, 200 MBps per TiB
  - Usage: short time processing
- Persistent File System:
  - Data is replicated within same AZ
  - Replace failed files within minutes
  - Usage: long term processing, sensitive data

NetApp ontap:

- NFS, SMB, iSCSI
- Storage grows automatically
- Snapshots, compression low-cost
- Point in time instantaneos cloning

OpenZFS

- NFS
- Snapshots, compression low-cost
- Point in time instantaneos cloning

Good To KNOW:

- You can only increase the amount of storage.
- If you restore a backup, you can only restore to the **same size**
- When restoring a backup, **you cannot decrease the size of the FSx**. To decrease the size you must create a new FSx smaller and migrate to it with `DataSync`.
- FSx for Lustre Data Lazy Loading: Data on S3 can be lazy loaded in the app, you do not need to download everything before start the processing, data is loaded on demand (less requests and cheaper)

### DataSync

Synchronize data from NFS and SMB protos of on-premise servers with DataSync agent (up to 10 GB/s). To move data via DX you have 2 options:

- `Public VIF`: Uses public URL of the DataSync Service
- `Private VIF` to a **VPC interface endpoint** in the VPC to connect to the DataSync Service

### Data Exchange

- Suscribe to 3rd party data in the cloud e.g. **Reuters**, **Foursquare**.
- Use Data Exchange API to load data into S3
- Data Exchange for **Redshift**

### Transfer Family

- Transfer data into and out of **S3** and **EFS** using FTP proto
- FTP, SFTP or FTPS
- Multi-AZ
- Pay per provision endpoint and data transfer
- Microsoft AD, Okta, Cognito integration
- Endpoint types:
  - Public: Public IP managed by AWS, **IP can change**. **Can not set allow lists** for access control
  - VPC Endpoint with internal access, static private IP. Can allow access with SG/NACL
  - VPC Endpoint with internet-facing access: **Static private IP**, **static public IP with Elastic IP**. Can allow access with SG

## Caching

### CloudFront

- 100000 req/s easy
- **Speeds up UPLOADS** e.g. S3 (Cloudfront ingress)
- `MediaStore Container & MediaPackage Endpoint` -> Front Video-on-demand (VOD) or live streaming with other AWS Services
- `VPC Origin` for apps hosted in **VPC private subnets**
- **Custom Origin (HTTP)** -> Api GW, S3, any HTTP backend outside AWS.
- Use a **CloudFront Custom Header (X-Custom-Header)** to restrict access from CloudFront, (must be a secret) and Allow **CloudFront IPs** in the SG
- Origin Groups (primary, secondary) for failover, can be **Cross-Region**, can be used in S3 bucket + S3 Cross Region Replication

**Cloudfront Geo Restriction**:

- Define **Allow and Block Lists**.
- The country is infered from a **3rd party Geo-IP database**.
- Automatically adds a header `CloudFront-Viewer-Country` **available at Lambda@Edge**.

3 **Price Classes**:

- All - All Regions, best performance
- 200 - most Regions, excludes the most expensive ones
- 100 - Only the least expensive Regions

CloudFront Sign URLs:

- Returned by an API call into CloudFront as a **trusted signer**
- Allow access to a **path**
- Account wide key-pair, only the root user can manage it
- Can filter by IP, path, date, expiration
- Can leverage caching features

**Custom Error Pages**:

- Return an object to the viewer when origin returns 4xx or 5xx
- Specify **Error Caching minimum TTL** to specify how cloudfront caches the custom error page.

Functions:

- **CloudFront functions** are deployed at Edge Location
  - **Modify Viewer Req/Res**
  - Written in JS
  - NO access to the **request body**
  - Millions of req/s, sub millisecond start time
  - Use cases: Req normalization, header manipulation, validate JWT
  - < 1ms execution time
  - Isolation process based
- **Lambda@Edge** is deployed at the Regional edge cache
  - Written in **NodeJS or Python**
  - Can call external APIs e.g Cognito
  - Access to the **request body**
  - Scale to 1000s req/s
  - Modify **Viewer Req/Res and Origin Req/Res**
  - Can route to a **different origin**
  - Must be authored at `us-east-1` and Cloudfront replicates it to all Regional Caches
  - 5 second viewer trigger, 30 seconds origin trigger
  - Isolation VM based
  - Use Case:
    - Modify origin based on **User-Agent** e.g. load lower resolution images for mobiles
    - Lambda@Edge queries DynamoDB Global tables for a Global App
- Both **CloudFront functions** and **Lambda@Edge** **can be used together** BUT CANNOT if **Lambda@Edge** also operates the **Viewer Req/Res**, we cannot use both **CloudFront functions** and **Lambda@Edge** on the **Viewer Req/Res**

### ElastiCache

`Redis` has **backup and restore features**

`Memcached` -> Non persistent cache, data is **sharded but not replicated**. Self-managed and serverless versions. Not Multi-AZ like Redis