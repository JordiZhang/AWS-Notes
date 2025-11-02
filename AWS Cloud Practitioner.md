--- 
Account ID: 508479635541
IAM Username: Jordi
Password: 6
# Cloud Computing
Types of Cloud Computing:
- `Infrastructure as a Service (IaaS)`: Provides hardware for user to do as they wish.
- `Platform as a Service (PaaS)`: Provides already setup machines for users to run applications on.
- `Software as a Service (SaaS)`: Provides ready made software, think Gmail.

# Identity and Access Manager (IAM)
--- 
## IAM Policies
Consists of:
- `Version`: policy language version
- `Id`: an identifier for the policy (optional)
- `Statement`: one or more individual statements (required)
Statement consists of:
- `Sid`: identifier for the statement (optional)
- `Effect`: Allow/Deny
- `Principal`: the account/user/role
- `Action`: list of actions the policy allows or denies
- `Resource`: list of resources to which actions can be applied to
- `Condition`: conditions for when the policy is in effect (optional)

# Elastic Cloud Compute (EC2)
--- 
## Instance Types
AWS has following name convention: 
	m5.2xlarge
	`m`: instance class
	`5`: generation
	`2xlarge`: size within the instance class

- `General Purpose`: Good for diverse workloads such as webservers or code repositories. 
- `Compute Optimized`: Good for compute-intensive tasks that require high performance processors.
- `Memory Optimized`: Fast performance for workloads that process large data sets in memory.
- `Storage Optimized`: Good for storage-intensive tasks, sequential read and write access to large data sets on local storage.
## Security Groups
They control how traffic is allowed into or out of the EC2 Instance.
They only contain allow rules. Rules can reference by IP or by security group.
They regulate:
- Access to ports
- Authorised IP ranges IPv4 and IPv6
- Control of inbound network
- Control of outbound network
Ports to know:
- `22` - SSH (Secure Shell)
- `21` - FTP (File Transfer Protocol)
- `22` - SFTP (Secure File Transfer Protocol), upload files using SSH
- `80` - HTTP, access unsecured websites
- `443` - HTTPS, access secured websites
- `3389` - RDP (Remote Desktop Protocol), log into Windows instance
Credentials for an EC2 Instance should be provided via IAM roles.
## EC2 Pricing
EC2 on demand:
- Linux or Windows: per second after the first minute
- Other OS: per hour
Reserved Instances: 
- Large discounts compared to on demand
- You reserve a specific instance attributes
- Reservation periods of 1 or 3 years
- No upfront, partial upfront and full upfront payment options
- Reserved Instance's Scope, regional or zone
- Can buy and sell
- Also exists convertible reserved instance where you can change the instance type, family, OS, scope and tenancy
Savings Plans
- Discounts based on time
- Commit to a certain type of usage 
- Locked to specific type of instance and AWS region
- Flexible across instance size, OS and tenancy
Spot Instances
- Cheapest
- Can lose instance at any point if max price is less than current spot price
- Suitable for workloads resilient to failure
Dedicated Hosts
- Physical server with EC2 instance capacity fully dedicated
- Allows addressing of compliance requirements and use existing server bound software licenses.
	- On demand pricing
	- Reserved, 1 or 3 years
- Most Expensive
EC2 Dedicated Instance
- Dedicated instance to your account
Capacity Reservations
- Reserve on demand instances capacity in a specific availability zone for any duration
- Always have access to EC2 when needed
- No time commitment nor billing discounts, for discounts combine with reserved or savings plans
- You are charged at On demand rate whether you run instances or not
# EC2 Instance Storage
--- 
## Elastic Block Store (EBS)
- Its a network drive (not physical drive)
	- It uses the network to communicate, so there may be a bit of latency
	- Can be detached from 1 EC2 to another
- Locked to an availability zone
	- To move an EBS to a different AZ we need to do a snapshot first
- Have a provisional capacity (GBs, IOPS)
	- Billed for all provisional capacity
- Can take snapshots as backups, priced incrementally, only billed for the changed blocks stored
## EC2 Instance Store
If high performance disk is needed, i.e. physical, use EC2 Instance Store.
- But they are ephemeral, lose their storage if they're stopped
## Elastic File System (EFS)
- Managed network file system that can be mounted on 100s of EC2 instances.
- Works only on Linux EC2 instances in multi availability zone
- Highly available and scalable
- Expensive, pay per use and no capacity planning
	- EFS Infrequent Access, storage class optimized for infrequently accessed files
	- Pay a fee for any read/write on this class
	- Much cheaper than standard EFS
	- EFS will move files automatically to IA based on last time they were accessed, only if Lifecycle Policy is turned on
