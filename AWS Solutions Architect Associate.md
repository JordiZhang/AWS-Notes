--- 
# EC2 
--- 
## Placement Groups
Strategies:
- `Cluster`: clusters instances into a low-latency group in a single AZ. For high network throughput workloads that doesn't care about availability.
- `Spread`: spreads instances across hardware in different AZ, max 7 instances per AZ. So if one server on an AZ fails, the other ones on the same AZ won't, for critical applications. Also since multi AZ setup, high availability.
- `Partition`: spreads instances across many partitions in different sets of racks within an AZ, scales to 100s of instances per group. Similar to spread, but instead of 1 instance in each server rack, instead each partition can have multiple instances and each partition is on a separate server rack. Up to 7 partitions per AZ.
## Elastic Network Interfaces (ENI)
Logical component in a VPC that represents a virtual network card that is AZ bound. Can be attached and moved independently from EC2 instances.
Can have the following:
- Primary private IPv4, one or more secondary IPv4
- One Elastic IPv4 per private IPv4
- One Public IPv4
- One or more security groups
- A MAC address
## EC2 Hibernate
- RAM state is preserved
- Instance boot is faster since the instance wasn't stopped
- Under the hood, the RAM state is written to a file in the root EBS volume
- Root EBS volume must be encrypted
- Cannot hibernate more than 60 days
## EC2 Spot Fleets
Set of Spot Instances and optionally on demand instances. Spot Fleet will try to meet target capacity with price constraints. User defines possible launch pools (instance type, OS, AZ). Can have multiple launch pools so that the fleet can choose. The Spot Fleet stops launching instances when reaching capacity or maximum cost. Essentially allows us to automatically request Spot Instances with lowest price.
Strategies to allocate Spot Instances:
- `lowestPrice`: chooses from pool with lowest price, good for cost optimization and short workloads
- `diversified`: distributed across all pools, good for availability and long workloads
- `capacityOptimized`: pool with optimal capacity for the number of instances
- `priceCapacityOptimized`: pools with highest capacity available first, then selects from pool with lowest price, best choice for most workloads
## EBS Volume Types
`gp2/gp3 (SSD)`: General purpose SSD volume that balances price and performance for a variety of workloads
- gp2: IOPS and size of volume are linked, 3 IOPS per GB and max IOPS is 16000
- gp3: Can set IOPS up to 16000 and throughput up to 1000 MiB/s independent of each other, i.e. not linked
`io1/io2 Block Express (SSD)`: Highest performance SSD volume for mission critical low latency or high throughput workloads, or applications that require more than 16000 IOPS. Also supports EBS Multi-attach
- io1: Max PIOPS (Provisioned IOPS) of 64000 for Nitro EC2 instances and 32000 for other ones. Can increase IOPS independently from storage size
- io2 Block Express: Sub-millisecond latency and has max PIOPS of 256000 with 1000 IOPS per GB of storage size
`st1 (HDD)`: Low cost HDD volume designed for frequently accessed, throughput optimized/intensive workloads
- Used for Big data, data warehouses 
- Max throughput of 500 MiB/s and max IOPS of 500
`sc1 (HDD)`: Lowest cost HDD volume designed for less frequently accessed workloads
- Infrequently accessed data and when lowest cost is important
- Max throughput of 250 MiB/s and max IOPS of 250
EBS Volumes are characterized in Size, Throughput and IOPS. Only gp2/gp3 and io1/io2 Block Express can be used as boot volumes
## EBS Multi-Attach io1/io2
Allows the same EBS volume to attach to multiple EC2 instances in the same AZ. Each instance has full read and write permissions to the volume. Can attach up to 16 EC2 instances at a time. Must use a file system that is cluster-aware. Use cases:
- Achieve higher application availability in clustered Linux applications
- Applications must manage concurrent write operations

## Elastic File System
Managed pay-per-use file system that can connect to 100s of EC2 instances, only supports Linux AMIs. Uses the NFSv4.1 Protocol and the POSIX (Linux) file system that has a standard file API. The file system scales automatically so no capacity planning needed.
### `Performance`
EFS Scale: 
- 1000s of concurrent NFS clients with 10 GB+/s throughput total
- Grow to Petabyte scale network file system automatically
Performance Mode set at creation time:
- General Purpose (default): latency-sensitive use cases
- Max I/O: higher latency and higher throughput. Highly parallel, so good for big data or media processing
Throughput Mode:
- Bursting: Throughput scales with amount of storage with a bit of a buffer in bursts
- Provisioned: set throughput regardless of storage size
- Elastic: automatically scales throughput based on workload. Great for unpredictable workloads. Up to 3 GB/s for reads and 1 GB/s for writes
### `EFS Storage Classes`
Storage Tiers (Lifecycle management, move file after N days):
- Standard
- Infrequent Access: cost to retrieve files but lower cost to store
- Archive: rarely accessed data, few times a year
- Implement lifecycle policies to move files between tiers
Availability and durability
- Standard: Multi-AZ, good for production
- One Zone: One AZ, good for dev. Backups are enabled by default and compatible with One Zone IA. Reduced cost
# Load Balancing and Auto Scaling 
--- 
## Application Load Balancer
Layer 7 (HTTP) Load Balancer. Supports HTTP, HTTPS and WebSocket and redirects (for example from HTTP to HTTPS). Supports routing tables to different target groups:
- Based on URL path (example.com/users or example.com/posts)
- Based on hostname (one.example.com or other.example.com)
- Based on query strings and headers (example.com/users?id=123&order=false)
Great for micro services and containers. Has port mapping feature to redirect to a dynamic port in ECS. They have a fixed hostname. Application servers don't see the IP of the clients directly, instead the true IP of the client is inserted in the header (X-Forwarded-For). We can also get the port and proto from X-Forwarded-Port/Proto
![[ALB Example.png]]
## Network Load Balancer
Layer 4 (TCP/UDP) Load Balancer. Handles millions of requests per second, very high performance and ultra low latency. NLB has a single static IP per AZ and supports assigning an Elastic IP. Target groups:
- EC2 instances
- IP addresses - must be private IPs
- Application Load balancer, so NLB is in front of ALB.
- Health checks support TCP, HTTP and HTTPS protocols
![[NLB Example.png]]
## Gateway Load Balancer
Used to deploy, scale and manage a fleet of 3rd party network virtual appliances in AWS. Layer 3 (Network Layer), IP Packets. Combines a Transparent Network Gateway (single entry/exit for all traffic), and a load balancer (Distributes traffic to virtual appliances). Uses GENEVE protocol on port 6081. Target Groups:
- EC2 instances
- IP addresses - must be private IPs
![[GLB Example.png]]
## Sticky Sessions or Session affinity
Stickiness = same client always redirected to the same instance behind a load balancer. Can be enabled for application and network load balancers, ALB uses cookies with an expiration date, while NLB doesn't. Stickiness could bring imbalance to the EC2 instances in the backend.

Application based cookies: 
- `Custom cookie`
	- Generated by the target
	- Can include any custom attributes
	- Cookie name must be specified individually for each target group.
	- Cannot use AWSALB, AWSALBAPP or AWSALBTG
- `Application cookie`
	- Generated by the load balancer
	- Cookie name is AWSALBAPP
Duration based cookies:
- Cookie generated by load balancer
- Cookie name is AWSALB for ALB, AWSELB for CLB (CLB is deprecated)
## Cross Zone Load Balancing
- ALB: enabled by default and no charges for inter AZ data
- NLB & GLB: disabled by default and pricing for inter AZ data.
![[Cross Zone Balancing.png]]
## SSL/TLS
An SSL certificate allows traffic between client and load balancer to be encrypted in transit. SSL is Secure Sockets Layer, used to encrypt connections. TLS is Transfer Layer Security, which is a newer version of SSL. Now a days, TLS certificates are mainly used but are commonly referred to as SSL. Public SSL certificates are issued by a Certificate Authority. SSL certificates have an expiration date and have to be renewed.

The load balancer uses an X.509 certificate (SSL/TLS server certificate). You can manage certificates using ACM (AWS Certificate Manager). You can create upload your own certificates alternatively. HTTPS listener:
- You must specify a default certificate
- You can add an optional list of certs to support multiple domains
- Clients can use SNI (Server Name Indication) to specify the hostname they reach
- Ability to specify a security policy to support older versions of SSL /TLS (legacy clients)

SNI (Server Name Indication) solves the problem of loading multiple SSL certificates onto one web server (to serve multiple websites). It's a "newer" protocol, and requires the client to indicate the hostname of the target server in the initial SSL handshake. The server will then find the correct certificate, or return the default one. Only works for ALB, NLB and CloudFront
## Connection Draining/Deregistration Delay
Essentially, if an EC2 instance is in draining mode, the ELB stops sending new requests to that instance but allows the existing requests to finish before deregistering the instance. Can set the delay between 1 to 3600 seconds, default is 300s.
## Auto Scaling Groups
Same info as Cloud Practitioner. It is possible to scale an ASG based on CloudWatch Alarms. The alarm could monitor a metric such as average CPU usage or another custom metric. The alarm can then trigger a scale in/out policy.

Scaling Policies:
- Dynamic Scaling
	- Target Tracking Scaling
		- Simple to set-up
		- Example: I want the average ASG CPU to stay at around 40%
	- Simple / Step Scaling
		- When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units
		- When a CloudWatch alarm is triggered (example CPU < 30%), then remove
- Scheduled Scaling
	- Anticipate a scaling based on known usage patterns
	- Example: increase the min capacity to 10 at 5 pm on Fridays
- Predictive Scaling: Continuously forecast load and schedule scaling ahead

Good Metrics:
- CPU utilization
- RequestCountPerTarget: say from testing you know the optimal number for your EC2 instances.
- Average Network In/Out: for network bound applications
After a scaling activity occurs, there is a cooldown period (default 300s), during which ASG will not launch or terminate instances to allow for metrics to stabilize.
# Databases - RDS & Aurora
---
Relational Database Service, managed DB service to create databases in the cloud that are managed by AWS. Supports Postgres, MySQL, MariaDB, Oracle, Microsoft SQL Server, IBM DB2, Aurora. Has Storage Auto Scaling, so when RDS detects you are running out of storage, it scales automatically. You have to set a maximum storage threshold, so no infinite scaling on accident, and can set conditions to modify storage if:
- Free Storage is less than 10% of allocated storage
- Low storage lasts at least 5 minutes
- 6 hours have passed since last modification
Useful for unpredictable workloads
## Read Replicas and Multi AZ
We can create up to 15 Read Replicas that help scale read operations within an AZ, cross AZ or cross region. Replication is ASYNC so reads are eventually consistent if given enough time. Replicas can be promoted to their own database. 

