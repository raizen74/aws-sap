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

Eventhough you already use Redis or Memcached, ElastiCache requires **application code changes**

`Redis` has **backup and restore features**

`Memcached` -> Non persistent cache, data is **sharded but not replicated**. Self-managed and serverless versions. Not Multi-AZ like Redis

## Databases

### DynamoDB

- Massive scale, millions of RPS
- Max Object size 400 kB
- Capacity
  - Provisioned RCU/WCU, optionally with auto scaling
  - On-Demand, pay for every single write and read
- Reads -> Strongly or Eventually consistent
- Supports transactions across multiple tables (ACID)
- Table Classes: **Standard** and **IA**
- Good `Sort Key`: timestamp
- `LSI`: Keep the **same PK** and select a **different SK**, defined at creation time
- You can only query by PK + SK on the main table + Indexes
- TTL at row level
- **DynamoDB Streams**
  - **24 hours retention**
- Send data directly to `Kinesis Data Streams` (up to 365 days) from `DynamoDB`, direct integration

DAX

- **Individual object, Query, Scan Cache**. For caching computations over the data use `ElastiCache`
- Does not require application code changes
- Micro second latency for cached reads & queries
- Writes go through DAX to DynamoDB
- **Solves Hot Key problem**
- 5 minutes TTL for cache by default
- Up to 10 nodes, multi-AZ (3 nodes recommended for HA)

### OpenSearch

- `Managed` cluster or `serverless`
- Kibana is named OpenSearch Dashboards
- **CW Logs Subscription Filters can send data to Firehose** to insert data into OpenSearch **near real time**

### RDS

- Launched in a VPC, control access via SG
- Storage by **EBS**, can increase volume size with auto-scaling
- Backups: Automatic with PIR. They expire.
- Snapshots: Manual
- RDS Events: get notified via SNS for events
- MultiAZ failover: The same DNS automatically resolves to the standby
- We can use Route 53 Weighted Record Set to distribute Reads across Read Replicas, + Health checks
- **Transparent Data Encryption for Oracle and SQL Server**. Encrypts data before its written to storage.
- IAM auth for Postgres, MySQL and MariaDB. You do NOT need password, just auth token (lifetime of 15 min). **SSL is mandatory**.

RDS for Oracle:

- Use **RDS backups** to restore to **RDS for Oracle**
- Use **Oracle RMAN (Recovery Manager) backups** to **restore for non-RDS**. You upload the backup into S3 and then restore in an **EXTERNAL oracle DB**
- RDS for Oracle does NOT support RAC (Real Application Clusters)
- RAC is working on Oracle on EC2 because you have full control
- DMS works on Oracle

RDS for MySQL: You can use the native `mysqldump` tool to **migrate RDS MySQL to non RDS**

RDS Proxy for lambda

- Supports **IAM auth**, **DB auth** and **auto-scaling**
- Can be public or private (lambda in VPC)

### Aurora

- Automatically grows up to 128 TB, 6 copies of data, multi-AZ
- Up to 15 RR, single read endpoint
- Cross Region Read Replicas of the entire DataBase (not tables)
- You can load and export data directly to S3
- **Single Write endpoint and Read endpoint**

HA and Read Scaling

- 4 of the 6 copies of data are needed for writes and 3/6 copies are needed for reads
- Self healing with peer2peer replication
- Storage is stripped across 100s of volumes
- Master is the only that can write to the storage
- Failover < 30s

Aurora Endpoints

- Endpoint: Host+Port
- Writer endpoint: Connects to the primary DB
- Reader endpoint: Load Balance replicas
- Custom endpoint: Represent a group of DB instances in the cluster, useful to define a parameter group specific to them
- Instance endpoint: Connect to a specific instance, diagnosis the specific instance

Aurora Logs

- Error, **slow query**, general and audit logs available as files.
- Can be **downloaded or published to CW Logs**.

Troubleshooting

- **Performance Insights**: find issues by waits,SQL statements, hosts and users
- **Enhanced Monitoring**: Host level, process view, per-second metric e.g. the top 100 processes running
- **CW Metrics**: CPU, Memory, Swap Usage

### Aurora Serverless

- Automatic instantiation, auto-scaling based on usage
- No need for capacity planning
- Good for unpredictable workloads
- Pay per second
- **You just connect to 1 endpoint**

Data API

- Access Aurora Serverless with a simple API, **without establishing a JDBC connection**
- **HTTPS** endpoint to run SQL Statements
- No persistent connection management
- Application needs IAM policy with access to **Data API and Secrets Manager (where credentials are checked)**