## Elastic Load Balancer (ELB)
- `Application Load Balancer`: HTTP/HTTPS only, layer 7
- `Network Load Balancer`: Ultra high performance, allows for TCP, layer 4
- `Gateway Load Balancer`: layer 3
![[Load Balancers.png]]
## Auto Scaling Group (ASG)
The goal of an ASG is:
- Scale out (add EC2 Instances) to match increased load
- Scale in (remove EC2 Instances) to match reduced load
- Ensures we have a minimum or maximum number of Instances running
- Automatically register new instances to a load balancer
- Replace unhealthy instances
- Saves tons of money
# Amazon S3
--- 
Amazon S3 allows users to store objects (files) in 'buckets' (directories). In particular, Amazon S3 advertises itself as infinitely scalable storage. These buckets must have a globally unique name, across all regions and accounts. However, buckets are region defined.
Naming Convention: 
- No uppercase
- No underscores
- 3-63 characters long
- Not an IP
- Start with lowercase letter or number
- Must not start with prefix xn--
- Must not end with suffix -s3alias
## Objects
Objects have a key, which is their full path in the bucket. There are no directories in Amazon s3, keys are just long names with slashes. Object values are the contents of the body:
- Max size is 5TB
- If uploading more than 5GB, use multi-part upload
Objects also have Metadata, Tags (for security/lifecycle) and Version ID (if versioning is enabled).
## Security
User based:
- IAM policies, which API calls are allowed for which IAM user
Resource based:
- Bucket Policies, bucket wide rules from the S3 console, allows cross account
- Object Access Control List (ACL), finer grain and can be disabled
- Bucket Access Control List (ACL), less common and can be disabled
An IAM principle can access the bucket if the user permissions allow it or the resource policy allows it, AND there is no explicit deny.
Encryption using encryption keys

The most common way of implementing security is using Bucket Policies:
JSON based policies
- Resources: buckets and objects
- Effect: Allow or Deny
- Actions: Set of API to apply effect
- Principal: The account or user to apply policy to
```JSON
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicRead",
			"Effect": "Allow",
			"Principal": "*",
			"Action": [
				"s3:GetObject"
			],
			"Resource": [
				"arn:aws:s3:::examplebucket/*"
			]
		}
	]
}
```
## Storage Classes
- `General Purpose`: Used for frequency accessed data, low latency and high throughput.
- `Infrequent Access`: For data that is less frequently accessed, but requires rapid access when needed.
	- Standard Infrequent Access
	- One Zone Infrequent Access
- `Glacier`: Low cost storage meant for archiving and backups
	- Glacier Instant Retrieval, Millisecond retrieval, minimum storage duration of 90 days
	- Glacier Flexible Retrieval, Expedited (1-5 minutes), Standard (3-5 hours), Bulk (5-12 hours), minimum storage duration of 90 days
	- Glacier Deep Archive, Standard (12 hours), Bulk (48 hours), minimum storage duration of 180 days
- `S3 Express One Zone`: High performance, single AZ class. Objects are stored in a Directory Bucket (bucket in a single AZ)
- `S3 Intelligent-Tiering`: Moves objects automatically between Access Tiers based on usage, with no retrieval charges.