In AWS there is a network cost when data goes from one AZ to another. For Read Replicas within the same region, you don't pay that fee.

Multi AZ is a SYNC replication of the database on standby. When using Multi AZ, there is one DNS name for the master DB and standby DB. There is automatic failover in case of loss of AZ. If master fails, standby takes over. Note that standby's are in different AZ from master. You can also set a read replica setup as Multi AZ for disaster recovery. 

When modifying a DB from single AZ to Multi AZ, this is a zero downtime operation so there is no need to stop the DB. Internally, a snapshot is taken, then a new DB is restored from the snapshot in a new AZ, then synchronization is established between the two DBs.
## RDS Custom
Allows OS and database customization for Oracle and Microsoft SQL server. RDS automates the setup, operation and scaling of database in AWS, but RDS Custom allows us to access underlying database and OS so you can:
- Configure settings
- Install patches
- Enable native features
- Access underlying EC2 instance using SSH or SSH Session Manager
When using RDS Custom, you should de-activate Automation Mode to perform your customization, better to take a DB snapshot before.
## Amazon Aurora
Proprietary technology from AWS that supports Postgres and MySQL. It is a cloud optimized database for AWS that has better performance than RDS running on Postgres or MySQL. Aurora storage automatically grows in increments of 10GB up to 128TB. Can have up to 15 replicas and replication is faster than MySQL. Failover in Aurora is instantaneous. Cloud native so high availability. Aurora is 20% more expensive than RDS but more efficient.

Aurora stores 6 copies of your data across 3 AZ:
- Needs 4 copies out of 6 for writes
- Needs 3 copies out of 6 for reads
- Self healing with p2p replication
- Storage is striped across 100s of volumes
One aurora instance takes writes (master), writes to shared storage volume. Automated failover for master in less than 30s. Master + up to 15 aurora read replicas serve reads. Any of the read replicas can become master. Supports cross region replication. Read replicas can be auto scaled to have the right number of read replicas. 

Client connects to writer endpoint, writer endpoint automatically points to the master. All read replicas are connected to reader endpoint which handles load balancing. Client can then connect to reader endpoint for reads. When using replica auto scaling, whenever new read replicas are added, the reader endpoint will be automatically extended to include the replicas. We could have different sized aurora read replicas, often done to define a subset of instances as a custom endpoint. For example, to run analytical queries on specific replicas. Reader endpoint is commonly not used anymore if using a custom endpoint. 

`Aurora Serverless`: Serverless version of aurora where the client connects to a proxy fleet managed by aurora. Databases are instantiated automatically and auto scaled based on usage. Good for infrequent, intermittent or unpredictable workloads. Pay per second.

`Global Aurora`: Supports cross region read replicas. Aurora global database is the recommended way of running global aurora. 
- 1 Primary Region (read/write)
- Up to 10 secondary (read only) regions, replication lag less than 1 second
- Up to 16 read replicas per secondary region
- Promoting a region has an RTO of less than a minute (DR purposes)
- Typical cross region replication takes less than 1 second

`Aurora Machine Learning`: Enables you to add ML based predictions to your applications via SQL. Integrates with SageMaker and Comprehend.

`Babelfish for Aurora PostgreSQL`: Allows Aurora PostgreSQL to understand commands targeted for MS SQL Server (T-SQL). Therefore MS SQL Server based applications can work on Aurora PostgreSQL. Requires little to no code changes and same applications can be used after a migration of your database.
## RDS Backups
Automated backups: 
- daily full backup of the database and transaction logs are backed up every 5 minutes
- Ability to restore to any point in time from oldest backup to 5 minutes ago
- 1 to 35 days of retention, set to 0 to disable automatic backups.
Manual DB Snapshots:
- Manually triggered by the user
- Retention of backup for as long as you want
## Aurora Backups
Automated Backups:
- 1 to 35 days (cannot be disabled)
- Point in time recovery in that timeframe
Manual DB Snapshots:
- Manually triggered by the user
- Retention of backup for as long as you want
## Restore options
- Restoring a RDS/Aurora backup or a snapshot creates a new database
- Restoring MySQL RDS database from S3, create a backup of your on premise database, store it on S3 and then restore the backup file onto a new RDS instance running MySQL
- Restoring MySQL Aurora cluster from S3. Create a backup of on premise database using Percona XtraBackup, store backup file on S3 and trestore onto a new Aurora cluster running MySQL
## Aurora Database Cloning
Create a new Aurora DB Cluster from an existing one, faster than snapshot and restore. Utilizes the copy-on-write protocol, initially the new DB cluster uses the same data volume as the original DB, when updates are made to the new DB cluster data, then additional storage is allocated and data is copied to be separated. Fast and cost effective.
## RDS & Aurora Security
At rest encryption:
- Master and replicas encryption using KMS, must be defined at launch time
- If master is not encrypted, read replicas cannot be encrypted
- To encrypt an un-encrypted database, uses snapshots and restore
In flight encryption:
- TLS ready by default, use AWS TLS root certificates client-side
IAM authentication: 
- IAM roles to connect to your database instead of user/pass
Security Groups:
- Control network access to your DB
No SSH except with RDS Custom
Audit logs can be enabled and sent to CloudWatch Logs for longer retention.
## RDS Proxy
Fully managed database proxy for RDS. Allows apps to pool and share DB connections established with the database. Improves DB efficiency by reducing stress on DB resources and minimize open connections. Serverless, autoscaling and highly available (Multi AZ). Supports RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server) and Aurora (MySQL, PostgreSQL). No code changes required for most apps. Enforce IAM Authentication for DB, and securely store credentials in AWS Secrets Manager. RDS Proxy is never publicly accessible, must be accessed from the VPC. Common usage of RDS proxy is to pool Lambda functions connections. 
## ElastiCache
Managed Redis or Memcached. Helps reduce load off of databases for read intensive workloads and helps make your application stateless. AWS takes care of OS maintenance, optimization, setup, configuration, monitoring, failure recovery and backups. Using ElastiCache involves heavy application code changes.

`DB Cache`:
Applications queries ElastiCache, if not available then get from RDS and store in ElastiCache for future queries. Relieves load from RDS. Cache must have an invalidation strategy to make sure only the most current data is used on there. 

`User Session Store`:
User logs into application, application then writes the session data into ElastiCache. If user wants to use another instance of the application, instance retrieves the session from the ElastiCache and thus the user is already logged in.

Redis vs Memcached
Needs more detail so look over this part
`Redis`:
- Multi AZ with auto failover
- Read replicas to scale reads and high availability
- Data Durability using AOF persistence
- Backup and restore features
- Supports Sets and Sorted Sets
`Memcached`:
- Multi node for partitioning of data (sharding)
- No high availability
- Non persistent
- Backup and restore
- Multi threaded architecture

Cache Security:
- IAM Authentication for Redis
- IAM policies on ElastiCache are only used for AWS API-level security
- Redis AUTH:
	- Can set a password/token when you create a Redis cluster
	- Extra level of security for your cache on top of security groups
	- Support SSL in flight encryption
- Memcached:
	- Supports SASL-based authentication (advanced)

Patterns:
- Lazy Loading, all read data is cached, data can become stale in cache
- Write Through, adds or update data in the cache when written to a DB (No stale data)
- Session Store: Store temporary session data in a cache

Redis Use Case
- Gaming Leaderboards are computationally complex
- Redis Sorted Sets guarantee both uniqueness and element ordering
- Each time a new element is added, its ranked in real time then added in correct order

**Important ports:**
- FTP: 21
- SSH: 22
- SFTP: 22 (same as SSH)
- HTTP: 80
- HTTPS: 443

**vs RDS Databases ports:**
- PostgreSQL: 5432
- MySQL: 3306
- Oracle RDS: 1521
- MSSQL Server: 1433
- MariaDB: 3306 (same as MySQL)
- Aurora: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)
# Databases - Extended
---
- Relational Database Management Systems (SQL): RDS, Aurora
- NoSQL DB: DynamoDB (JSON), ElastiCache (key/value pairs), Neptune (Graph), DocumentDB (MongoDB), Keyspaces (Apache Cassandra)
- Object Store: S3/Glacier
- Data Warehouse (SQL analytics and BI): Redshift, Athena, EMR
- Search: OpenSearch
- Graphs: Neptune
- Ledge: Amazon Quantum Ledge Database
- Time series: Timestream
## DocumentDB
AWS optimized implementation of MongoDB, NoSQL DB.
- Used to store, query and index JSON data
- Fully managed and highly available with replication across 3 AZ
- DocumentDB storage automatically grows in increments of 10GB
- Automatically scales workloads with millions of requests per second
## Neptune
- Managed graph database
- A graph database could be a social network that is very interconnected as a graph
- Highly available across 3 AZ, up to 15 read replicas
- Build and run applications working with highly connected datasets
- Can store up to billions of relations and query the graph with millisecond latency

Neptune Streams
- Real time ordered sequence of every change to your graph data
- Changes are available immediately after writing
- No duplicates, strict order
- Stream data is accessible using an HTTP REST API from Neptune Streams
- Use cases for enabling Neptune streams:
	- Send notifications when certain changes are made
	- Maintain graph data synchronized with another data store
## Keyspaces (for Apache Cassandra)
Managed service for deploying Apache Cassandra compatible databases
- Serverless, Scalable, highly available
- Automatically scales tables up/down based on applications traffic
- Tables are replicated 3 times across multiple AZ
- Uses Cassandra Query Language (CQL)
- Single digit millisecond latency at any scale, 1000s of requests per second
- Capacity: On demand or provisioned with auto scaling
- Encryption, backups, PITS up to 35 days
## Timestream
Managed service for fast, scalable, time series database. 
- Automatically scales up/down to adjust capacity
- Store and analyze trillions of events per day
- 1000s of times faster and 1/10th the cost of relational databases
- Scheduled queries, multi-measure records, SQL compatibility
- Data storage tiering: recent data kept in memory and historical data kept in a cost optimized storage
- Built in time series analytics functions
- Encryption in transit and at rest
# Data and Analytics
---
## Athena
Serverless query service to analyze data stored in S3
- Uses SQL to query files, built on Presto
- Supports CSV, JSON, ORC, Avro and Parquet
- Pricing $5 per TB of data scanned
- Commonly used with Quicksight for dashboards