RDS Proxy for Aurora

- Can front the Primary Instance (write and reads)
- Can create a read only endpoint that **connects only to the Read Replicas**

Aurora Global

- You define a primary region (write/read)
- **Up to 10** secondary (read-only) regions, **replication lag < 1 second**
- Up to 16 RR per secondary region
- Promoting another region has **RTO < 1 minute**
- Ability to **manage the RPO for Aurora PostgreSQL**
- **Write Forwarding**: You can submit writes to secondary regions and are forwarded to the primary.

Convert RD to Aurora:

- Take a snapshot and restore, quick
- **Create an Aurora RR from an RDS instance**, you can promote it to primary and migrate the app to Aurora.

## Service Communication

### SNS

SNS topic suscribers: EMAIL, MOBILE, Custom HTTP(S), SQS, LAMBDA, **FIREHOSE**

**DLQ works at the subscription level, not topic level**. Every subscription has its DLQ

`Delivery Policy` is applied in case of server-side errors, `Custom Delivery Policies` are **only available for HTTP(S) suscribers**

## Data Engineering

### Kinesis Data Streams

Data **CAN'T be deleted from Kinesis** until expiration (365 days)

Ordering via **Partition ID**

To process messages in Real-time, Data Streams is much more cheap than DynamoDB streams

**Enhanced Fanout** reduces latency

**Provisioned Mode**:

- Choose the number of shards
- Provisioned Mode: 1 MB/s or 1000 req/s producer, 2 MB/s out consumer
- Scale the number of shards manually
- Pay per shard provisioned per hour, **0.015$/hour**

**On-Demand Mode**:

- No need to provision shards
- **Default** capacity provided: **4 MB/s or 4000 req/s producer**
- **Scales automatically** based on observed throughput peak during the last 30 days
- Pay per stream/hour and data in/out per GB

Api GW has service integration with Kinesis Data Streams

### Amazon Data Firehose

- Input records up to 1 MB
- No Data Storage like Data Streams
- Writes data in batches -> near real time
- 3rd party integrations: Datadog, splunk, mongoDB, New Relic...
- Custom destinations: HTTP
- Source Records can be stored in S3
- If you have a **lambda transform failure or a delivery failure**, you can store the data in S3
- For CSV to JSON -> use lambda transform
- Provides **data conversion** JSON to Parquet/ORC (only for S3 dest) without lambda transform
- Supports compression when **target is S3**: GZIP, ZIP, Snappy
- If target is Redshift (COPY from S3), only GZIP
- **Kinesis Producer Library KPL CAN send data to Firehose**
- **Kinesis Client Library KCL CANNOT read from Firehose, can only read from kinesis Data Streams**

Buffer Size:

- Based on **Time and Size e.g. 32MB and 2 min**, whatever first flushes the buffer
- Firehose can **automatically increase the buffer size to improve throughput**
- **Minimum buffer time of 1 minute**

### Managed Service for Apache Flink

- **Reads ONLY from Kinesis Data Streams and MSK**
- **CANNOT read from Firehose**
- Java, Scala or SQL

### MSK

- Data is stored in EBS for as much as you want
- Option to use MSK Serverless: You do not manage capacity
- 1 MB per message (default) can configure for higher
- MSK consumers: Flink, Glue, lambda or Custom consumer Apps (EC2, ECS, EKS)

### Batch

AWS Batch, container jobs can run on:

- Fargate
- EC2 & Spot Instances in VPC, need access to the ECS service via **NAT GW or VPC Endpoint**

Uses EBS or Instance Store for storing data

**Managed compute environment**:

- On-Demand and Spot launched in your VPC
- You specify the **Spot instance max price**
- Set **min and max vCPUs** -> EC2 are automatically created to respond to demand
- You have a **Batch Job Queue** that distributes the jobs
- SDK to send jobs to the **Batch Job Queue**
- You can set up autoscaling based on the jobs in the queue

**Unmanaged compute environment**: You control the EC2 config and scaling

Batch - **Multi Node Mode (HPC)**:

- Leverages multiple EC2 at the same time, representing a single job
- 1 main node and many children nodes
- **Does not work with Spot Instances**
- Works better with EC2 **placement group cluster**

### EMR

- EC2s are launched in a **single AZ in the VPC**, HDFS uses EBS volumes
- If you want your data multi-AZ, you can use EMRFS native integration with S3 (SSE)
- You can use **Hive to directly read data from DynamoDB**

EMR Instance configuration:

- Uniform instance groups: Select Master, Core and Task nodes instances On-Demand or Spot. **Has autoscaling**
- Instance Fleet: Select number of OnDemand and spot, **no autoscaling**

### Redshift

- 2 Modes: Provisioned and Serverless
- Has a SQL interface for running queries
- Columnar data storage (OLAP)
- PB scale
- Data is loaded from S3, Firehose, DynamoDB, DMS...
- Up to +100s nodes, each up to 16 TB
- Default: Nodes are single AZ. But for some types of Cluster you can set up multi-AZ (have failover capability in case of failure)
- Redshift enhanced VPC Routing: COPY/UNLOAD goes through the VPC
- RedShift is a provisioned service (not Serverless) -> good when you have a sustained usage, for sporadic queries use Athena

Snapshots, stored in S3, you can restore a **new Cluster**:

- **Automated snapshots**: Every **8 hours, every 5GB or on a schedule**. You specify a retention period.
- Manual snapshots: permanent retention until deletion
- Can be copied to another Region for DR or configure **automated CRR**

Copy an encrypted snapshot into another Region: You must allow Redshift to perform encryption ops in the destination Region (**snapshot copy grant**), Redshift uses KMS in the destination Region to provision the cluster

Redshift spectrum: Must have a Redshift cluster to start the query, query is submitted to 1000s of nodes, results aggregated by cluster Computes nodes

Redshift Workload Management (WLM):

- Prevent short-running queries from being bloqued by long-running ones
- You define multiple `Query Queues`
- 2 Flavors: `Automatic WLM` (resources managed by Redshift) and `Manual WLM`

Redshift Concurrency Scaling: Virtually unlimited concurrent queries and users. Concurrency scaling cluster scales nodes automatically, it is **managed by WLM**

### DocumentDB

**Provisioned Service**, not On-Demand. Very similar to Aurora.

Pay for what you use:

- On-Demand EC2
- Database I/O
- Database Storage (GB/month)
- Backup Storage e.g. S3 (GB/month)

### TimeStream

- Serverless
- 1000s faster and 1/10 cheaper than RDS
- Recent data kept in memory and historical data in cost-optimized storage
- Builtin time series analytics functions

Producers:

- Prometheus
- Telegraph
- Kinesis DS
- Flink
- IoT
- Lambda

Consumers:

- QuickSight
- Grafana
- SageMaker
- JDBC compatible

### Athena

Use big files, +128 MB

Federated Queries: Uses `Data Source Connectors` (lambda functions) to execute queries in **relational and non-reliational** DB: RDS, Aurora, On-premises DB, every JDBC compatible DB. Results can be stored in S3

### QuickSight

Enterprise edition: Possibility to setup Column-level security (CLS)

`SPICE` in-memory compute engine only works if **data is imported** to QuickSight: CSV files, TSV, JSON...

You create **users (standard edition)** and **groups (only enterprise edition)**.

- **Dashboard**: Read-Only snapshot of an **analysis**, preserves configuration params
- You can share the analysis or dashboard with users and groups. You must first **publish the Dashboard**. Then, users or groups have access to the underlying data.

QuickSight data sources: RDS, Aurora, **Redshift, Athena**, S3, OpenSearch, Timestream, **SaaS (Salesforce, Jira)**, teradata, on-premises DB compatible with JDBC

## Monitoring

### CW Metrics

- EC2 Standard: 5 min. Detailed Monitoring: 1 min.
- Custom metrics: standard resolution 1 min, high resolution 1 sec

### CW Alarm

Can trigger:

- EC2 Action: `Reboot`, `Stop`, `Terminate`, `Recover`
- Auto Scaling
- **SNS**

Can be intercepted by `EventBridge`

### CW Dashboards

- Can display `Metrics` and `Alarms`
- Can show metrics in **multiple Regions**.

### Synthetic Canaries

- Scripts in Node.js or Python, access to headless Google Chrome
- Can run once or on schedule
- Check availability and latency of APIs, URLs, endpoints
- Can trigger CW Alarms

Available **Blueprints**:

- HeartBeat Monitor - Load URL
- API Canary - R/W apis
- Broken Link Checker
- Visual Monitoring: Compares against screenshot
- Canary Recorder: Records actions in a web
- GUI Workflow builder: Verifies that a sequence of actions can be taken on the web.

### CW Logs

Destinations:

- S3 with SSE-S3 or SSE-KMS. NOT real time, data can take up to **12 hours** to be available for export. Manual trigger with `CreateExportTask` API
- Kinesis Data Streams
- Firehose
- Lambda