| Tier                               | Min Duration (days) |
| ---------------------------------- | ------------------- |
| Frequent Access (automatic)        | 0                   |
| Infrequent Access (automatic)      | 30                  |
| Archive Instant Access (automatic) | 90                  |
| Archive Access (optional)          | 90-700+             |
| Deep Archive Access (optional)     | 180-700+            |
## Data Migrations
![[Transfer Times.png]]
Use AWS Snowball devices to perform data migrations that take very long, say more than a week. 
# Databases and Analytics
--- 
## Managed Databases
`RDS (Relational Database Service)` allows you to create databases in the cloud that are managed by AWS, using whatever SQL dialect needed. Can Reserve for cost optimization.
`AWS Aurora` is a proprietary technology from AWS which supports PostgreSQL and MySQL. It is a cloud optimized service so it is faster than using MySQL on RDS. Aurora storage automatically grows in increments of 10GB up to 128TB. It is more expensive than RDS but more efficient.
`AWS Aurora Serverless Database`, automated database instantiation and auto-scaling based on usage. No capacity planning needed, least management overhead, pay per second.  
## RDS Deployment
`Read Replicas`: 
- Scale the read workload of your DB
- Can create up to 15 Read Replicas
- Data is only written to main DB
`Multi AZ`:
- Failover in case of AZ outage
- Data is only read/written to the main database
- Can only have 1 Failover DB
`Multi Region Read Replicas`:
- Disaster recovery in case of region issue
- Local performance for global reads
- Replication cost
![[Multi Region Read Replicas.png]]
`ElastiCache` is essentially database 'RAM'. It is managed Redis or Memcached. It is in-memory databases with high performance that helps reduce load off databases for read intensive workloads.
## DynamoDB
- Fully managed highly available with replication across 3 AZ 
- It is a NoSQL database
- Scales to massive workloads, distributed "serverless" database
- Millions of requests per second, trillions of rows, 100s of TBs of storage
- Fast and consistent in performance
- Single digit millisecond latency
- Integrated with IAM 
- Low cost and auto scaling capabilities
- Standard & Infrequent Access Table Class
- Can reserve for cost optimization
DynamoDB is a key/value database, essentially dictionaries or hash tables. DynamoDB also has an option for in-memory cache, which is DynamoDB Accelerator or DAX. It has microsecond latency and is specialized for DynamoDB. 
## Redshift
- Based on PostgreSQL, but its not used for OLTP (Online Transaction Processing)
- It is meant for Online Analytical Processing (OLAP), so analytics and data warehousing
- Loads data once every hour, not every second
- 10x better performance than other data warehouses?
- Columnar storage of data, instead of row based
- Massively Parallel Query Execution (MPP), highly available
- Pay as you go
- Has a SQL interface for querying
- Integrated with BI tools such as AWS Quicksight or Tableau
## Elastic MapReduce (EMR)
- Not actually a database, helps creating Hadoop clusters (Big Data) to analyze and process vast amounts of data
- Clusters can be made of hundreds of EC2 Instances
- Supports Apache Spark, HBase, Presto, Flink...
- EMR takes care of all provisioning and configuration
- Auto scaling and integrated with Spot Instances
## Other Databases
`AWS Athena`:
- Serverless query service to perform analytics against S3 objects
- Uses standard SQL
- Supports CSV, JSON, ORC, Avro and Parquet
`AWS Quicksight`:
- Serverless machine learning powered BI service for interactive dashboards
`DocumentDB`:
- AWS implementation of the NoSQL database MongoDB, similar to Aurora being the AWS implementation of PostgreSQL and MySQL.
`AWS Neptune`: 
- Fully managed graph database
- Highly available across 3 AZ, with up to 15 read replicas
- Build and run applications working with highly connected datasets, optimized for complex and hard queries 
- Great for knowledge graphs (Wikipedia), fraud detection, recommendation engines, social networking
`AWS Timestream`:
- Serverless time series database
`AWS Managed Blockchain`:
- Essentially Crypto blockchain ledgers. 
- Managed service to join public blockchain networks or create your own scalable private network
- Compatible with the frameworks Hyperledger Fabric or Etherium
`AWS Glue`:
- Managed ETL serverless service
`Database Migration Service (DMS)`:
- Quick and secure migration service
- Source database remains available during the migration
- Supports:
	- Homogenous migration: Oracle -> Oracle
	- Heterogeneous migration: Microsoft SQL server -> Aurora
# Other Compute Services
--- 
## Docker Containers
`Elastic Container Service (ECS)`:
- Launch Docker containers on AWS
- You must provision and maintain the infrastructure, i.e. EC2 instances
- AWS takes care of starting/stopping containers
- Has integrations with the Application load balancer
`Fargate`:
- Also for launching Docker containers on AWS
- No need to provision infrastructure, so simpler
- Serverless
- Runs containers for you based on CPU/RAM
`Elastic Container Registry (ECR)`:
- Private Docker Registry on AWS
- Store your Docker images so they can be run by ECS or Fargate
## Kubernetes
`Elastic Kubernetes Service`:
- Allows you to launch managed Kubernetes clusters on AWS
- Kubernetes is an open source system for management, deployment and scaling of containerized apps (Docker)
- Containers can be hosted on EC2 instances, Fargate
- Kubernetes is cloud agnostic (can be used on any cloud)
## Lambda (Serverless Functions)
New paradigm in which developers don't manage servers.
- Virtual functions
- Limited by time, so short executions
- Run on demand
- Scaling is automated
- Pricing is per request and compute time
- Event Driven: functions get invoked by AWS when needed, so Lambda is a reactive service
- We can use an API gateway, which is a fully managed service to easily create, publish, maintain, monitor and secure APIs, to Proxy requests with Lambda
## AWS Batch
- Fully Managed batch processing at any scale
- Efficiently run 100,000s of computing batch jobs
- A batch job is a job with a start and an end
- Batch will dynamically launch EC2 Instances or Spot Instances 
- Batch provisions the right amount of compute/memory
- Batch Jobs are defined as Docker images and run on ECS
## Lambda vs Batch
`Lambda`:
- Time limit
- Limited runtimes
- Limited temporary disk space
- Serverless
`Batch`:
- No time limit
- Any runtime as long as its packaged as a Docker image
- Rely on EBS/Instance store for disk space
- Relies on EC2
## AWS Lightsail
- Standalone service, cloud computing for dummies
- Virtual servers, storage, databases and networking
- Low pricing
- Great for people with little cloud experience
- High availability but no auto-scaling, limited AWS integrations
# Deployment and Infrastructure Management
--- 
`CloudFormation`:
- Declarative way of outlining AWS infrastructure
- Infrastructure as code, no need to manually create resources
- Using Infrastructure Composer, we can visualize the cloud formation template (visualization of the cloud structure)
`AWS Cloud Development Kit (CDK)`:
- Define cloud infrastructure using a familiar programming language of your choice
- The code is compiled into a CloudFormation Template
- Can deploy infrastructure and application runtime together
`Elastic Beanstalk`:
- Developer centric view of deploying applications
- Implements Elastic Load Balancer into EC2 instances in an Auto Scaling Group which feeds data into an RDS and optionally an ElastiCache
- Three architecture models:
	- Single Instance deployment
	- Load balancer + auto scaling groups
	- Auto scaling groups only.
