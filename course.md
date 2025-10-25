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