Logs `Insights`: **Query logs and add queries to CW Dashboards**

Subscription Filters Destinations:

- Trigger a **lambda function managed by AWS** to load data in **real-time** to OpenSearch. You can create your **custom lambda**.
- Integration with Firehose for **near real-time** load on S3
- Kinesis Data Streams -> Firehose, Flink, Lambda, EC2. Useful to centralized logs in multi-Account/Region setting by sending them to a single Stream in a centralized account

### CW Agent

Install the CW agent on EC2 with SSM, 2 options, **EC2 must have the SSM agent**:

- SSM Run Command, document: AWS-ConfigureAWSPackage, Name: AmazonCloudWatchAgent
- SSM State Manager, document: AWS-ConfigureAWSPackage, Name: AmazonCloudWatchAgent

Without SSM Agent:

- Install manually the CW agent and retrieve its config from Parameter Store

### EventBridge

You can access Event buses cross-Account/Region with Resource based policies: Useful to aggregate events in a central event bus

You can **archive events and replay them**

There is a **Schema Registry** which allows to generate code for your app based on the data format of the event received from the EventBridge.

3 types of event buses:

- Default
- 3rd party SaaS: DataDog...
- Custom event bus

## Deployment

### Beanstalk

- Great for **replatform** (make it cloud native) your app
- Resources have the X-ray agent installed

Runtimes supported:

- Go
- Java
- Java with tomcat
- .NET on windows server with IIS
- Node.js
- PHP
- Python
- Ruby
- Packer Builder
- Docker (single, multicontainer and preconfigured)

3 Architectures:

- Single instance, good for dev
- LB + ASG, good for prod
- SQS + ASG only, good for non web apps (worker tier)

Worker environment: You can define periodic tasks in a `cron.yaml` file

Blue/Green deployment (not a native feature of Beanstalk): You create an v2 enviroment and use **Route 53 weighted routing policy** or **Beanstalk Swap URLs feature**

### CodeDeploy

Targets: EC2, ASG, ECS and Lambda

EC2: `appspec.yaml` + deployment strategy, **in-place deployment**, you can use hooks after each phase

ASG: **in-place update** or **blue/green where a new ASG is created, must use an ELB which serves both ASGs at the same time until you CodeDeploy terminates the old one**

Lambda: Pre and Post **traffic hooks**, traffic shifting, SAM natively uses CodeDeploy, can rollback to v1 with CW alarm

ECS:

- Blue/Green launches a **new task set** from task definition v2
- defined in the **ECS Service Console**
- supports **Canary deployments** `Canary10Percent5minutes`

### CloudFormation

Deletion Policy, default to `Delete` EXCEPT for `AWS::RDS::DBCluster` which is `Snapshot`

Custom Resources (authored Lambda function):

- AWS resource that is not yet supported
- On-premises resource
- Emptying an S3 bucket before deletion
- Fetch an AMI Id
- ...

Stacksets: **Automatic Deployment** feature to deploy to all trusted accounts

Resource import, you need:

- A template that describes the final stack (original stack resources + target resource to import)
- A unique identifier for each target resource e.g. an S3 bucket name
- Each target resource must have a `DeletionPolicy`

### SSM

Patch Manager:

- Define a patch baseline
- Define patch groups
- Define manteinance windows
- Add `AWS-RunPatchBaseline` (works on **Linux and Windows**) Run Command as part of the registered tasks in the manteinance window
- Define Rate Control (concurrency and error threshold) for the task
- Monitor patch compliance with **SSM Inventory**

Session Manager: All commands run on the EC2 are logged on CW logs and `StartSession` events are captured by CloudTrail

OpsCenter: Resolve Issues related to AWS resources, OpsItems created by EventBridge or CW Alarms, provides Runbooks to resolve them.

## Cost Control

Cost Allocation Tags: **Only appear in the billing console**. Take up to 12 hours to appear. Show up as new columns in the cost report.

AWS Generated Cost Allocation Tags: `aws` prefix e.g. `aws:createdBy`. Not applied to resources created before activation

User tags: `user` prefix

### Trusted Advisor

- Security
- Cost Optimization
- Service Limits (can increase some of them)
- Operational excellence

**Can enable weekly email notification from the console**

Good to know:

- Trusted Advisor can check if any S3 bucket if public, but it **cannot check if any object is public**
- Only service that can **monitor service limits**, but to change them use `AWS Support Center` or `Service Quotas`

Support plans:

- Basic, free and **7 core checks**
- Developer, **7 core checks**
- Business, **all core checks**
- Enterprise, **all core checks**