- Health monitoring pushes metrics to CloudWatch
`AWS CodeDeploy`:
- Works with EC2 instances
- Used to update applications from 1 version to the next
- Also works with On premise servers
`AWS CodeCommit`:
- Discontinued
- Any service which had CodeCommit integration has Github integration
`AWS CodeBuild`:
- Code building service
- Compiles source code, run tests and produces packages that are ready to be deployed
- Serverless
`AWS CodePipeline`:
- Orchestrate the different steps to have code automatically pushed to production
`AWS CodeArtifact`:
- Artifact Management service, i.e. Code dependencies
- Can retrieve code dependencies straight from CodeArtifact
## AWS Systems Manager (SSM)
- Manage EC2 Instances and on premise systems at scale
- Patching automation for enhanced compliance
- Run commands across an entire fleet of servers
- Store parameter configuration with the SSM Parameter Store
`AWS SSM Session Manager`:
- Allows you to start secure shell on your EC2 and on premise servers
- No SSH access, bastion hosts or SSH keys needed
- No port 22 needed
- Sends session log data to S3 or CloudWatch logs
`AWS SSM Parameter Store`:
- Secure storage for configuration and secrets
- Serverless
- Versioning and encryption
- Control access permissions using IAM
# Global Infrastructure
--- 
`AWS Route 53`:
- Managed Domain Name System (DNS)
- DNS is a collection of rules and records which helps clients understand how to reach a server through URLs
- Most common records are:
	- URL -> IPv4 == A record
	- URL -> IPv6 == AAAA record
	- URL -> URL == CNAME
	- URL -> AWS Resource == Alias
- Routing Policies:
	- Simple Routing Policy, No health checks
	- Weighted Routing Policy, Health checks and routes traffic to instances based on weights
	- Latency Routing Policy, Health checks and routes traffic based on location to lowest latency server
	- Failover Routing Policy, Health checks on primary, Routes to failover based on health of primary instance
`CloudFront`:
- Content Delivery Network (CDN), main use is caching
- Improves read performance, content is cached at the edge 
- Hundreds of points of presence globally
- Great for static content that must be available everywhere, S3 Cross Region replication should be used for dynamic content instead
- Origins:
	- S3 bucket, distributing and caching at the edge. Secured using Origin Access Control (OAC)
	- VPC, for applications hosted in VPC private subnets. Application load balancer, network load balancer, EC2 instances
	- Custom Origin (HTTP), S3 website or any public HTTP backend.
`S3 Transfer Acceleration`:
- Increase Transfer speed by transferring file to an AWS edge location which then forwards the data to the S3 bucket in the target region
`AWS Global Accelerator`:
- No Caching, proxying packets at the edge to applications running in AWS
- Improve global application availability and performance using AWS global network
- Leverage AWS internal network to optimize the route to your application
`AWS Outposts`:
- Essentially server racks that companies can buy from Amazon to have AWS compatible on premise servers
- They are managed and setup up in your on premise site by AWS, sort of interfaces between on premise and cloud
- Used for Hybrid cloud users
`AWS Wavelength`:
- Wavelength zones are infrastructure deployments embedded within the infrastructure of a 5G company network
- Brings AWS to the edge of 5G networks
- Ultra low latency through 5G networks
- Traffic doesn't leave the communication service providers network
`AWS Local Zones`:
- We can extend the reach of a VPC on an AWS region through local zones
- Example:
	- AWS Region: N.Virginia (us-east-1)
	- AWS Local Zones: Boston, Chicago, Dallas...
# Cloud Integrations
--- 
## Simple Queue Service (SQS)
- Serverless service to decouple applications
- Default retention of messages is 4 days, up to 14
- No limit to messages in queue
- Messages are deleted after they're read
- Low latency
- Consumers share the work to read messages and scale horizontally
- Can use First In First Out queue
## AWS Kinesis
- Real time big data streaming 
- Collect, process and analyze at any scale
## Simple Notification Service (SNS)
- Event Publishers only sends messages to one SNS topic
- Many event subscribers listen to the SNS topic notifications
- Each subscriber to the topic will get all the messages
- Up to 12,500,000 subscriptions per topic up to 100,000 topics
## AWS MQ
- SQS and SNS are native to AWS
- Traditional applications may use other open protocols
- Instead of reengineering the application to use SQS or SNS we can use AWS MQ to do this
- Managed message broker service for RabbitMQ and ActiveMQ
# Cloud Monitoring 
--- 
## CloudWatch
`Important Metrics`:
- EC2 Instances: CPU Utilization, Status Checks, Network
	- Default metrics every 5 minutes
	- Option for detailed monitoring, metrics every minute