Performance Improvements
- Use columnar data for cost savings (less scans), therefore Parquet or ORC is recommended
- Huge performance improvement
- Use Glue to convert data to Parquet or ORC 
- Compress data for smaller retrievals
- Partition datasets in S3 for easy querying on virtual columns
- Use larger files to minimize overhead

Federated Query
- Allows you to run SQL queries across data stored anywhere
- Uses Data Source Connectors that run on AWS Lambda to run Federated queries on anything
## Redshift
PostgreSQL based database used for OLAP (Online Analytical Processing).
- 10x better performance than other data warehouses and scales to PBs of data
- Columnar storage of data and parallel query engine
- Two modes: Provisioned cluster or Serverless cluster
- Has a SQL interface for queries
- Integrates with Quicksight and Tableau

Within a Redshift cluster, there are leader nodes (for query planning and results aggregation) and compute nodes (for performing queries and sending results to leader). In provisioned mode:
- You choose instance types in advance
- Can reserve instances for cost savings

### Snapshots and Disaster Recovery
Redshift is mostly single AZ, but has Multi AZ mode for some clusters. Snapshots are PITR backups of a cluster that are stored internally in S3. Snapshots are increments, only saves changes. You can restore snapshots into a new cluster. Snapshots are automated for every 8 hours, every 5GB or on a set Schedule. Also allows user to set the retention time. Manual snapshots can also be done, and these are retained until you explicitly delete them. You can also configure Redshift to automatically copy snapshots of a cluster to another AWS Region for replication and disaster recovery.
### Loading Data into Redshift
Large inserts are much better in general. We can use the following:
- Data Firehose
- S3 using COPY command
- EC2 instance using JDBC driver

### Redshift Spectrum
This feature allows you to query data that is in S3 without loading it
- Must have Redshift cluster available to start the query
- The query is then submitted to thousands of Redshift Spectrum nodes
## Amazon OpenSearch Service
Used to be called ElastiSearch. In DynamoDB, queries only exist by primary key or indexes. With OpenSearch, you can search any field, even partial matches. Its common to use OpenSearch as a complement to another database. It has a managed cluster or serverless cluster. OpenSearch has its own query language so it doesn't natively support SQL but can be enabled via a plugin. Security through Cognito and IAM, KMS encryption, TLS. Comes with OpenSearch Dashboards for visualization. Essentially used to search for items in different services/storages.
## Amazon ElasticMapReduce (EMR)
EMR helps to create Hadoop clusters to analyze and process vast amounts of data. Clusters can be made of hundreds of EC2 instances. EMR comes bundled with Spark, HBase, Presto, Flink... EMR takes care of all the provisioning and configuration. Has autoscaling capabilities and is integrated with Spot instances.

EMR clusters are made of hundreds of EC2 instances. We have: 
- Master nodes: manage and coordinates the cluster and health - long running
- Core nodes: Run tasks and store data - long running
- Task Nodes (optional): Just to run tasks - Usually spot instances
Purchasing options:
- On demand: reliable, predictable, won't be terminated
- Reserved: cost savings, EMR will automatically use if available
- Spot Instances: cheaper, can be terminated, less reliable
Finally, you can have long running clusters or transient/temporary clusters.
## Amazon Quicksight
Serverless machine learning powered BI tool to create dashboards. Integrates with RDS, Aurora, Athena, Redshift, S3... Has feature to do in-memory computation using the SPICE engine, only available if data is imported into Quicksight. For Enterprise edition, there is possibility to setup Column-Level Security (CLS).

In Quicksight, you define Users (Standard version) and Groups (enterprise version). These users and groups are Quicksight only, not IAM. A dashboard is a read-only snapshot of an analysis that you can share that preserves the configuration of the analysis such as filters and others. You can share the analysis or the dashboard with Users or Groups. Users who see the dashboard can also see the underlying data.
## AWS Glue
Managed ETL service. Glue data catalog is a feature that allows for Glue Data Crawlers to crawl different databases and writes the metadata into a Glue Data Catalog. Glue can then use the catalog to do ETL. Athena, Redshift Spectrum and EMR Leverages Glue Data catalog to perform data discovery.

Features to know:
- Glue Job Bookmarks: prevents re processing old data
- Glue DataBrew: Clean and normalize data using prebuild transformations
- Glue Studio: New GUI to create, run and monitor ETL jobs in Glue
- Glue Streaming ETL (built on Spark Structured Streaming): Instead of batch jobs, it runs the ETL as Streams. Compatible with Kinesis Data Streaming and MSK (managed Kafka)
## AWS Lake Formation
A data lake is a central place to have all your data for analytics purposes. Lake formation is a fully managed service to easily setup a data lake in days, normally might take months, in S3. Useful to discover, cleanse, transform and ingest data into your data lake. Automates many complex manual steps and deduplicates using Machine Learning Transforms. Combines structured and unstructured data in the data lake. It also has blueprints that help you migrate data from S3, RDS and other. Allows for fine grain access control for your applications at the row and column level. Built on AWS Glue. A key use of lake formation is centralized permissions, instead of allowing permissions to tons of sources, Athena and Quicksight and others, we can instead just give permissions to Lake Formation
## Amazon Managed Service for Apache Flink
Flink is a framework for processing data streams in Java, Scale or SQL. Run any Apache Flink application on an AWS managed cluster. It can read from Kinesis Data Streams or Amazon MSK (Kafka). Doesn't read from Data Firehose.
## Amazon Managed Streaming for Apache Kafka
It is an alternative to Amazon Kinesis used to stream data. This service gives us a fully managed Apache Kafka cluster on AWS. Allows you to create, update, delete clusters. Deploys the MSK cluster in your VPC and has multi AZ up to 3 for high availability. Automatic recovery from common Kafka failures. Data is stored on EBS volumes for as long as you want.

There is also a Serverless mode where we don't have to manage capacity and MSK automatically provisions resources and scales.
![[Data Stream vs MSK.png]]
## Big Data Ingestion Pipeline
- We want a fully serverless ingestion pipeline
- We want to collect data in real time
- Transform data
- Query data using SQL
- Create reports using the queries and store in S3
- Load data into warehouse and create dashboards
![[Big Data Pipeline.png]]

# Route 53
---
## What is a DNS??
Domain Name System which translates human friendly hostnames into a machine IP address. I.e. converts a URL into a IP address. It is the backbone of the internet. DNS uses hierarchical naming structure. 

`DNS Terminology`:
- Domain Registrar: i.e. website registrar
- DNS Records: A, AAAA, CNAME, NS
- Zone File: Contains DNS records
- Name Server: resolves DNS queries (authoritative or Non authoritative)
- Top Level Domain (TLD): .com, .us, .in, .gov
- Second Level Domain (SLD): amazon.com, google.com
![[URL Hierarchy.png]]
![[DNS Works.png]]
## Amazon Route 53
Highly available, scalable, fully managed and Authoritative DNS. Authoritative means that the customer (you) can update the DNS records. Route 53 is also a Domain Registrar. It has the ability to check the health of your resources and is the only AWS service which provides 100% availability SLA. 

`Records`:
- Domain/subdomain Name - example.com
- Record Type - A or AAAA
- Value - 12.34.56.78
- TTL - Amount of time the record is cached at DNS Resolvers
Route 53 supports the following DNS record types:
- Must know: A, AAAA, CNAME, NS
- Advanced: CAA, DS, MX, NAPTR, PTS, SOA, TXT, SPF, SRV

### Record Types
- A - maps a hostname to IPv4
- AAAA - maps a hostname to IPv6
- CNAME - maps a hostname to another hostname
	- Target is a domain name which must have an A or AAAA record
	- Can't create a CNAME record for the top node of a DNS namespace (Zone Apex), Only for non root domains
	- Example: Can't create for example.com, but can for www.example.com
- Alias:
	- Points a hostname to an AWS resource
	- Works for root domain and non ROOT domain
	- Free of charge
	- Native Health Check
	- Route 53 specific
	- Alias records are always type A/AAAA and TTL is set by AWS
	- Cannot set an ALIAS record for an EC2 DNS name
- NS - Name Servers for the Hosted Zone
	- Controls how traffic is routed to a domain

## Hosted Zones
A container for records that define how to route traffic to a domain and its subdomains
`Public Hosted Zones`: Contains records that specify how to route traffic on the internet. Anyone on internet can query records.
`Private Hosted Zones`: Contains records that specify how you route traffic within one or more VPCs. Only clients on the VPN can query for records.

You pay $0.50 per month per hosted zone.
## Routing Policies
Defines how Route 53 responds to DNS queries. Not the same as load balancer routing which routes the traffic, instead the DNS doesn't route traffic at all, it only responds to DNS queries. 

`Simple`: Routes traffic to a single resource. Can specify multiple values in the same record, if multiple values are returned, a random one is chosen by the client. If alias record, specify only one AWS resource. Can't have health checks

`Weighted`: Control the percentage of requests that go to each resource by assigning each record a relative weight. DNS records must have same name and type. Can be associated with Health Checks. If all records have weight = 0, all are equally weighted.

`Latency`: Redirect to the resource with least latency close to us. Latency is based on traffic between users and AWS regions, so users in Germany may be redirected to the US if that's the lowest latency (Could be because the Germany one is overloaded). Can be associated with Health checks.

`Failover`: Basically we choose a primary and secondary instances. We set up a mandatory health check on the primary instance and if unhealthy, DNS will answer with the secondary records. Can also set up health checks for secondary resource.

 `Geolocation`: Based on user location. Specify location by Continent, Country or by US state, if overlapping, most precise location is selected. Should create a default record if there is no match on location. Can be associated with health checks. Useful for localization.

`Geoproximity`: Based on user location and resources. Ability to shift more traffic to resources based on defined bias. To expand (1 to 99), more traffic to resource. To shrink (-1 to -99), less traffic to resource. Sort of like a weighted geolocation. Resources can be AWS resources (specify region) or Non-AWS resources (specify Latitude and Longitude). You must use Route 53 Traffic Flow (advanced) to use Geoproximity.