**Full trusted Advisor** (Only available for **business and enterprise plans**): Ability to wire up CW Alarms and Access to AWS Support API.

### Service Quotas

Can notify when a quota threshold is broken and trigger a CW Alarm e.g. Lambda concurrent execs

### Savings plans

1-3 years commitment e.g. 10$ hour

**EC2 Instance Savings plan**: Same discount as Reserved Instances (72%). **Region Locked**, and **instance family locked** e.g. M5, flexible across instance type, OS and tenancy.

**Compute Savings Plan**: Same discount as Convertible Reserved Instances (66%). Flexible across instance family e.g. move from M5 to C5, **Region**, **compute type (EC2, fargate, lambda)**, OS and tenancy.

### S3 Requester pays

- You need to setup a bucket policy mandating IAM auth for the requester.
- The bucket owner still pays storage cost
- **If an IAM role is assumed, the owner account of that role pays**

### Budgets

- Up to 5 SNS per Budget Alarm
- Budget actions, triggered when a budget exceeds a cost or usage threshold. **3 actions**, can be executed automatically or require **manual approval**:
  - Aply IAM policy to user, group or role
  - Aply SCP to an OU
  - Stop EC2 or RDS
- We can use SNS to trigger a lambda function that applies an SCP to the member account or **move the account to a more restrictive OU**

### Cost Explorer

- Useful to choose optimal **Savings Plan**
- Feature to forecast usage up to 12 months based on past usage

### Compute optimizer

Uses ML to **analyze resource config and CW metrics**, recommendations can be exported to S3

- EC2 instances
- EC2 AutoScaling
- Lambda
- EBS

### Reserved instances

You can renew RIs by **queuying them**: Purchase RI whenever the previous one expires and will renew automatically.

## Migration

- Relocate: A special case is Relocate using `VMware Cloud on AWS`, where you literally move your entire VMware software-defined data center (vSphere hosts, vCenter, NSX, vSAN) to run on AWS infrastructure. Your VMs run on AWS hardware but are managed with the exact same VMware tools you use on-premises. The primary service for this is AWS `Application Migration Service (MGN)`, which continuously replicates source servers (physical, virtual, or cloud) into ready-to-launch AMIs in AWS.
- Rehost: Don't use cloud optimizations, app migrated as is. **Application Migration Service**
- Replatform: database to RDS e.g. on-premise PostgreSQL to RDS Postgres, app to Beanstalk. Leverage fully managed, serverless, cloud optimizations
- Repurchase: Use SaaS. SalesForce, Workday etc

### Storage GW

S3 File GW: NFS or SMB

- Most Recently used data is cached
- Supports **Standard**, **Standard IA**, **One-Zone IA** and **Intelligent Tiering**
- Transition to Glacier with Lifecycle policy
- Bucket access with IAM Role for File GW
- `SMB` proto has **integration with Active Directory for user auth**

Volume GW: iSCSI, backed by S3

- Backed with EBS snapshots
- `Cached GW`: Caches most recent data
- `Stored Volume`: entire dataset on premise, scheduled backups to S3

Advanced Architectures:

- You can connect an extra File GW appliance deployed in a VPC to the same S3 bucket, both on-premises and EC2 share data in S3
- Deploy a Read Only replica of File GW appliance in a VPC to give EC2 read-only access with **low latency**
- If you enable **Object Versioning** you have file versions and restore a previous version of a File, then you must use `RefreshCache` API on File GW to notify any S3 changes
- Enable **S3 Object Lock**, File GW creates new version of the object without affecting priori versions -> Compliance and auditability

### Snowball

- Storage Optimized, **210TB**
- Compute Optimized, **28TB**, run EC2 instances and lambda functions at the edge

Improving data transfer rates:

- Perform multiple writes at one time
- Transfer small files in batches
- Data transfer rate is 25-40 MB/s -> Use the `Amazon S3 adapter for Snowball`, 250-400 MB/s

### DMS

You need to create an EC2 instance running DMS software.

`Redshift`, `Kinesis Data Streams` and `OpenSearch` are **targets only**.

Good things to know:

- Works over VPC-peering, DX and VPN
- Full load -> 1 time
- Full load + CDC -> Continuous data replication
- CDC only -> just stream changes
- Oracle
  - Source: Supports TDE using `BinaryReader`
  - Target: Supports BLOBs in tables that have a primary key and TDE. You need to make sure that there is pk associated with the BLOB
- Opensearch
  - Target: Possible to migrate from RDS e.g. RDS MySQL