- EBS Volumes: Disk Read/Writes
- S3 Buckets: BucketSizeBytes, NumberOfObjects, AllRequests
- Billing: Total Estimated Charge, only in us-east-1
- Service Limits: how much you have been using a service API
- Custom metrics
`Cloudwatch Alarms`:
- Used to trigger notifications for any metric
- Auto Scaling: increase or decrease EC2 instances desired count
- EC2 Actions: stop, terminate, reboot or recover an EC2 instance
- SNS Notifications: send a notification into an SNS topic
- Various options, sampling, %, max, min...
- Can choose period on which to evaluate alarm
- Alarm states: OK. INSUFFICIENT_DATA, ALARM
`CloudWatch Logs`:
- Self explanatory, reads logs from different AWS services
- For EC2 logging, you must install CloudWatch Logs Agent on the Instance, can also be installed on on-premise servers for hybrid cloud
`EventBridge`:
- Formerly CloudWatch Events
- Schedule CRON jobs (scheduled scripts)
- Event pattern: Event rules to react to a service doing something
- Can receive events from AWS partners via Partner Event Bus and from custom apps via Custom Event Bus
- Events from AWS services come via Default Event Bus
- Schema Registry: model event schema
- You can archive events and also replay them
## CloudTrail
- Provides Governance, Compliance and Audit for your AWS account
- Enabled by default
- Allows you to get a history of events/API calls made within your AWS account by console, SDK, CLI or AWS services
- Can put logs into CloudWatch Logs or S3
- A trail can be applied to all regions (default) or a single region
## AWS X-Ray
- Debugging tool for distributed services
- Visual analysis of the applications
- Troubleshooting performance bottlenecks
- Understand dependencies in architecture
- Pinpoint service issues
- Review request behaviour
- Find errors and exceptions
- Where am I throttled
- Identify impacted users
## AWS CodeGuru
- Machine learning powered service for automated code reviews (reviewer) and application performance recommendations during runtime (profiler)
- Supports Java and Python
- Integrations with git systems
## AWS Health Dashboard
Service history: 
- Shows all regions, all services' health, shows historical information
Your Account:
- Alerts and remediation guidance for events that affect you
- Personalized view into performance and availability of AWS services underlying your AWS resources
- Displays relevant and timely information
- Provides proactive notifications for scheduled activities (like maintenance)
- Can aggregate data for your entire AWS organization
# VPC and Networking
--- 
## IP Addresses
`IPv4`:
- 4.3 Billion Addresses
- Public IPv4
- EC2 gets new Public IPv4 when stopped and restarted
- Private IPv4, and its fixed for EC2 instances even if restarted
- Paid per hour
`Elastic IP`:
- Fixed Public IPv4 address for an EC2 instance
`IPv6`:
- $3.4\times 10^{38}$ Addresses
- Every IP is public, there is no private range
- Free
## VPC
- Virtual Private Cloud, private network to deploy your regional resources
- Subnets allow you to partition your network inside your VPC.
- To define access to the internet and between subnets, we use a route table.
- To connect a VPC to the internet we use Internet Gateways. Public subnets have a route to the internet gateway.
- NAT Gateways (managed) and NAT Instances (self-managed) allow your VPC instances in your private subnets to access the internet while remaining private.
- Logging is done via Flow Logs, network traffic logs
- VPC peering connects two VPCs privately using AWS network, makes them behave as if they were the same network
	- Must not have overlapping CIDR rules
	- Not transitive connection, must be established for each VPC that needs to communicate with one another
- Transit Gateway allows transitive peering between thousands of VPC and on premises
	- Uses a hub and spoke (star) connection
	- One single gateway to provide this functionality
- VPC endpoints allow you to connect to AWS services using a private network instead of the public www network
	- This gives enhanced security and lower latency
	- 2 types, Gateway -> S3 and DynamoDB, Interface -> most services including S3 and DynamoDB
## Network ACL and Security Groups
`NACL`:
- Firewall which controls traffic to and from a subnet
- Numbered list of rules, rules are evaluated in increasing order to decide whether to allow or not
- Can have ALLOW or DENY rules
- Attached at the Subnet level
- Rules only include IP addresses
`Security Groups`:
- Firewall that controls traffic to and from an EC2 Instance
- Can have only ALLOW rules
- Rules include IP addresses and other security groups
- Stateful, automatically allows return traffic
## AWS PrivateLink
- Most secure and scalable way to expose a service to 1000s of VPCs
- Does not require VPC peering, internet gateway, NAT, route tables...
![[PrivateLink.png]]
## On-Premise to VPC
`Site to Site VPN`:
- Connect an on premises VPN to AWS 
- Connection is automatically encrypted
- Goes over public internet
- Fast to set up
	- On premise must use a Customer Gateway
	- AWS must use a Virtual Private Gateway
	- Then use Site to Site VPN to connect CGW with VGW