`IP based`: Routes based on client IP addresses. You provide a list of CIDRs for your clients and corresponding endpoints. Used to optimize performance and reduce network costs.

`Multi-Value`: Routes traffic to multiple resources. Route 53 returns multiple values/resources. Can be associated with health checks. Up to 8 healthy records are returned for each Multi-Value query. It is not a substitute for an ELB, instead it acts as a client side load balancing. Differs from simple because it allows health checks so only healthy resources are returned.
## Health Checks
Health checks can be implemented for some routing types, they are only for public resources. Essentially they enable automated DNS failover in case one of the endpoints stops working or something is wrong with it. Health checks are integrated with CloudWatch
- Health checks that monitor an endpoint (Application, Server, other AWS resource)
	- About 15 global health checkers will check the endpoint health, if >18% of health checkers report healthy, the endpoint is healthy. Interval is 30s but can set to 10s at a higher cost. Supports HTTP, HTTPS and TCP. 
	- Can set failure threshold, i.e. how many fails before considered unhealthy for each checker
	- Health checks pass if response is 2xx or 3xx.
	- Checks can be setup to pass/fail based on text in the first 5120 bytes of response
	- Resource that is being checked must allow requests from the checkers' IP address ranges.
- Health checks that monitor other health checks (Calculated health checks)
	- Combine results of multiple health checks into a single health check
	- OR, AND or NOT
	- Can monitor 256 child health checks
	- Specify how many child health checks passed to make parent pass
- Health checks that monitor CloudWatch Alarms (Full Control)
	- Useful for Private Hosted Zones and outside health checks. 
	- Health checks can't access private endpoints private VPC or on premise resource, so we can create a CloudWatch Metric and associate an alarm. Then create a health check that checks the alarm
## Hybrid DNS
Essentially having multiple DNS resolvers talking to each other. Inbound endpoints forwards DNS queries from other DNS resolvers to the Route 53 resolver. Outbound endpoints forwards DNS queries from the Route 53 resolver to your other resolvers. Check Lecture 120 for more detail. 
# Solution Architectures
---
## WhatsTheTime.com
Stateless Web App
- We don't need a database
![[Stateless Webapp.png]]
## MyClothes.com
Stateful Web App
- Allows people to buy clothes online
- There is a shopping cart
- Has hundreds of users at the same time
- We need scaling and keep the web app as stateless as possible
- Users should not lose their shopping cart
- Users should have their details stored in a database

How can we keep the shopping cart?
- We could enable ELB Stickiness so that the shopping cart is stored on the EC2 Instance and thus the user always accesses the same instance
- We could send shopping cart content as a User Cookie, this way it doesn't matter which EC2 instance is being used the shopping cart is still saved as the cookie. This is a stateless solution but the HTTP requests are heavier, also cookies could be altered, so the EC2 instances should validate the cookies. Cookies can only be 4 KB large.
- Introduce a Server Session. The cookie only contains the session id. We can store the shopping cart session on ElastiCache, so the EC2 instances receive the web cookie with the session id and store/retrieve the session data (Shopping cart) from the ElastiCache
![[Stateful Server Session.png]]

Storing User Data
- We can scale reads by setting up and RDS master and then having read replicas with replication
- Alternatively we can scale reads by using lazy loading with ElastiCache. I.e. always try reading from the cache, if hit then good, if miss, read from the RDS DB and Store in cache for next time.
![[Stateful Store User Data.png]]
![[Stateful Webapp Security Groups.png]]
## MyWordPress.com
We are creating a fully scalable WordPress website. We want the website to access and correctly display images. 

We could choose to use aurora for better scaling and performance.
We can store images in EBS volumes for each instance. Works well for a single EC2 instance, but doesn't work for multiple. Instead we can use an EFS instead. This way we have ENI's in each AZ so the storage is shared between all the instances we have.
![[Wordpress example.png]]
## Instantiating Applications Quickly
EC2 Instances:
- Use a golden AMI: Install your applications, OS dependencies etc... beforehand and launch your EC2 instances from the Golden AMI
- Bootstrap using User Data: For dynamic configuration, use User Data scripts
- Hybrid: Mix golden AMI and User Data (useful on elastic beanstalk)
RDS Database:
- Restore from snapshot: schemas and data ready
EBS Volumes:
- Restore from snapshot: disk will be formatted and have data ready
## Elastic Beanstalk
Managed service to deploy solutions architectures. Most web apps have the same architecture of a ALB + ASG, so Beanstalk does this. We still have full control over the configuration. Beanstalk is free but you pay for underlying resources. 

Components
`Application`: Collection of Beanstalk components (environments, versions, configurations...)
`Application Version`: Iteration of your application code
`Environment`: Collection of AWS resources running an application version, i.e. test, production blah blah

![[Web Server vs Worker.png]]
Worker, can scale based on the number of SQS messages. Can push messages to SQS queue from another Web Server Tier.

Deployment Modes:
- Single instance, great for development
- High availability with load balancer, great for production

# Amazon S3 Buckets
---
## Lifecycle Rules
`Transition Actions`: configure objects to transition to another storage class.
- Move objects to standard IA class 60 days after creation
- Move to Glacier for archiving after 6 months
`Expiration actions`: configure objects to expire/delete after some time
- Can be used to delete old versions of files if versioning is enabled
- Can be used to delete incomplete Multi-part uploads
Rules can be specified for a certain prefix or for certain object tags  

S3 Analytics helps you decide when to transition objects to the right class. It has recommendations for Standard and Standard IA, doesn't work for One Zone IA or Glacier. S3 Analytics generates a daily report. It requires at least 24-48 hours to start seeing data analysis. 
## Requester Pays
In general, bucket owners pay for all Amazon S3 storage and data transfer associated with their bucket. With requester pays buckets, the requester instead of the bucket owner pays for the cost of the request and the data download from the bucket. The owner still pays for storage costs but all the networking costs and the data transfer are paid by the requester. The requester must be authenticated in AWS.
## S3 Event Notifications
Notifications can happen when an object is created, removed, restored, copied etc... Object name filtering is possible. Can create as many S3 events as desired. The notification can then be sent to other services to such as to SNS, SQS or to a Lambda Function.

IAM Permissions are required to send Notifications to services. For SNS an SNS Resource Access Policy is required, and similarly SQS Resource access Policy and Lambda Resource Policy are required.

Notifications can also be sent to Amazon EventBridge. Rules are defined within EventBridge which then sends the events to over 18 AWS services as destinations. EventBridge allows for: 
- Advanced filtering options with JSON rules (metadata, object size, name...)
- Multiple destinations - step functions, kinesis streams/firehose...
- EventBridge Capabilities - Archive, Replay Events, Reliable delivery
## S3 Performance
Baseline performance - S3 automatically scales to high request rates, latency 100-200ms. Your application can achieve at least 3500 PUT/COPY/POST/DELETE or 5500 GET/HEAD requests per second per prefix in a bucket. There are no limits to the number of prefixes in a bucket. 

`Multi-Part Upload`: 
- recommended for files > 100MB, must use for files > 5GB
- Parallelizes uploads
`S3 Transfer Acceleration`:
- Increases transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region
- Compatible with the multi-part upload.
- Used to speed up uploads
`S3 Byte Range Fetches`:
- Parallelize GETs by requesting specific byte ranges, i.e. a bunch of downloads similar to how multi-part does uploads.
- Better resilience in case of failure, if not found in a byte range, it tries a smaller one.
- Used to speed up downloads
## S3 Batch Operations
Perform bulk operations on existing S3 objects with a single request:
- Modify object metadata and properties
- Copy objects between S3 objects
- Encrypt unencrypted objects
- Modify ACLs, tags
- Restore objects from S3 Glacier
- Invoke Lambda functions to perform custom actions
A job consists of a list of objects, the action to perform and optional parameters. S3 Batch operations manages retires, tracks progress, sends completion notifications and generates reports, hence better than doing it yourself. You can use S3 inventory to get object list and use Athena to query and filter your objects.
## S3 Storage Lens
Service to understand, analyze and optimize storage across entire AWS organization. Discover anomalies, identify cost efficiencies and apply data protection best practices. Aggregate data for organization, specific accounts, regions, buckets or prefixes. Can use a default dashboard or create your own. Can be configured to export daily metrics to an S3 bucket.

`Summary Metrics`:
- General insights about your S3 storage, StorageBytes, ObjectCount...
- Used to identify fastest growing or not used buckets and prefixes
`Cost Optimization Metrics`:
- Insights to manage and optimize storage costs
- NonCurrentVersionStorageBytes, IncompleteMultipartUploadStorageBytes...
- Used to identify buckets with incomplete multipart uploads older than 7 days, identify which objects can be transitioned to lower cost storage classes
`Data Protection Metrics`:
- Insights for data protection features
- VersioningEnabledBucketCount, MFADeleteEnabledBucketCount, SSEKMSEnabledBucketCount...
- Used to identify buckets that aren't following data protection best practices
`Access Management Metrics`:
- Insights on S3 object ownership
- ObjectOwnershipBucketOwnerEnforcedBucketCount...
- Used to identify object ownership settings your buckets use
`Event Metrics`:
- Insights for S3 Event notifications
- EventNotificationEnabledBucketCount
- Identify which buckets have S3 Event Notifications configured
`Performance Metrics`:
- Insights into S3 Transfer Acceleration
- TransferAccelerationEnabledBucketCount
- Identify which buckets have S3 transfer acceleration enabled
`Activity Metrics`:
- Insights about how your storage is requested
- AllRequests, GetRequests, PutRequests, ListRequests, BytesDownloaded...
`Status Code Metrics`:
- Insight into HTTP Status codes
- 200OKStatusCount, 403ForbiddenErrorCount, 404NotFoundErrorCount...

### Free vs Paid
`Free`:
- Available for everyone
- Contains 28 usage metrics
- Data is available for queries for 14 days
`Advanced Metrics and Recommendations`:
- Paid metrics and features
-  Advanced Metrics:
	- Activity
	- Advanced Cost Optimization
	- Advanced Data Protection
	- Status Code
- CloudWatch Publishing - Access metrics in CloudWatch without additional charges
- Prefix level Aggregation
- Data is available for queries for 15 months
# S3 Security
---
## S3 Encryption
`Server Side Encryption (SSE)`:
- Server Side encryption with Amazon S3 Managed Keys (SSE-S3)
	- Encryption keys are handled, managed and owned by AWS
	- Object is encrypted server side
	- Encryption type is AES-256
	- Must set header "x-amz-server-side-encryption": "AES256", when uploading the file
	- Enabled by default for new buckets and new objects