Snowball + DMS:

1. Use Schema Conversion Tool to extract data locally and move it to Snowball
2. Ship snowball
3. Snowball automatically loads data into S3
4. DMS starts replication to the target

### Cloud Adoption Readiness Tool CART

- Transform your idea of moving to the cloud into a detailed plan that follows best practices
- Works by answering a set of questions across 6 perspectives
- Generates a report

### Application Discovery Services

Plan migrations by gathering on-premises info, service utilization and dependency mapping.

- Agentless Discovery (Application Discovery Agentless Connector)
  - Open Virtual Appliace (OVA) that you need to deploy on your VM host on-premise
  - Performs VM inventory, configuration and performance history (CPU, memory and disk usage)
  - OS agnostic
- Agent Based discovery, get a full map of how servers are communicated
  - System config, system performance, **running processes** and **detailed network connections between systems**
  - Microsoft Server, Amazon Linux, Ubuntu, SUSE, Redhat...

Resulting data can be exported to CSV or `AWS Migration Hub` -> data is stored in `S3` at regular intervals, you can query it with `Athena`

### Application Migration Service (MGN)

- Look for keyword **migrate**
- Migrates entire VM to AWS
- Evolution of `CloudEndure Migration` & `AWS Server Migration Service`
- Run the **replication agent on-premises** which **continuosly replicates** your workloads to a staging environment on AWS, when you are ready to fully migrate you **cutover**
- Minimal downtime

### AWS Elastic Disaster Recovery (DRS)

- Used to be named `CloudEndure Disaster Recovery`
- Look for keyword **recover**
- Run the **replication agent on-premises** which continuous block replicates to the staging environment

### AWS Migration Evaluator

- Build data driven business case for migration to AWS
- You install `Agentless Collector` on-premises

### AWS Backup

- Supports **Cross-Region/Account backups**
- You create Backup policies known as **Backup plans**
  - Backup frequency
  - Backup window
  - Transition policy to cold storage
  - Retention policy period
- AWS **Backup Vault Lock** -> WORM, even the root user cannot delete the backups, neither alter its retention period

## VPC

### VPC Peering

- VPC peering can work **Cross-region/account**
- `SGs` from **peered VPCs** can be referenced
- Route tables -> CIDR with **longest prefix match** wins e.g. `10.0.0.77/32 to A` wins over `10.0.0.0/16 to B`
- If **ANY** of the CIDRs in a VPC overlap, Peering does not work
- A VPC can have **both IPv4 and IPv6 CIDRs**
- **No edge to edge routing** applies to VPN, DX, IGW, NATGW and GW endpoints

### Transit GW

- Regional Service but **you can peer together Transit GW deployed in different regions**
- Share TransitGW cross-account with `AWS RAM`
- In the route table you can limit which VPC can talk to with other VPC
- Supports `IP Multicast`, no other services support it
- Instances in a VPC can access a `NAT GW`, `NLB`, `PrivateLink` and `EFS` in other VPCs attached to the Transit GW
- You can create an **egress VPC** to give internet access to another **private VPC**. Deploy an **ENI in a private subnet of the egress VPC** and link it to the Transit GW, update the route table of the TransitGW to route traffic from the private VPC to this ENI in the egress VPC
- You can share Transit GW through RAM
- You can create multiple **route tables** e.g. prod, dev. each route table segregates the VPCs (TG attachments) that can route to each other
- You can connect a `DX connect GW` to multiple `TransitGW` in different regions
- TransitGW allows you to create **multiple VPN connections that can be aggregated** for more capacity
- You are **billed hourly** for each peering attachment, no data processing charges

### VPC Endpoints

VPC Endpoints interface:

- Enable `DNS hostnames` and `DNS Support` to use AWS DNS names in the route table e.g. `athena.us-east-1.amazonaws.com`
- Can be accessed from `DX`, `VPN`, peered VPCs
- Attach SG to the ENI
- Corporate Data Center via DX can only access S3 **privately** using **Private VIF + Privatelink VPC Endpoints Interface**
- A VPC in another region that has a **peering connection** can use VPC Endpoint interface to access S3

VPC Endpoints GW

- **only 1 per VPC**
- `DNS resolution` must be enabled in the VPC
- Only work to connect resources deployed in the VPC (No peering, DX or VPN)

`aws:sourceVpce` targets a single vpc endpoint, `aws:sourceVpc` targets all endpoints in a VPC, `aws:SourceIp` just works for **public IPs**

### PrivateLink (VPC Endpoint Services)