`Direct Connect (DX)`:
- Establish a physical connection between on premise and AWS
- Connection is secure and fast
- Goes over private network
- Takes at least a month to establish, also very expensive
`AWS ClientVPN`:
- Connect from your computer using OpenVPN to your private network in AWS and on premise
- Allows you to connect to your EC2 instances over a private IP, as if you were on the private VPC network
- Goes over public network
- If AWS VPC is connected via site to site VPN to on premise servers, you can also connect to them with your computer via ClientVPN
# Security and Compliance
--- 
## Network Protection
`DDoS Protection`:
- AWS Shield Standard, protects against DDoS attacks, enabled for all customers for free
	- Provides protection from attacks such as SYN/UDP floods, Reflection Attacks and other layer 3/4 attacks
- AWS Shield Advanced, 24/7 Premium DDoS protection
	- Expensive
	- Protects against more sophisticated attacks
	- 24/7 access to AWS DDoS response team
- AWS WAF, Filter specific requests based on rules
	- Protects web applications from common web exploits on layer 7 (HTTP)
	- Deploy on Application Load Balancer, API Gateway, CloudFront
	- Define Web ACL
		- Rules can include IPs, HTTP headers, HTTP body or URL strings
		- Protects from common SQL injection and Cross-Site Scripting attacks
		- Size constraints for packets or block certain countries
		- Rate-based rules
- CloudFront and Route 53, availability protection using global network, combined with AWS Shield provides attack mitigation at the edge
- Be ready to scale when under attack, i.e. leverage AWS Auto Scaling
`AWS Network Firewall`:
- Protects your entire Amazon VPC
- From layer 3 to layer 7 protection
`AWS Firewall Manager`:
- Manage all security rules in all accounts of an AWS organization
- Manages VPC security groups, WAF rules, AWS Shield Advanced, AWS Network Firewall
- Rules are applied to new resources as they are created (good for compliance) across all current and future accounts in your organization
`Pentesting`:
- It is allowed only on 8 services
- Anything that might look like an attack is not allowed, like DDoS or flooding
## Encryption
`AWS Key Management Service (KMS)`:
- Manages the encryption keys for us
- Optional encryption for some services, basically all the storage ones, whether it be for objects or databases
- Some services like CloudTrail Logs, S3 Glacier and Storage Gateway, EFS are automatically encrypted
- Customer Managed Key:
	- Create, manage and used by customer
	- Can use rotation policy
	- Can bring your own key
- AWS Managed Key:
	- Created, managed and used on the customers behalf by AWS
	- Used by AWS services (aws/s3, aws/ebs...)
- AWS Owned Key:
	- Collection of CMKs that an AWS service owns and manages 
	- AWS can use those to protect your resources but you can't view the keys
- CloudHSM Keys (custom keystore):
	- Generated from your own CloudHSM hardware device
	- Cryptographic operations are performed within the CloudHSM cluster
`CloudHSM`:
- AWS provisions encryption hardware with a Hardware Security Module (HSM)
- You manage your own encryption keys entirely
- HSM device is tamper resistant, FIPS 140-2 level 3 compliance
## Other Services
`AWS Certificate Manager (ACM)`:
- Manages SSL/TLS Certificates
- Used to provide in flight encryption for websites (HTTPS)
- Supports both public and private TLS certificates
- Free for public TLS certificates
- Automatic TLS certificate renewal
`Secrets Manager`:
- Meant for storing secrets
- Capability to force rotation of secrets every X days
- Automate generation of secrets on rotation using Lambda
- Integration with RDS
- Encrypted with KMS
`AWS Artifact`:
- Portal that provides on demand access to AWS compliance documentation and agreements
- Artifact Reports -> security and compliance documents from third party auditors
- Artifact Agreements -> AWS agreements
`AWS GuardDuty`:
- Intelligent Threat discovery
- Machine learning algorithms for anomaly detection 
- If findings are detected, send to EventBridge
`AWS Inspector`:
- Automated security assessments
- For EC2 Instances:
	- Leverage AWS System Manager
	- Analyze against unintended network accessibility
	- Analyze running OS for known vulnerabilities
- For Container images push to Amazon ECR:
	- Assessment of container images as they are pushed
- For Lambda functions:
	- Identifies software vulnerabilities in function code and package dependencies
	- Assessment of functions as they are deployed