- Server Side Encryption with KMS (SSE-KMS)
	- Encryption using keys handled and managed by AWS KMS
	- KMS allows for user control and audit key usage using CloudTrail
	- Object is encrypted server side
	- Must set header "x-amz-server-side-encryption": "aws:kms", when uploading the file
	- May be impacted by the KMS limits
	- When uploading a file, it calls the GenerateDataKey KMS API, and when downloading it calls the Decrypt KMS API
	- Both count towards the KMS quota per second, quota can be increased using Service Quotas Console
	- Can be throttled for high throughput
	- There is a new option called DSSE-KMS which is double encryption with KMS
- Server Side Encryption with customer keys (SSE-C)
	- Encryption using customer managed keys outside of AWS
	- S3 does not store the encryption keys you provide
	- HTTPS must be used
	- Encryption key must be provided in HTTP headers for every HTTP request made
`Client Side Encryption`:
- Use client libraries such as Amazon S3 Client Side Encryption Library
- Clients must encrypt data before sending to S3
- Clients must decrypt data when retrieving from S3
- Customer fully manages keys and encryption cycle
`Encryption in transit (SSL/TLS)`:
- Amazon S3 exposes 2 endpoints,
	- HTTP Endpoint - non encrypted
	- HTTPS Endpoint - encryption in flight
- HTTPS is recommended
- HTTPS is mandatory for SSE-C
- Most clients would use the HTTPS endpoint by default
- We can force encryption in transit using a bucket policy, aws:SecureTransport 
## Cross Origin Resource Sharing (CORS)
Origin = scheme (protocol) + host (domain) + port
	Example: https://www.example.com (implied port is 443 or 80)
CORS is a web browser based mechanism to allow requests to other origins while visiting the main origin.
Same origin: http://example.com/app1 & http://example.com/app2
Different origin: http://www.example.com & http://other.example.com
The requests won't be fulfilled unless the other origin allows for the requests using CORS headers. 
If a client makes a cross origin request on our S3 bucket, we need to enable the correct CORS headers via Access-Control-Allow-Origin: `origin`
## MFA Delete
Multi Factor Authentication to force users to generate a code on a device before doing important operations on S3. Only the bucket owner (root account) can enable/disable MFA Delete. When enabled, MFA is required to permanently delete an object version and to suspend versioning on the bucket. MFA won't be required to enable versioning or list deleted versions. To use MFA Delete, versioning must be enabled on the bucket.
## S3 Access Logs
For audit purposes, you may want to log all access to S3 buckets. Any request made to S3, from any account, authorized or denied will be locked into another S3 bucket (logging bucket). Data can then be analyzed using data analysis tools like Athena. Target logging bucket must be in the same AWS region. 

Warnings:
- Never set your logging bucket to be the monitored bucket, will create a logging loop and the bucket will grow exponentially
## S3 Pre Signed URLs
Generate pre signed URLs using the S3 console, AWS CLI or SDK
URL Expiration:
- S3 console - 1m up to 12 hours
- AWS CLI - default 3600s max 604800s ~ 168 hours
Users given a pre signed URL inherit the permissions of the user that generated the URL for GET/PUT. Essentially used to temporarily give permissions to someone.
## S3 Glacier Vault Lock
Used to adopt a WORM (Write Once Read Many) model. You need to create a Vault Lock Policy and then you lock the policy for future edits (can no longer be changed or deleted). Helpful for compliance and data retention.
## S3 Object Lock
Also used to adopt a WORM model. Used to block an object version deletion for a specified amount of time, i.e. locks the object. Retention period is user set and can be extended.

There is also an option for 'Legal Hold', which protects the object indefinitely, independent from the retention period. A user with the proper IAM permission can freely place and remove legal holds.

Retention modes:
`Compliance`:
- Object versions can't be overwritten or deleted by any user, including root user
- Object retention modes can't be changed and retention periods can't be shortened
`Governance`:
- Most users can't overwrite or delete an object version or alter its lock settings
- Some users have special permissions to change retention or delete object
## S3 Access Points
Access points simplify security management for S3 buckets. They have their own DNS name and an access point policy (similar to bucket policy).
![[Access Point.png]]
We can define access point to be accessible only from within the VPC. To do so you must create a VPC endpoint to access the access point. The VPC Endpoint policy must allow access to the target bucket and access point. 

## S3 Object Lambda
Say we want to use an AWS Lambda function to change the object before it is retrieved by the caller application, we use Object Lambda. Only one S3 bucket is needed, on top of which we create an S3 access point and S3 Object Lambda Access Points.
![[Object Lambda.png]]
# CloudFront and Global Accelerator
---
## CloudFront Overview
Content delivery network that improves read performance by caching content at edge locations. It uses hundreds of points of presence globally (edge locations, caches), because it is global, it has built in DDoS protection, and it integrates with Shield and WAF.

CloudFront has several potential origins:
- S3 bucket, distribute files and cache at the edge and capability of uploading files to S3 via CloudFront. Secured using Origin Access Control (OAC)
- VPC Origin, for applications hosted in a VPC private subnet, ALB, NLB, EC2 instances
- Custom HTTP Origin, a statis S3 website, or any public HTTP backend
## ALB or EC2 as an origin
Recommended way is to use VPC Origins. It allows you to deliver content from your applications hosted in your VPC private subnets. CloudFront will direct traffic to the VPC Origin and the VPC origin will then interact with the ALB or EC2 instances. 

The other way but older is to use a public network. You would have the ALB or EC2 instance be public using a security group that would allow the public IPs from CloudFront. Tedious and potentially risky due to human error.
## Geo Restrictions
Allows restricting who can access your distribution based on geographical location. There is an Allowlist or a Blocklist. The 'country' of the user is determined via a 3rd party Geo-IP database. A use case could be copyright laws.
## Price Classes
The cost of data out per edge location varies depending on which edge location. 
You can choose to reduce the number of edge locations to reduce the costs. 
`Price Class All`: All regions, best performance and most expensive
`Price Class 200`: Most regions, but excludes the most expensive regions
`Price Class 100`: Only the least expensive regions
## Cache Invalidations
In case you update the backend origin, CloudFront does not update the cached content immediately, only will do so after the TTL has expired. However we can force an entire or partial cache reset by performing a CloudFront Invalidation. You can invalidate all files or a specific path.
## Global Accelerator
Say you have an application that has global users who want to access it. As they go over the public internet, they may experience a lot of latency due to a lot of router hops. 

`Unicast IP`: One server holds one IP address
`Anycast IP`: All servers hold the same IP address and the client is routed to the nearest one.

Global Accelerator fixes this by implementing Anycast IP. It leverages the internal AWS network to route to your application, so instead of going over the public internet, it will send it to an edge location which then routes it internally. Works with Elastic IP, EC2 instances, ALB, NLB, public or private. 

Features
`Consistent Performance`:
- Intelligent routing to lowest latency and fast regional failover
- No issue with client cache because the IP doesn't change
- Uses internal AWS network
`Health Checks`:
- Global Accelerator performs a health check of your applications
- Helps make your application global
- Great for disaster recovery
`Security`:
- Only 2 external IP need to be whitelisted, the 2 anycast ones
- DDoS protection via implementation with AWS Shield 

Overall Global Accelerator is useful for non HTTP use cases such as gaming (UDP), IoT (MQTT) or VoIP. Can also be used for HTTP cases that require a static IP address, or deterministic fast regional failover.
# Amazon Storage Extras
---
## Snowball
Highly secure and portable device to collect and process data at the edge and migrate data in and out of AWS. Used for transfers of up to petabytes of data. It is an offline device that is shipped to AWS.

Can also be used to process data as its being created on an edge location, such as a truck on a road, a ship on the sea or a underground mining station. Places with limited internet and computing power. A snowball edge device can be set to do edge computing, run EC2 instances or Lambda functions at the edge.

Snowball cannot import to Glacier directly, instead we must first import to an S3 bucket and then use lifecycle policies to move it to Glacier
## Amazon FSx
Managed service to launch 3rd party high performance file systems on AWS. 

`FSx for Windows File Server`:
- Managed Windows file system share drive
- Supports SMB protocol and Windows NFTS
- Microsoft Active Directory Integration, ACLs, user quotas
- Can be mounted on Linux EC2 instances
- Supports Microsoft's Distributed File System (DFS) Namespaces to group files across multiple FS
- Scales up to 10s of GB/s, millions of IOPS and 100s of PB of data
- Supports SSD (latency sensitive workloads) and HDD (general workload)
- Can be accessed from on premise infrastructure via VPN or Direct Connect
- Can be configured to be Multi AZ
- Data is backed up daily to S3
`FSx for Lustre`:
- Lustre is a type of parallel distributed file system for large scale computing (Linux Clusters)
- Used for Machine learning and high performance computing
- Scales up to 100s of GB/s, millions of IOPS and sub ms latencies
- Supports SSD (low latency, IOPS intensive workloads, small and random file operations) and HDD (throughput intensive workloads, large and sequential file operations)
- Seamless integration with S3, can read write to S3 as a file system
- Can be used on premise via VPN or direct connect
### Deployment Options for Lustre
Scratch File System:
- Temporary storage
- Data is not replicated, doesn't persist if file server fails
- High bursting
- Used for short term processing and optimize costs
Persistent File System:
- Long term storage
- Data is replicated within same AZ
- Replace failed files within minutes
- Used for long term processing and sensitive data

`FSx for NetApp ONTAP`:
- Managed NetApp ONTAP
- File system compatible with NFS, SMB, iSCSI protocols
- Used to move workloads running on ONTAP or NAS to AWS
- Works with Linux, Windows, MacOS, VMware Cloud, Amazon Workspaces and AppStream 2.0, EC2, ECS and EKS.
- Storage shrinks or grows automatically
- Features snapshots, replication and data de-duplication
- You can do point in time instantaneous cloning, useful for testing new workloads
`FSx for OpenZFS`:
- Managed OpenZFS file system
- Compatible with NFS
- Used to move workloads running on ZFS to AWS
- Up to 1 million IOPS with < 0.5ms latency
- Snapshots, compression
- Point in time instantaneous cloning
## Storage Gateway
Bridges on premises data and cloud data. Use cases include disaster recovery, backup and restore, tiered storage, data stored on cloud but using on premise as cache. Storage gateway has to be installed in your on premise data centers. 