- Most secure and scalable way of expose a service to 1000s (other accounts)
- Does not need VPC peering, IG, NAT, route tables...
- NLB in the Service VPC and ENI in the Customer VPC

### VPN (AWS managed VPN)

On-premise side:

- Setup software or hardware appliance in the corporate network
- on-premise VPN should be accessible using public IP

AWS side:

- Setup VGW which is a VPC level resource and attach it to the VPC
- Setup Customer GW to point to VPN appliance

2 VPN connections are created for redundancy

You can use Global Accelerator

You need to update the route table of the VPC subnets to point to the VGW and the route table on-premises to point to CGW

**Border GW protocol**: Just need to specify the ASN (Autonomous System Number) of the VGW and CGW

Site2Site VPN or DX and Internet Access:

- On-premises cannot access the internet with NAT GW (Not allowed) + IGW. If you replace the NAT GW for a **self managed NAT Instance** then it's **OK**, because you manage the software
- Other way: Instances in a private subnet CAN access the internet if you have an on-premise NAT. You don't need NATGW/

`VPN CloudHub`: 1 VPC to multiple Data Centers. Can connect up to 10 CGW for each VGW, enabling communication across all the parts (hub and spoke)

**Multiple VPC to a single Data Center**:

- AWS recommends a separate VPN connection for each VPC, very complicated
- Consider using Direct Connect GW
- Or use a single `shared services VPC connected to on-premises` + `VPC Peering` and replicate the services from the corporate DC to the shared VPC or use a proxy to forward requests on-premises

### Client VPN

Connect from your computer using OpenVPN to your private network AWS and on-premises, uses a private ip

- e.g. 2 VPCs peered and you deploy a Client VPN ENI in VPC A, you can connect and access VPC B through the peering connection
- Same scenario but VPC A is connected to on-premises with S2S VPN, you can connect to VPC A and access on-premises
- If you deploy Client VPN ENI in a public subnet, you have internet access

So the client VPN is deployed in a subnet and all the VPC connections work for the client VPN e.g. VPC peering, DX, VPN, Transit GW...

### Direct Connect

- Public VIF: Connect to public AWS endpoints
- Private VIF: Connect to resources inside a VPC (VPC Interface Endpoints, ALB, EC2...)
- Transit VIF: Connect to resources in a VPC through TransitGW

From the customer router you connect to a DX Location (AWS Direct Connect Endpoint)

From DX Location you connect it to the VGW (private VIF) or public resources (public VIF)

- **Dedicated connections**: You make the request directly to AWS
- **Hosted Connections**: You make the request to AWS DX partners (50 Mbps, 100 Mbps up to 10 Gbps)
  - Capacity can be **added or removed on demand**

VPN over DX uses **Public VIF**

**Link Aggregation Groups (LAG)**:

- Gain speed and failover by summing existing **CGW to DX Connections** into a **logical one**
- Can aggregate up to 4 connections, can be added over time
- All connections in the LAG must be **dedicated** and have the **same bandwidth** and must terminate at the **same Direct Connect Endpoint**
- You can specify a min number of connections for the LAG to function

DX Gateway:

- Connect DX to multiple VPC in multiple regions/accounts
- **Private VIF**
- Only way to connect TransitGW to DX

DX GW **SiteLink**:

- Allows to send data from one DX Location to another **bypassing AWS Regions**
- Data is sent over the fastest path
- If you have 2 Data Centers, each connected to a different DX Location, both DX Locations connected to a DXGW. Connectivity between the 2 data centers is stablished

### VPC Flow Logs

Destinations: S3, CW Logs, Firehose

Can also capture info of AWS Managed interfaces: ELB, RDS, ElastiCache, Redshift, Workspaces, Transit GW

For **streaming** analysis, `CW Logs Insights`

Inbound traffic for NATGW -> if traffic is permitted by SG and NACL is possible to have inbound ACCEPT, check it by running the query of the video in CW Logs to see if the private IP of NATGW appears

### Network Firewall

- Protect a VPC from layer 3 to layer 7
- Internally it uses GW LB but all managed by AWS
- Rules can be centrally managed with `Firewall Manager` to apply them to multiple VPCs
- Allow, Drop or Alert actions for the traffic that matches the rules
- Rule matches can be sent to S3, CW Logs, Firehose

## Other Services

### CI/CD

`CodeCommit` can trigger a `Lambda` function and scan for **leaked credentials**.

`CodeBuild` can build a Docker image and push it to `ECR`

`CodePipeline` **GitHub integration**: 