- Reporting and integration with AWS Security hub and EventBridge
`AWS Config`:
- Records configurations and changes over time
- Per region service and can aggregate results over all regions and accounts
- View compliance and configuration of a resource over time
`AWS Macie`:
- Managed data security and data privacy service that uses machine learning and pattern matching to discover and protect sensitive data
- Also alerts you
`AWS Security Hub`:
- Central security tool to manage security across several AWS accounts and automate security checks
- Automatically aggregates alerts from various AWS security services
- Must first enable AWS Config
`AWS Detective`:
- Analyzes, investigates and identifies the root cause of security issues
- Automatically collects and processes events from VPC Flow Logs, CloudTrail, GuardDuty and creates a unified view
## Root User Privileges
Actions that can be performed only by the root user:
- Change account settings
- View certain tax invoices
- Close your AWS account
- Restore IAM user permissions
- Change or cancel AWS support plan
- Register as a seller in the Reserved Instance Marketplace
- Configure S3 Bucket to enable MFA
- Edit or delete an S3 bucket policy + smth else not important i think
- Sign up for GovCloud
# Machine Learning
--- 
`AWS Rekognition`:
- Finds objects, people, text, scenes in images and videos using ML
`AWS Transcribe`:
- Automatically convert speech into text
- Uses a deep learning process called automatic speech recognition (ASR)
- Automatically remove Personally Identifiable Information (PII) using Redaction
- Supports automatic language identification for multi-lingual audio
`AWS Polly`:
- Turns text into lifelike speech using deep learning
`AWS Translate`:
- Translates languages
`AWS Lex`:
- Powers Alexa
- Build Chatbots
- Automatic speech recognition (ASR) to convert speech to text
- Natural language understanding to recognize intent 
`AWS Connect`:
- Receive calls, create contact flows, cloud based virtual contact center
- Can integrate with other customer relation management (CRM) systems or AWS
- So someone calls the company number, which gets sent to AWS Connect, it streams the call to Lex which recognizes what the user wants, and invokes some Lambda function for redirection or maybe sending a command to a CRM
`AWS Comprehend`:
- Natural Language Processing (NLP)
- Serverless and fully managed
- Uses machine learning to find insights, information or relationships in text
	- For example analyzing customer emails to find what leads to positive or negative experiences
`AWS SageMaker`:
- Fully managed service for developers/data scientists to build ML models
- Basically what I need to know for the exam is the process of building and training a model and making predictions using it. My SHP
`AWS Kendra`:
- Managed document search service 
- Extract answers from within a document
- Kendra builds a knowledge index, so makes it easy to search for information in the documents
- Natural language search capabilities
- Incremental Learning
`AWS Personalize`:
- ML service to build apps with real time personalized recommendations
`AWS Textract`:
- Automatically extracts text, handwriting and data from any scanned documents
- For example, scan an id, extracts all the info
# Account Management
--- 
`AWS Organizations`:
- Allows to manage multiple AWS accounts
- Main account is master method
- Consolidated Billing across all accounts
- Pricing benefits from aggregated usage, more discounts
- Pooling of Reserved EC2 instances for optimal savings
- API is available to automate AWS account creation
- Restrict account privileges using Service Control Policies
	- Whitelist or blacklist IAM actions
	- Applied at organizational unit or account level
	- Doesn't apply to master account
	- Applies to all users and roles of the account
`AWS Control Tower`:
- Easy way to set up and govern a secure and compliant multi account AWS environment based on best practices
- It runs on top of AWS Organizations
`AWS Resource Access Manager (RAM)`:
- Share AWS resources that you own with other AWS accounts, can be done within your organization
- Avoids resource duplication
`AWS Service Catalog`:
- Self service portal to launch a set of authorized products predefined by admins
# Billing
--- 
## Pricing Models
- `Pay as you go`: Pay for what you use
- `Save when you reserve`: Reserving resources in advance and commit to them
- `Pay less by using more`: Volume discounts
- `Pay less as AWS grows`
Check 'Pricing Models of the Cloud' from the course for a good rundown of all the pricing
## Savings Plan
- Commit a certain amount of money per hour for 1 or 3 years
- EC2 Savings plan, Commit to usage of individual instance families in a region
- Compute Savings Plan, discounts for EC2, Fargate, Lambda
- Machine Learning Savings Plan: SageMaker
## Tools
`AWS Compute Optimizer`:
- Reduce costs and improve performance by recommending optimal AWS resources for your workloads
- Supports EC2, EC2 ASG, EBS, Lambda
`AWS Billing and Costing Tools`:
- Estimating Costs:
	- Pricing Calculator, estimates the cost of your architecture solution
- Tracking Costs:
	- Billing Dashboard, shows the monthly cost and forecasted
	- Cost Allocation Tags, allows to track AWS costs on a detailed level by tagging resources, there are AWS generated ones and user defined ones. Also used to organize resources.
	- Have to enable either tag enabled to use them, also each tag key must be unique and only 1 value.
	- Cost and Usage Reports, Most comprehensive set of AWS cost and usage data available. Generates reports to analyze your bill
	- Cost Explorer, Visualize, understand and manage your costs and usage. Create custom reports, but at a high level. Forecast your bills up to 12 months
- Monitoring Costs:
	- Billing Alarms, Simple alarm if threshold is crossed
	- AWS Budgets, Create budget and send alarms when costs exceed the budget or forecast does