`S3 File Gateway`:
- Allows access to S3 buckets as a file system on premise
- Configured buckets are accessible using NFS and SMB protocol
- Most recently used data is cached in the file gateway
- Supports S3 Standard, Standard IA, One Zone IA, Intelligent Tiering. Doesn't support Glacier, for that use lifecycle policy
- Bucket access using IAM roles for each gateway
- SMB protocol has integration with Active Directory for user authentication
`Volume Gateway`:
- Block storage using iSCSI protocol backed by S3
- Backed by EBS snapshots which can help restore on premise volumes
- 2 options:
	- Cached volumes: low latency access to most recent data
	- Stored volumes: entire dataset is on premise, scheduled backups to S3
- Overall, used for backups
`Tape Gateway`:
- Some companies have backup processes using physical tapes
- With Tape Gateway, companies can have the same processes but in the cloud
- Uses Virtual Tape Library (VTL) backed by S3 and Glacier
- Backup data using existing tape based processes and iSCSI interface
- Works with leading backup software vendors
![[Storage Gateways.png]]
## AWS Transfer Family
Managed service for file transfers into and out of S3 or EFS using the FTP protocol. Supports FTP, FTPS or SFTPS. Scalable, reliable and highly available. Pay per provisioned endpoint per hour and data transfers in GB. You can store and manage Transfer Family credentials within itself and can integrated with existing authentication systems. 
## AWS DataSync
Used to synchronize data. Moves large amounts of data to and from:
- On premises/other cloud to AWS - requires agent
- AWS to AWS - no agent needed
Can synchronize to S3, EFS, FSx. Replication tasks for DataSync are not continuous, but rather scheduled hourly, daily or weekly. File permissions and metadata are preserved. Others don't do this. 
## Storage Comparison
![[Storage Comparison.png]]
# Decoupling Applications
--- 
## Simple Queueing Service (SQS)
Producers send messages into the SQS queue. Consumers poll messages from the SQS. There can be multiple producers or consumers.

### Standard Queue:
- Used to decouple applications
- Unlimited throughput, unlimited messages in the queue
- Messages are shortlived, by default retention of messages 4 days, maximum 14 days. Messages have to be polled during that time otherwise it is lost
- Low latency, < 10ms on publish and receive
- Limitation of 1024 KB per message
- Can have duplicate messages (at least once delivery)
- Can have out of order messages (best effort ordering)

Producers send messages using the SDK (SendMessage API). Message is persisted in SQS until a consumer deletes it, taking into account retention.
Consumers (running on EC2 instances, servers, Lambda or else) polls the SQS for messages, can receive up to 10 messages at a time. Consumers then process the messages (for example insert the message into RDS). Messages are then deleted using the DeleteMessage API from the SDK, which guarantees that no other consumer will see this message.

The advantage of using SQS is that if we want to scale horizontally, we can add consumers to improve throughput of message processing, perfect use case for using auto scaling groups. Can set up a CloudWatch metric monitoring the queue length which triggers a CloudWatch Alarm to increase the scaling of the ASG. Another use could be to decouple database writes. Sometimes, if the load to write is too large, some transactions may be lost. Instead we can have an enqueue ASG to send messages to an SQS, and then a dequeue ASG to receive messages that are then inserted into the database. Its should only be used for applications that do not require confirmation that the data has been inserted.

`Encryption`:
- In flight using HTTPS API
- At rest using KMS keys
- Client side encryption if the client wishes to manage it 
`Access Controls`: 
- IAM policies to regulate access
`SQS Access Policies`:
- Similar to S3 bucket policies
- Useful for cross account access to SQS queues
- Or for allowing other services to write to an SQS queue
### FIFO Queue
First in First out, so messages are sent and polled sequentially. Using this kind of queue there is a limited throughput, 300 messages per second without batching and 3000 with batching. Capability for "Exactly once send", duplicates are removed at the queue level using Deduplication ID (messages will have them attached), deduplication in a 5 minute window. Capability for ordering by Message Group ID.

### Message Visibility Timeout
After a message is polled, it becomes invisible to other consumers. By default the time out is 30s, during which the message has 30s to be processed. If a message is not processed within the timeout, it might be processed multiple times, i.e. after its visible again. A consumer could call the ChangeMessageVisibility API to get more time. If timeout too long and consumer crashes, reprocessing takes very long. If timeout too low, there may be duplicates.
### Long Polling
When a consumer requests messages from the queue, it can optionally wait for messages to arrive if there are none in the queue. Long Polling decreases the number of API calls made to SQS while increasing the efficiency and reducing latency of the application. Long polling can be set from 1 to 20 seconds. Overall it is preferable to short polling.
## Simple Notification Service (SNS)
Used to send one message to many receivers. Uses a publication/subscription model. The event producer only sends messages to one SNS topic, the event receivers listen to the SNS topic for notifications. Each subscriber to the topic will get all the messages. Up to 12,500,000 subscriptions per topic with 100,000 topics limit.

To publish we have 2 methods:
`Topic Publish`:
- Uses the SDK
- Create a topic
- Create a subscription or many
- Publish to the topic
`Direct Publish`:
- For mobile apps SDK
- Create a platform application
- Create a platform endpoint
- Publish to the platform endpoint
- Works with Google GCM, Apple APNS, Amazon ADM...

SNS has the same security offerings as SQS.

## SNS + SQS Fan Out 
Say we need to push messages to many SQS queues. Instead we put once to an SNS, and receive from all the SQS queues that are subscribers. Fully decoupled model with no data loss. SQS allows for data persistence, delayed processing and retries of work. Allows to add more SQS subscribers over time. Works with SQS queues in other regions.

Only 1 S3 event rule can be defined for each combination of event type and prefix. So if you want to send the same S3 event to many SQS queues, use fan out.

Can also do a SNS FIFO + SQS FIFO fan out.

SNS can also do message filtering via a JSON policy used to filter the messages sent to subscribers. If a subscription doesn't have a filter policy, it receives every message.
## Kinesis Data Streams
Used to collect and store streaming data in real-time. Data that is created and used on the spot. To send data to a Kinesis Data Stream, we need a producer, which can be a custom build application or a kinesis agent. Kinesis Data Stream will send the real time data to consumers, which could be an application, Lambda, Data Firehose, or Managed Service for Apache Flink. 

Kinesis offers the following features:
- Retention up to 365 days
- Ability to replay data by consumers
- Data can't be deleted from Kinesis, have to wait for it to expire
- Data up to 1 MB, typical use case is lots of small real time data
- Data ordering guarantee for data with the same Partition ID
- At rest KMS encryption, in flight HTTPS encryption
- Can use the Kinesis Producer Library (KPL) to write an optimized producer applications
- Can use the Kinesis Client Library (KCL) to write an optimized consumer application

Capacity modes:
`Provisioned mode`:
- Allows you to choose the number of shards
- Each shard gets 1 MB/s in
- Each shard gets 2 MB/s out
- Scale manually to increase or decrease the number of shards
- You pay for each provisioned shard per hour
`On demand mode`:
- No need to provision or manage capacity
- Default capacity provisioned 4MB/s
- Scales automatically based on observed throughput peak during the last 30 days
- Pay per stream per hour and data in/out in GBs
## Data Firehose
![[Firehose.png]]Fully Managed service to send data to target sources. It has automatic scaling, is serverless and has pay for what you use. It is a Near Real Time service, with buffering capability based on size/time. Data is buffered and then sent in batches, hence Near Real Time. Supports CSV, JSON, Parquet, Avro, Raw Text, Binary data. Allows conversions to Parquet/ORC, compressions with gzip/snappy. Also allows custom transformations using Lambda.
## Kinesis Data Stream vs Data Firehose
`Kinesis`:
- Streaming data collection
- Producer and consumer code
- Real time
- Provisioned/on demand
- Data storage up to 365 days
- Replay capability
`Firehose`:
- Load streaming data into other services, AWS or 3rd party/custom
- Fully managed and serverless
- Near real time
- Automatic scaling
- No data storage
- No replay capability
## Amazon MQ
SQS and SNS are cloud native services running on proprietary AWS protocols. However, traditional applications may be running from on premises using open protocols such as MQTT, AMQP, STOMP, Openwire, WSS. When migrating to the cloud, instead of re-engineering the application to use SQS and SNS, we can use Amazon MQ. 

It is a managed message broker service for RabbitMQ and ActiveMQ. It doesn't scale as much as SQS and SNS though. Amazon MQ runs on servers and can run in Multi AZ with failover. It also has both a queue feature to mimic SQS and topic features to mimic SNS. 

To setup failover, we need to mount the active and standby MQ brokers to an EFS. When the active fails, standby takes over.
# Containers on AWS
---
## Elastic Container Service (ECS)
Used to launch Docker containers on AWS. When launching a container, it essentially launches an ECS task on ECS clusters, ECS clusters can be of many types as we will see. AWS takes care of starting and stopping containers

`EC2 Launch Type`: You must provision and maintain the EC2 instances. Each EC2 instance must run the ECS agent to register in the ECS cluster.
`Fargate Launch Type`: Serverless, so no need to provision the infrastructure. You just create task definitions and AWS runs the ECS Tasks automatically. Uses Fargate.

IAM roles should be used for ECS. Roles should be given to the ECS agent for the necessary API calls, and a role for each ECS task to allow each role to have access to different ECS services you may run, for example one -> S3, two -> DynamoDB. 

ECS has integrations with load balancers. In general we can run an application load balancer for load balancing. Network load balancer can also be used for high throughput/performance use cases or to pair with an AWS Private Link. ECS clusters can also have mounted EFS.

For ECS auto scaling we can use AWS Application Auto Scaling to automatically increase/decrease the desired number of ECS tasks. We can set it to scale on:
- CPU utilization of ECS
- Memory Utilization of ECS
- ALB Request Count per Target - metric from ALB

We can use Target Tracking, scale based on target value for a specific CloudWatch metric. Step Scaling, scale based on a specified CloudWatch Alarm and Scheduled scaling, scale based on a specified time/date. Keep in mind ECS autoscaling is at the task level, whereas EC2 auto scaling is at the instance level. 