- **Version 1**: Enable **GitHub webhooks** that send notifications to CodePipeline whenever the repo is updated.
- **Version 2 (NEW!)**: **CodeStar Source Connection**, through this connection any changes pushed to Github will be picked up by `CodePipeline`

`CodeGuru` (ML powered):

- CodeGuru Reviewer: Automated code static analysis
- CodeGuru Profiler: Recommendations about application runtime performance. Supports apps running on AWS and **on-premises**

### Alexa for Business, Lex & Connect

**Alexa for Business**: increase the utilization of **meeting rooms** in the workplace

**Lex**: Same tech as Alexa, speech to text and NLP recognize intent of text. Build chatbot

**Connect**: You give a phone number and receive the calls then connect the calls to contact flows, virtual contact center. You can connect the call to `Lex` to recognize the intent of the call and trigger a lambda.

### Kinesis Video Streams

- **1 video stream per streaming device**, Kinesis Video Streams Producer library.
- Underlying data is stored in `S3` but we have **no access to it**. If you want access to stream data you need to build a **custom solution** (the **exam can test you on this**).
- Consumers
  - EC2 instances for real time, **Kinesis Video Streams Parser library**
  - Direct integration with `AWS Rekognition` for facial detection, **Face Collection**

### WorkSpaces

- **Linux** and Windows
- Pricing per Hour (on-demand) or monthly subscription
- Integration with Microsoft AD

**Workspace application management (WAM)**:

- Containerize your app and deploy it to all your WorkSpaces
- Keep Apps updated with WAM, just the apps (**not the OS**)

**Windows Update**:

- Windows WorkSpaces has windows update **activated by default**
- By default, WorkSpaces is configured to install software updates
- Updates are installed during **maintenance windows** (you define them)
  - **Always On Workspaces**: WorkSpaces always running, updates installed by default **00:00 - 04:00 Sunday**
  - **AutoStop Workspaces**: Automatically starts the workspaces **once a month** to install updates
  - **Manual maintenance window**: you define your windows and perform maintenance. During the window the instances are off for your users.

Multi-Region Architecture:

- Region A (primary) has workspaces integrated with `AWS Managed Microsoft AD`
- Region B (failover) has workspaces integrated with `AD Connector (proxy)`. **Multi-Region AWS Managed Microsoft AD is not supported**.
- Create **connection aliases** in both regions
- In `Route 53`, you create a **TXT Record** (e.g. desktop.example.com) of **failover type** pointing to connection aliases.
- The failover region will not have access to user data in the primary region -> Use `Amazon WorkDocs` for persist user data cross-region

Amazon WorkSpaces IP Access Control Groups -> Like **Security Groups**

### AppStream 2.0

- Desktop **Apps** delivered within a Web Browser, **e.g. Blender**
- Each app can be resource configured (CPU, RAM, GPU)

### AppFarm

- Test an app, mobile and web (e.g. a one published on google store) across many **real** browsers and devices
- Automatically generates videos and logs to document the issues encountered
- Can remotely log-in to devices for debugging

### Macie

- Fully managed, one click to enabled it in the S3 buckets you want
- Notifies discoveries through `EventBridge`

### SES

**Configuration Sets** -> Help customize and analyze your email **send events**. 2 types:

- Event Destinations
  - Firehose: receives metrics (number of sends, deliveries, opens, clicks, complaints) for each email, you can run **analytics** with Flink or S3 + Athena
  - SNS: Immediate feedback on bounce and **complaint information**
- IP pool management: use IP pools to send particular types of emails e.g. 1 IP set for **transactional emails** and 1 IP set for **marketing emails**

### PinPoint

2-way (inbound/outbound) marketing communication service

Can create segment types, **personalize messages** and delivery schedules

Possibility to receive replies

Integrations for stream events (**TEXT_SUCCESS**, **TEXT_DELIVERED**...): SNS, Firehose, CW logs

### EC2 Image Builder

- Automate the creation, mantain, validate and test **EC2 AMIs** or **Container Images**
- Can publish to multiple accounts and regions
- How it works:
    1. The image builder service creates a **Builder EC2 Instace** based on your builder config
    2. Then it creates an AMI from this Builder EC2 Instace
    3. Creates an EC2 from the AMI
    4. Runs a test suit against the EC2
    5. Distributes the AMI in the regions/accounts you have defined

Architecture: `Cloudformation` triggers EC2 Image Builder to build the AMI and then triggers a rolling update to change the AMI of the running instances

### IoT

**Topics** (e.g. receive MQTT message), **rules** and **actions** (integrations with other Services). Important integration is `Firehose`