
# AWS Developer Certification Notes

Table of Contets  
[1. IAM + Security](#iam--security)  
[2. EC2 + ENI](#ec2--eni)  
[3. ELB](#elastic-load-balancers)  
[4. ASG](#auto-scaling-group)  
3. ECS2 Storage:  EBS & EFS  
4. RDS + Aurora + ElastiCache  
5. Route 53  
6. VPC Fundamentals  
7. Amazon S3 Introduction  
8. loreipsum  
9. loreipsum  

### IAM + Security

#### AWS Regions
AWS has Regions all around the world. -> *eu-west-1*  
Each Region has many availability zones -> *eu-west-1*__(a,b,c,..)__  
Each availability zone has one or more discrete data centers with redundant power, networking, and connectivity  
#### IAM 
Users -> For physical persons  
Groups -> Permit group Users per Functions or Teams applying permissions to groups  
Roles -> For machines  
Policies -> JSON documents that defines what Users/Groups/Roles actions can/cannot do  

#### IAM Federation
Integrate corporate repository of users with AWS IAM.
Let login to AWS using their company credentials
Identity Federation uses the SAML standard (Active Directory)


#### Introduction to Security Groups
Security Groups -> the basic network security firewall in AWS
Regulate:
* Access to Ports
* Authorised IP Ranges
* Control Inboud network
* Control Outbound network

Security Groups are **stateful** - If u can send a request, It will always get the response traffic regardless the security group rules applied on the response network.

**TODO _Insert Security Group Picture+ EC2 Machine_**

Security Groups knowledge:
* Can be attached to multiple instances.  
* Are Locked down to a region /VPC combination
* All inbound traffic is **blocked** by default
* All outbound traffic is authorised by default
#### Public IP , Priavte IP & Elastic IP
* Public IP:
    *  Must be unique across the whole internet
    *  All the company devices are behind the Public IP.
* Private IP:
    * IPs which you can't acces internet.
    * Diferents networks can have the same IP because only works at local level.
* Elastic IP:
    * Can bind the instance to an Elastic IP to have always the same IP.
---
### EC2 + ENI
EC2 is Elastic Compute Cloud
Mainly consist in:
* Renting Virtual machines
* Storing data on virtual drives
* Distributing load across machines
* Scaling the services using auto-scaling group
#### SSH to EC2
|               | SSH           | Putty  | EC2 Instance Connect |
| :------------:|:-------------:|:-----:|:-------------:|
| Mac/Linux*      | OK |  |OK|
| Windows      |       |   OK |OK|
| Windows 10 | OK      |    OK |OK|  

*Mac/Linux must remember to restrict your .pem key using chmod 400

#### EC2 User Data
Specified camp from building Instances that let u write bootstrap commands.
* The script only run once at the instrance first start.
* EC2 user data is used to automate boot tasks.
* The EC2 User Data Script runs at root level
* Remember to #!/bin/bash

#### EC2 Instances Launch Types
There are On demand Instances, Reserved, Spot Instances & Dedicated:
* On demand instances -> Short Workload and predictable pricing
* Reserved: (_Minum 1 Year_)
    * Reserved Instances:
        * 75% discount comparet to on-demand
        * Reservation preiod 1 or 3 years
        * Reserve specific instance type
    * Convertible Reserved Instances: Long workloads with flex instance
        * Can change the EC2 instance Type
        * Up to 54% discount
    * Scheduled Reserved Instances
        * Launch withing the interval windows you reserve
        * When u requiere fraction of day/week/month
* Spot instances:
    * Can get 90% compared to On-demand
    * The instance will shutdown if your max price is less than the current spot price.
    * Cost-efficient instances in AWS
* Dedicated Host:
    * Physical dedicated EC2 server
    * Full control of EC2 Instance
    * 3 Years period reservation
    * Really expensive
    * Useful for software that have complicated licensing model (BYOL)
* Dedicated Instance:
    * Instance running on a physical machine that's dedicated to you
    * The instance can share the physical machine with other instance from the same Account.
    * Other Accounts will never use their instances on the same machine as yours. 
#### Elastic Network Interfaces (ENI)
Represents a Virtual Network Card like Virtual Networks Adapters
ENI attributes:
* Primary private IPv4
* One or more secondary IPv4
* One Elastic IPv4 x private IPv4
* Attached to one or more security groups
* MAC Addres

Useful for failover to EC2 instances.

---

### Elastic Load Balancers
Elastic Load balance wants to powerful the AWS high availability characteristic.
Load Balancer functions:
* Spread load across configured instances
* Expose a single DNS to your app
* Check the health of ur instances
* Frontend SSL for your websites
* Separate public traffic from private

AWS have 3 types of load balancers:
* Classic Load Balancer
    * Supports TCP _Layer 4_, HTTP & HTTPS _Layer 7_
    * Link to Security Group, Config HealthChecks and Add EC2 Instances
* Application Load Balancer
    * HTTP and HTTPS _Layer 7_
    * LB to multiple http applications across target groups
    * LB to multiple applications oriented to containers 
    * HTTP/2 and WebSocket
    * Routing tables to different target groups:
        * **Path URL**  example[.]com/_users_ example[.]com/_admins_
        * **Hostname in URL** _mail_.example.com _print_.example.com 
        * **Query String** example.com/id=?_12345_
    * Target Groups **_Check_**
        * IP must be private
    * Best Practices
        * Fixed Hostname XXX.region.elb.amazonaws.com
        * The application server don't see the IP from the client.
        * IP Client -> X-Forwarded-For
* Network Load Balancer
    * Forward TPC/UDP to your instrances
    * Millions of request per seconds
    * Less latency than ALB
    * One static IP per AZ
    * NLB are used for extreme perfonance

Load Balancer Attributes
* Load Balancer Stickiness
    * The same client is always redirected to the same instance behind a load balance.
    * CLB & ALB
* Cross-Zone Load Balancing
    * Each load balancer instance distributes evenly across all registered instances in all AZ
    * CLB 
        * Disabled by default
        * No charges for inter AZ data
    * ALB
        * Always on
        * No charges for inter AZ
    * NLB
        * Disabled by default
        * Pay charges for inter AZ data
* SSL/TLS
    * Allows traffic between clients and your load balancer to be encrypted in transit.
    * Public SSL certificates are issued by Certificate Authorities (CA)
    * Certifiactes using ACM or ur own certificates
    * HTTPS listener:
        * Specify a default certificate
        * Add optional list of certs to supp multiple domains
        * Clients can use **SNI** (Server Name Indicator) to specify the hostname they reach.
    * Server Name Indicator
        * Solvers the problem of loading multiple SSL certificates onto one web server.
        * Newer protocol which requires the client to indicate the hostname of the target server in the initial SSL handshake
        * The server will then find the correct certificate or return the default one.
        * Only works for ALB & NLB, CloudFront.
        * Does not work for CLB.
    * ELB Connection Draining
        * CLB: Connection Draining
        * Target Group: Deregistration Delay (For ALB & NLB)
        * Time to complete "in-flight requests" while the instance is de-registering or unhealthy
    
### Auto Scaling Group
Before start, we have to underestand the main concepts of auto scaling:
* Vertical Scalability:
    * Increase the size of the instance
    * Vertical scalability common for non distributed systems like databases.
    * RDS and ElastiCache typical scalables
* Horizontal Scalability:
    * Increase the number of instances/systems
    * Common on distributed systems.
    * Web application/microservices applicatons
* Goal of the ASG:
    * Scale out to match an increased load (_+ instances_)
    * Scale in to match a decreased load (_- instances_)
    * Define min and max num of machines running.
* ASG steps to config
    * Conf. panel
        * AMI + Instance Type
        * EC2 User Data
        * EBS Volumes
        * Security Groups
        * SSH Key Pair
    * Min/Max size Inital Capacity
    * Network+ Subnets Information
    * Load Balancer Information
    * Scaling Policies

* ASG ALARMS
    * Scale ASG with CloudWatch Alarms
    * Alarm monitors basic metrics like CPU Usage, RAM Usage...
    * We can create scale-in/out policies:
        * Target Tracking Scaling: always want CPU at 40%
        * Simple/step Scaling: CPU > 75% add 1 unit
        * Scheduled Action: Increase 1 unit to 18 at 24 pm on Fridays



---
### Advanced S3
##### S3 MFA-Delete
Force MFA b4 important operations on S3
* To use MFA-Delete -> enable versioning on the S3 Bucket
* Need MFA:
    * Permanent delete an object version
    * Suspend versioning on the bucket
* No need MFA:
    * enable versioning
    * listing deleted versions
* Onle the bucket owner **(root account)** can enable/disable MFA-Delete
* MFA-Delete can only be enabled using the CLI  

##### S3 Default Encruption
To use traffic encrypted on S3 u have the default encryption option
Bueckt policies are evaluated before "Default encryption"
##### S3 Access Logs
For audit purpose. Any request to a s3 will be logged into another s3 bucket.
##### S3 Replication (CRR & SRR)
Must enable versioning in source and destination
Cross Region Replication (CRR)
Same Region Replication (SRR)
* Bucket can be in different acc
* Copying is asynchronous
* Must give proper IAM permissions to S3
* Only new objects are replicated

Delete operations:
* if u delete withour a version ID, it adds a delete marker, not replicated
* if u delete with a version ID, it deletes in the source, not replicated

No chaining replication
* A->B
* B->C
* A content never arrives to C.

##### S3 pre-signed URLs
Can generate pre-signed URLs using SDK or CLI
* For downloads
* For uploads
* Default time of 3600 Seconds
* Users given a pre-signed URL inherit the permission of the person who generated the URL for GET/PUT

##### S3 Storage Classes
* Amazon S3 Standard - General Purpose
    * High durability 99,9p9% across multiple AZ
    * Availability 99.99%
    * sustain 2 concurrent facility failures
    * Use:Big data analytics, mobile & gaming applications, content distribution
* Amazon S3 Standard - Infrequent Access (IA)
    * Suitable for data that is les frequently accessed.
    * High durability 99,9p9% across multiple AZ
    * Availability 99.9%
    * Low cost compared to amazon S3 Standard
    * Sustain 2 concurrent facility failures
    * Use:Disaster recovery, backups
* Amazon S3 One Zone-Infrequent Access
    * Same as IA but single AZ
     * High durability 99,9p9% single AZ
     * 99.5% Availabilitty
     * low latency and high throughput performance
     * Supports SSL for data at transit and encryption at rest
     * Low Cost compared to IA
     * Use: Secondary backup copies, storing data you can recreate.    
* Amazon S3 Intelligent Tiering
    * Low latency and high performance of S3 standard
    * Small monthly monitoring and auto-tiering fee
    * Automatically moves objects between two access tiers based on access paterns
    * Designed for durability 99,9p9 across multiple Availability Zones
    * Designed for 99.9% availability 
* Amazon S3 Glacier
    * Low cost object storage for archiving/backup
    * Data is retained for the longer term _10 years_
    * Alternative to on-premise magnetic tape storage
    * Average annual durability 99.9p9
    * Cost per storage per month + retrieval cost
    * Each item in Glacier is called "Archive" _up to 40TB_
    * Archives are stored in "Vaults"
    * Retrieval options (_Minum storage duration **90 days**_):
        * Expedited (_1-5 minutes_)
        * Standard (_3-5 hours_)
        * Bulk (_5 to 12 hours_)

    
* Amazon S3 Glacier Deep Archive (Minum 180 days - cheaper)
    * Standard (_12 hours_)
    * Bulk (_48 hours_)

##### S3 Moving between storage classes
You can transition S3 storage classes to anothers:  
Infrequently accessed object -> Standard_IA  
For archieve objects -> Glacier or Deep_Archive  
Lifecycle Rules:
* Transition actions - When objects are transitioned to another storage class
    * Move objects to Standard IA class 60 days after creation
    * Move to gGlacier for archiving after 6 months
* Expiration actions - Configure objects to expire (_delete_) after some time
    * Acess log files can be set to delete after a 365 days
    * Can be used to delete old version of files (if versioning is enabled)
    * Can be used to delete incomplete multi-part upload
* Rules can be created for a certain prefix (ex  s3://mybucket/mp3/*)
* Rules can be created for certain objects tags (ex - Department: Finance)

##### S3 Performance
Amazon S3 automatically scales to gith request rates.
* Performance:
    * Latency 100-200ms
    * 3500 PUT/COPY/POST/DELETE req x sec x prefix in bucket
    * 5500 GET/HEAD req x sec x prefix in bucket
    * There are no limits to the number of prefixes in a bucket.
* KMS Limitation
    * Upload -> Call GenerateDataKey KMS API
    * Download -> Call Decrypt KMS API
    * Count towards the KMS quota per second (_5500, 10000, 30000 req/s_)
    * Can not increase request a quiota increase for KMS
* Multi-Part upload:
    * Recommended for files > 100MB
    * Must for > 5GB fles
    * Parallelize uploads
* S3 Transfer Acceleration (_Upload only_)
    * Increase transfer speed by transferring file to an aws edge location which will forward the data to the S3 bucket in the target regions
    * Compatible with multi-part upload

##### S3 Event notifications
* SNS -> Mail
* SQS -> Queue
* Lambda Function -> Execute Code

##### AWS Athena
* Serverless service to perform analytics againts S3 files
* Uses SQL language to query the files
* Has a JDBC/ODBC dirver
* Charged per query and amount of data scanned
* Supports CSV,JSON, ORC, Avro, and Parquet
* Use: Business intelligence/Analytics/reporting/VPC Flow Logs, ELB Logs, CloudTrail trails...
##### S3 Object Lock & Glacier Vault Lock
* S3 Object Lock
    * Adopt a WORM model
    * Block an object version deletion for a specified amount of time
* Glacier Vault Lock
    * Adopt a Worm model
    * Lock the policy for future edits
    * Helpful for compliance and data retention

### AWS CloudFront
Content Delivery Network (CDN)
* Improves read performances -> content is cached at the edge
* 216 Poin of Presence globally
* DDoS protection, integration with Shiedl, AWS Web Application Firewall
* Can expose external HTTPS and can talk to internal HTTPS Backends
##### S3 bucket
 * For distributing files and caching them at the edge
 * Enhanced security with CloudFron Origin Access Identity (OAI)
 * CloudFront can be used as an ingress (to upload files to S3)
##### Custom Origin (HTTP)
* Application Load Balancer
* EC2 instance
* S3 Website (must first enable the bucket as a static S3 website)
* Any HTTP Backend you want
##### CloudFront vs S3 Cross Region Replication
* CloudFront:
    * Global Edge network
    * Files are cached for a TTL
    * Great for static content that must be available everywhere
* S3 Cross Region Replication:
    * Must be setup for each region you want replication to happen
    * Files are updated in near real-time
##### CloudFront Caching
* Cache based on
    * Headers
    * Session Cookies
    * Query String Parameters
* The cache lives at each CloudFront Edge Location
* You want to maximize the cache hit rate
* Control the TTL
* You can invalidate part of the cache using the CreateInvalidation API
* Invalidating objects removes them from CloudFront edge caches (Interesting)
##### CloudFront Signed URL
* You want to distribute paid shared content to premium users over the world
* We can use CloudFront Signed URL/Cookie:
    * Includes URL expiration
    * Includes IP ranges to access the data from
    * Trusted signers (which aws accounts can create signed URLs)
* How long should the URL be valid for?
    * Shared content (movie,music): make it short
    * Private content (private to the user): u can make it last for yearys
* Signed URL **=>** acess to individual files (one signed URL per file)
* Signed Cookies **=>** access to multiple files (one signed cookie for many files)
##### CloudFront Signed URL vs S3 Pre-Signed URL
* CloudFront Signed URL:
    * Allow access to a path. no matter the origin.
    * Account wide key-pair, only the root can manage it
    * can filter by IP, path, date, expiration
    * Can leverage caching features
* S3 Pre-Signed URL:
    * Issue a request as the person who pre-signed the URL
    * Uses the IAM key of the signing IAM principal
    * Limited lifetime

### ECs, ECR & Fargate
###### Docker
All we know what is docker.
* Public -> Docker hub
* Private -> Amaozn ECR (Elastic Container Registry)
Resources are shard with the host