To scale when using EC2 launch type, we can scale the ASG, or we can use the newer option ECS Cluster Capacity Provider. 
- Automatically provision and scale infrastructure for ECS Tasks
- Capacity provider paired with an auto scaling group
- Adds EC2 instances when you're missing capacity
## Elastic Container Registry
Store and manage Docker images on AWS. We can store images privately, or publicly on public galley. Fully integrated with ECS and backed by S3. On top of being a repo, it supports image vulnerability scanning, versioning, image tags, image lifecycle...
## Elastic Kubernetes Service
Managed service to launch Kubernetes Clusters on AWS. Kubernetes is an open source system for automatic deployment, scaling and management of containerized applications, usually docker. It is an alternative to ECS. Supports EC2 launch mode and Fargate. Use case for EKS if your company is already using Kubernetes and wants to migrate to AWS. EKS works by deploying nodes, which is a group of instances running on an AZ.

`Managed Node Groups`:
- Creates and manages nodes for you
- Nodes are part of an ASG managed by EKS
- Supports on demand and spot instances
`Self Managed Nodes`:
- Nodes created by you and registered to the EKS cluster and managed by an ASG
- You can use prebuilt AMI - EKS Optimized AMI
- Supports on demand and spot instances
`Fargate`:
- Managed and serverless

`Data Volumes`
Need to specify StorageClass manifest on your EKS cluster. Leverages a Container Storage Interface (CSI) compliant driver. Supports EBS, EFS, FSx for Lustre and FSx for NetApp ONTAP
## App Runner
Managed service to deploy web applications and APIs at scale. No infrastructure required. Start with your source code or container image, then configure basic settings for the web application such as the CPU, RAM, auto scaling, health checks. Then App Runner automatically builds and deploys the web app. Automatic scaling, highly available, load balancing, encryption, and application can have VPC access support. So it can connect to databases, cache and message queue services.
## App2Container
CLI tool for migrating and modernizing Java and .NET web apps into Docker Containers. Used for Lift and shift migrations, where apps are running in on premise machines. No need to change code, allows to migrate legacy apps. Generates CloudFormation for the compute/network. And app is registered as a Docker container to ECR. Then can be deployed to ECS, EKS or App Runner. Supports pre built CI/CD pipelines. 

Very likely only basic knowledge is needed. Name says enough.
# Serverless Services
---
## AWS Lambda
Managed service for serverless functions. Supports basically every language and for the ones that aren't there are custom runtimes available. Integrates with:
- API Gateway to create and easily publish APIs
- Kinesis, for data transformation
- DynamoDB for creating triggers, when something happens lambda is triggered
- S3 triggers
- CloudFront, Lambda@Edge
- And others

`Limitations to Know - per region`
Execution: 
- Memory allocation, 128 MB - 10GB 
- When memory is increased, we have more available vCPU
- Max execution time 15minutes
- Environment variables up to 4KB
- Temporary space for bigger files, disk capacity in /tmp 512 MB to 10GB
- Concurrent executions is 1000
Deployment:
- Lambda function for compressed .zip is 50MB
- For uncompressed, deployment max size is 250 MB
- Can use /tmp to load other files at startup
- Size of environment variables: 4KB

`Concurrency and Throttling`
Concurrent limit of up to 1000 concurrent executions. We can set a reserved concurrency set at the function level which limits the number of concurrent executions. Each invocation over the concurrency limit will trigger a throttle, with 2 different behaviours.
- Synchronous invocation -> return Throttle Error - 429
- Asynchronous invocation -> retry automatically and then go to dead letter queue. Continues retrying up to 6 hours with an exponential backoff time, from 1s to a maximum of 5 minutes
The concurrency limit is for your account, so it is important to reserve the limits otherwise if one of your applications gets overloaded, i.e. it scales up to 1000 concurrent executions, then if we have another application, that one will be throttled.

Use provisioned concurrency to avoid cold starts (initialization can take time, so first request served by a new instance has higher latency). Concurrency is allocated before in advance before the function is invoked, which can also be managed by application auto scaling.

`Lambda SnapStart`
Improves your Lambda functions' performance up to 10x for free for Java, Python and .NET. SnapStart basically pre-initializes the functions, so no need to initialize the functions from scratch on every invocation. When you publish a new version of your function, Lambda initializes it, takes a snapshot of memory and disk state of the function, and snapshot is cached for low latency access.

By default, Lambda is launched outside your own VPN in an AWS VPC. Therefore it cannot access resources in your VPC. To allow it to access your resources, you can launch it in your VPC. You must define the VPC ID, the subnets and security groups. Lambda will create an ENI in your subnets. A common use case is to launch in your VPC to pool Lambda RDS connections with RDS Proxy.x
## Functions at the Edge
Many applications execute some form of logic at the edge. We can use Edge Functions to attach to CloudFront distributions to do this. This way it runs close to the users to minimize latency. We can do this 2 ways.

CloudFront Functions
- Lightweight functions written in JavaScript
- For high scale, latency sensitive CDN customizations
- Sub ms startup times and supports millions of requests/second
- Used to change viewer requests (after CloudFront receives a request from viewer) and responses (before CloudFront forwards response to viewer)
- Mostly for very simple functions, for more complex ones use Lambda@Edge
Lambda@Edge
- Written in NodeJS or Python
- Scales to 1000s of requests/second
- Used to change viewer requests and responses
- But also to change Origin requests (before CloudFront forwards request to origin) and Origin responses (after CloudFront receives the response from origin)
- Author the functions in one AWS region and then CloudFront replicates to its locations
- Overall Lambda@Edge is better for longer functions that require more memory and/or package size.
## DynamoDB
Fully managed, highly available with replication across multi AZ, NoSQL DB. Scales to massive workloads, millions of requests per second, trillions of rows, 100s of TB of storage. Single-digit latency, integrated with IAM for security, authorization and administration. It is low cost and has auto scaling capabilities. It is always available and there is no maintenance or patching required.

DynamoDB is made of Tables
- Each table has a primary key
- Each table can have an infinite number of items (rows)
- Each item has attributes (columns) which can be added over time and can be null
- Maximum size of an item is 400KB
- Supports Scalar Types (string, number, binary, boolean, null), Document Types (List, Map) and Set Types (String Set, Number Set, Binary Set)
- DynamoDB can rapidly evolve schemas
There are Standard tables and IA tables. There are also global tables that are 2 way replicated across regions. To use this, must enable DynamoDB Streams.

Read/Write Capacity modes
`Provisioned Mode (default)`:
- Specify number of reads/writes per second
- Plan capacity beforehand
- Pay for provisioned Read Capacity Units (RCU) and WCU
- Can add auto scaling mode for RCU and WCU
`On Demand Mode`:
- Automatic scaling of read/writes based on workload
- No capacity planning needed
- Pay for what you use, much more expensive though
- Great for unpredictable workloads and steep spikes
## DynamoDB Advanced Features
### DynamoDB Accelerator (DAX)
Managed, highly available, seamless in memory cache for DynamoDB. Fully compatible with existing DynamoDB APIs. By default has 5 minute TTL. DAX is useful for caching individual objects, while ElastiCache should be used to store larger queries, for example you calculated a long computation and you store the result on ElastiCache.
### Stream Processing
Used to stream real time data, create/update/delete. Similar to Kinesis Data Stream
### Disaster Recovery
DynamoDB supports continuous backups using point in time recovery PITR, optionally enabled for the last 35 days. Can recover to any time within the backup recovery. The recovery process creates a new table.

There are also on demand backups, i.e. full backups for long term retention until explicitly deleted. No effect on performance or latency. Can be configured and managed in AWS Backup.
### Integration with S3
Can export directly to S3 but needs PITR enabled. Works for any point of time in the last 35 days. Can be used to perform data analysis on top of DynamoDB by exporting the data. Can retain snapshots for auditing. Can do ETL on S3 data before importing back to dynamo. Supports JSON or ION format.

Similarly can import from S3, supports CSV, DynamoDB JSON or ION. Doesn't consume any write capacity and creates a new table. Import errors are logged in CloudWatch Logs.
## API Gateways
Serverless service to create REST APIs. 
![[API Gateway example.png]]
Supports WebSocket Protocol. Handles API versioning and also handles different environments (dev, test, prod). Handles security, authentication and authorization. Creates API keys and handles request throttling. Can use standards such as Swagger/Open API to quickly define APIs. Transforms and validates requests and responses. Generate SDK and API specifications. Cache API responses.

Integrations
`Lambda`:
- Easy way to expose Rest API backed by Lambda
`HTTP`:
- Expose HTTP endpoints in the backend.
- Useful to add rate limiting, caching and other features
`AWS Service`:
- Any AWS API through the gateway.
- For example a kinesis data stream

Endpoint Types
`Edge Optimized (default)`:
- Requests are routed through CloudFront Edge locations
- API Gateway still lives in only 1 region
`Regional`:
- For clients within the same region
- Could manually combine with CloudFront, this way allows for more control over caching strategy and distribution
`Private`:
- Only accessible from your VPC using an interface VPC endpoint (ENI)
- Use a resource policy to define access

Security
`User Authentication Through`:
- IAM roles - internal applications
- Cognito - external users
- Custom Authorizer
`Custom Domain Name HTTPS via AWS Certificate Manager`:
- If using Edge Optimized endpoint, then the certificate must be in us-east-1
- If using regional endpoint, the certificate must be in the API Gateway region
## AWS Step Functions
Used to build serverless visual workflows to orchestrate your Lambda Functions. Features: sequencing, parallel, conditions, timeouts, error handling... Can integrate with EC2, ECS, on premise servers, API Gateway, SQS queue, etc... Possibility of implementing human approval feature.
## Amazon Cognito
Give users an identity to interact with our Web or mobile application. 
`Cognito User Pools`:
- Create a serverless database of users for your web and mobile apps
- Simple login, username or email/password combination
- Password resets
- Email and Phone number verification
- MFA
- Integrates with Identity Pools
`Cognito Identity Pools`:
- Provide AWS credentials to outside users so they can access AWS resources directly
- Users source can be Cognito User Pools, 3rd party logins, etc...
- Users can then access AWS directly or through an API Gateway
- IAM policies are applied to the credentials and are defined in Cognito
- They can be customized based on the user_id for more control
- There are default IAM roles for authenticated and guest users
# Serverless Architectures
--- 
## Mobile App: To do List
- Expose REST API with HTTPS
- Users can directly interact with their own folder in S3
- Users should authenticate through a managed serverless service
- Users can write and read to dos, but they mostly read them
- Database should scale and have high read throughput
![[Todolist.png]]
## Hosted Website: Blog
- Website should scale globally
- Blogs are rarely written, but often read
- Most of the website is purely static files, some of it may be dynamic REST API
- Caching must be implemented where possible
- Any new users that subscribes should receive a welcome email
- Any photo uploaded to the blog should have a thumbnail generated
![[Blog.png]]For the email, we can use DynamoDB streams to stream any new users, have it invoke a lambda function to then send an email via SES (Simple Email Service).
## Micro Services Architecture
Not exactly serverless. 
- Many services interact with each other directly using a REST API
- Each architecture for each microservices may vary in form and shape
- We want a microservice architecture so we can have a leaner development lifecycle for each service
![[Microservices.png]]
- Each microservice can be designed as you want
- Synchronous patterns: API Gateway, Load Balancer
- Asynchronous patterns: SQS, Kinesis, SNS, Lambda S3 Triggers