`AWS Cost Anomaly Detection`:
- Uses machine learning to detect unusual spendings
`AWS Service Quotas`:
- Notifies you when close to a service quota threshold
- Request a quota increase or shutdown resources before limit is reached
`AWS Trusted Advisor`:
- High level account assessment
- For a full set of checks, you need the Business or Enterprise Plan
![[Trusted Advisor.png]]
## Support Plans
`Basic`
`Developer`:
- Business hour email access to cloud support associates
- Unlimited cases/contacts
- Case severity response times, General guidance < 24h, System impaired < 12h
`Business`:
- For production workloads
- Trusted advisor, full set of checks and API access
- 24/7 phone, email and chat access to cloud support engineers
- Case severity response times, Production system impaired < 4h, Production system down < 1h
`Enterprise On-Ramp`:
- Production or business critical workloads
- Access to a pool of Technical Account Managers (TAM)
- Concierge support team (for billing and account practices)
- Infrastructure event management, well architected and operations reviews
- Case severity response times, Business critical system down < 30m
`Enterprise`:
- Mission critical workloads
- Designated Technical account manager
- Access to AWS Incident Detection and Response for an additional fee
- Business critical system down < 15m now
## Best Practices Summary
![[Best Practices.png]]
# Advanced Identity
--- 
`AWS Security Token Service (STS)`:
- Runs in the background for loads of things we've done so far like IAM roles
- Creates temporary, limited privileged credentials to access AWS resources
- Short term credentials, you configure the expiration period
`AWS Cognito`:
- Identity for your web and mobile applications
- Instead of creating an IAM user, which you shouldn't ever do, you create an user in Cognito
`AWS Directory Services`:
- Active Directory support 
`AWS IAM Identity Center`:
- Single sign on for all your AWS accounts in AWS organizations
- 1 login for multiple accounts
![[Identity Center.png]]
# Other Services
--- 
May or may not appear in the exam
`AWS WorkSpaces`:
- Managed desktop as a service for windows or linux desktops
`AWS AppStream 2.0`:
- Desktop application streaming, delivered from within a web browser
`AWS IoT Core`:
- Easily connect IoT devices to the AWS cloud
`AWS AppSync`:
- Store and sync data across mobile and web apps in real time
- Makes use of GraphQL
`AWS Amplify`:
- Set of tools and services to develop and deploy scalable full stack web and mobile applications
- Elastic beanstalk for mobile and web apps equivalent
`AWS Infrastructure Composer`:
- Visually design and build serverless applications quickly on AWS
- Deploy AWS infrastructure as code without needing to be an expert on AWS
`AWS Device Farm`:
- Tests your web and mobile apps against desktop browsers, real mobile devices and tablets
- Quite literally running stuff on real devices farm
`AWS Backup`:
- Need I say more?
`AWS DataSync`:
- Move large amount of data from on premise to AWS
- Replication tasks can be scheduled however you want and are incremental after the first full load
`Cloud Migration Strategies`:
- Retire, turn off things you don't need
- Retain, Do nothing for now, not worth or other reason
- Relocate, Move on premise to cloud or move stuff within the cloud
- Rehost, Simple migrations by re-hosting on AWS
- Replatform, Migrate database to RDS, or another example
- Repurchase, Moving to a different product while moving to the Cloud
- Refactor/Rearchitect, Reimagining how the application is architected using Cloud native features
`AWS Application Discovery Service`:
- Plan migration projects by gathering information about on premises data centers
- Agentless discovery (AWS Agentless Discovery Connector)
- Agent based discovery (AWS Application Discovery Agent)
`AWS Application Migration Service`:
- Rehosting solution
`AWS Migration Evaluator`:
- Helps build a data driven business case for migration to AWS
- Analyze current state, define target state, then develop migration plan
`AWS Migration Hub`:
- Central location to assess, plan and track migrations to AWS
`AWS Fault Injection Simulator`:
- Runs fault injection experiments on AWS workloads based on chaos engineering
`AWS Step Functions`: 
- Build visual workflows to orchestrate Lambda functions
- Can integrate with other AWS services
- Possibility of implementing human approval feature
`AWS Ground Station`:
- Lets you control satellite communications, process data and scale satellite operations
`AWS Pinpoint`:
- Scalable 2 way marketing communications service
# Architecting and Ecosystem
--- 
`General Guiding Principles`:
- Stop guessing your capacity needs
- Test systems at production scale
- Automate to make architectural experimentation easier
- Allow for evolutionary architecture
- Drive architecture using data
- Improve through game days, simulate applications for flash sale days
`Design Principles`:
- Scalability
- Disposable resources, easily configured
- Automation
- Loose Coupling, break monolith applications into smaller loosely coupled components
- Services, not Servers
## Well Architected Framework
`1. Operational Excellence:
`2. Security`
`3. Reliability`
`4. Performance Efficiency`
`5. Cost Optimization`
`6. Sustainability`
We can use the AWS Well-Architected Tool to review your architectures against the 6 pillars and adopt best practices.
For the sustainability pillar, we can use the AWS Customer Carbon Footprint Tool to display our carbon footprint over time.
`AWS Cloud Adoption Framework (CAF)`:
- Not a service, but a white paper
- Identifies specific organizational capabilities that underpin successful cloud transformations
- Its capabilities are grouped in 6 perspectives:
	- Business
	- People
	- Governance
	- Platform
	- Security
	- Operations