Challenges with microservices:
- Repeated overhead for creating each microservice
- Issues with optimizing server density/utilization
- Complexity of running multiple versions of multiple microservices simultaneously
- Proliferation of client side code requirements to integrate with many separate microservices
Some of these challenges are solved by Serverless Patterns:
- API Gateway, Lambda scale automatically and you pay per usage
- Can easily clone API, reproduce environments
- Generated client SDK through Swagger integration for the API Gateway
## Software Updates Offloading
- We have an EC2 application that distributes software updates once in a while
- When a new software update is out, we get a lot of requests and the content is distributed in mass over the network, its very costly
- We don't want to change our application, but want to optimize our cost and CPU usage
- Just use CloudFront to cache the software update files at the edge since software updates are static files
# Machine Learning
--- 
`AWS Rekognition`:
- Finds objects, people, text, scenes in images and videos using ML
- Has option for content moderation, set a minimum confidence threshold for items that will be flagged
- Flag sensitive content for manual review in Amazon Augmented AI (A2I)
`AWS Transcribe`:
- Automatically convert speech into text
- Uses a deep learning process called automatic speech recognition (ASR)
- Automatically remove Personally Identifiable Information (PII) using Redaction
- Supports automatic language identification for multi-lingual audio
`AWS Polly`:
- Turns text into lifelike speech using deep learning
- Can use Lexicon and SSML
- Customize pronunciation of words with pronunciation lexicons (sort of rules for special cases)
- Generate speech from documents marked up with Speech Synthesis Markup Language (SSML) which enables more customization such as emphasizing certain words, phonetic pronunciation, whispering, newscaster speaking style etc... 
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
- Comprehend Medical detects and returns useful information in unstructured clinical text:
	- Physician notes
	- Discharge summaries
	- Test results
	- Case notes
- Uses NLP to detect protected health information
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
# CloudWatch (Monitoring)
---
CloudWatch provides metrics for every service in AWS. 
Metrics also have "metadata" called dimensions, which are attributes of a metric, could be instance id, environment etc... Each metric can have up to 30 dimensions and each metric has a timestamp. You can also set up custom metrics. CloudWatch metrics can be streamed to a destination of your choice. 
## CloudWatch Logs
Stores application logs in AWS. To do so, we must first define Log Groups with an arbitrary name, usually representing an application. Then within each group we have multiple log streams which are sequences of log events that come from the same source. We can then define the log expiration policy. We can send CloudWatch Logs to
- S3
- Kinesis Data Streams
- Firehose
- Lambda
- OpenSearch
Logs are encrypted by default and can be setup to use KMS encryption with your own keys.

Sources:
- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
- Elastic Beanstalk: application logs
- ECS: collection from containers
- Lambda: Function logs
- VPC Flow Logs
- API Gateway
- CloudTrail based on filter
- Route53: DNS queries

CloudWatch Logs Insights allows you to perform queries on your logs and visualize them. Helps you search and analyze log data stored on CloudWatch Logs. Uses a purpose built query language that automatically discovers fields from AWS services and JSON log events. Can also query multiple Log groups in different AWS accounts. Importantly, it is a query engine, not a real time engine.

CloudWatch can be exported into many destinations. The first of which is S3 export. Log data can take up to 12 hours to become available for export, and as such is not real time. If we want real time or near real time, we use Logs Subscriptions instead. This allows you to get a real time stream of log events from CloudWatch Logs. Can send to Kinesis Data Streams, Data Firehose or Lambda. Can filter which logs are delivered to destination.

By default, no logs from EC2 instances will go to CloudWatch. To get those logs, we need to run a CloudWatch agent on EC2 to push the log files. The EC2 instances must have proper IAM roles to push to CloudWatch. Similarly CloudWatch agent can also be used for on premises servers. There are 2 types of agents:
`CloudWatch Logs Agent`:
- Old version
- Can only send to CloudWatch Logs
`CloudWatch Unified Agent`:
- Collects additional system level metrics such as RAM, processes etc... and at a much finer detail compared to Logs Agent.
- Collects logs to send to CloudWatch Logs
- Centralized configurations using SSM Parameter Store
## CloudWatch Alarms
Alarms are used to trigger notifications for any metric. There are various options such as sampling, %, max, min, etc... There are 3 alarm states:
- OK
- INSUFFICIENT_DATA
- ALARM
You set up the period of the alarm, the length of time to evaluate the metric. There are 3 main targets for alarms: 
- Stop, Terminate, Reboot or Recover an EC2 instance
- Trigger Auto Scaling Action
- Send Notifications to SNS (from which you can do anything you want such as trigger a Lambda function)
Alarms can be manually triggered for testing purposes via the CLI.
## Composite Alarms
CloudWatch alarms are on a single Metric. Composite Alarms instead monitor the states of multiple other Alarms. It uses AND and OR conditions. Helpful to reduce alarm noise by creating complex composite alarms. 
## CloudWatch Insights
### Container Insights
Collect, Aggregate, summarize metrics and logs from containers. Available for containers on ECS, EKS, Kubernetes platforms on EC2 and Fargate. In EKS and Kubernetes on EC2, it uses a containerized version of the CloudWatch agent to discover containers.
### Lambda Insights
Monitoring and troubleshooting solution for serverless applications running on Lambda. Collects, aggregates and summarizes system level metrics (CPU, memory, network...) and diagnostic information (cold starts, worker shutdowns...). Lambda Insights is provided as a Lambda Layer.
### Contributor Insights
Analyze log data and create time series that display contributor data. This helps you find top talkers and understand who or what is impacting system performance. Works for any AWS generated logs. For example finding bad hosts, heaviest network users or most error prone URLs. It also provides built in rules that you can use to analyze metrics from other AWS services.
### Application Insights
Provides automated dashboards that show potential issues with monitored applications to help isolate ongoing issues. Powered by SageMaker. Gives you enhanced visibility into your application health to reduce the time it will take you to troubleshoot and repair your applications. Findings and alerts are sent to EventBridge and SSM OpsCenter.
# CloudTrail (Auditing Actions)
---
Provides governance, compliance and audit for your AWS account. It is enabled by default. It allows you to get a history of events and API calls made within your AWS account by:
- Console
- SDK
- CLI
- AWS Services
You can put the logs from CloudTrail into CloudWatch Logs or S3. A trail can be applied to All regions (default) or to a single Region. Who did what and when.
## CloudTrail Events
`Management Events`:
- Operations that are performed on resources in your AWS account, configuring security, rules for routing data, setting up logging etc...
- By default trails are configured to log management events
- Can separate Read Events (don't modify) from Write Events (may modify resources)
`Data Events`:
- By default, data events are not logged because they have a high volume of operations
- Amazon S3 object level activity, can separate Read and Write Events
- AWS Lambda function execution activity
`CloudTrail Insights`:
- You have to enable it and pay for it
- Detects unusual activity in your account
- CloudTrail Insights analyzes normal management events to create a baseline and then continuously analyzes write events to detect unusual patterns 

Events are stored by default for 90 days. To keep events beyond this period, log them into S3, then analysis can be done using Athena.
# AWS Config (Compliance of Configurations)
Service that allows you to do auditing and recording compliance of your AWS resources. In particular, records configurations and changes over time. You set up config rules that are then check for compliance. For example:
- Is there unrestricted SSH access to my security groups?
- Do my buckets have any public access?
- Has my ALB configuration changed over time?
You can receive alerts via SNS notifications for any changes. Config is a per region service but can be aggregated across regions and accounts. You can store the configuration data into S3 and later analyzed by Athena.

`Config Rules`:
- Can use AWS managed config rules
- Can make custom config rules defined in Lambda
- Rules can be evaluated/triggered for each configuration change and/or at regular time intervals
- AWS Config Rules does not prevent actions from happening, only for compliance, it gives you a overview of your configuration 

There is no free tier, you pay per configuration item recorded per region and per config rule evaluation per region.

Although you cannot prevent actions from happening, you can automate remediation of non compliant resources using SSM Automation Documents. So you can trigger remediation actions when a resource becomes non compliant. For example when an IAM Access Key is expired, AWS Config can trigger a remediation action to deactivate the Key. These can be AWS managed automation documents or you can create custom automation documents that invoke a Lambda function. You can set Remediation retries in the event the resource is still non compliant after remediation.

We can use EventBridge to trigger notifications when AWS resources are non compliant. It also has the ability to send configuration changes and compliance state notifications to SNS, which can then notify an admin for example.
# EventBridge
---
Central console to manage events. Can schedule CRON jobs, Event rules to react to a service doing something, Trigger Lambda functions, send SQS/SNS messages etc... 

Events are sent to EventBridge, from which it then generates a JSON file with the information of the event, and then can be sent to many many destinations, practically anything you want.

AWS Services utilize the default Event bus. AWS has integrated with other outside partners. This means these AWS partners can also send their events into a specified Partner Event Bus in EventBridge. We can also send events from our own custom apps into EventBridge via a Custom Event Bus. Each Event bus can have Resource Based Policies to manage permissions. EventBridge also allows archiving events send to an event bus, these events can then be replayed (For example for debugging). 

EventBridge has a Schema Registry. EventBridge can analyze the events in your bus and infer the schema of the data. The Registry allows you to generate code for your application that will know in advance how data is structured in the event bus, these schemas can also be versioned

