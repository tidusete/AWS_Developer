
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
    * Access is controlled through IAM
    * AWS CLI v1 login command -> **$**(aws ecr get-login --no-include-email --region eu-west-1)
    * AWS CLI v2 login command aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.eu-west-1.amazonaws.com
    * Docker Push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
    * Docker Pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
Resources are shard with the host
##### ECS Cluster Overview
ECS -> Elastic Containter Service
* ECS CLuster are logical grouping of EC2 Instances
* EC2 instances run the ECS agent
* ECS agents register the instance to the ECS cluster
* EC2 instances run a special AMI made for ECS.
Like kubernetes

##### Practice
Important file /etc/ecs/ecs.config
##### ECS Task Definitions
Task definitions are metadata in JSON. Tells ECS how to run a Docker Container.
* Contains:
    * Image Name
    * Port Binding for Container and Host
    * Memory and CPU required
    * Environment variables
    * Networking information
##### ECS Services
ECS Services help define how many task should run and how they should be run.
* Ensure that the number of tasks desired is runing across our fleet of EC2 instances
* Can be linked to ELB / NLB / ALB if needed
##### Fargate
Like ECS but is serverless.
Amazon do it all
You only have to configure the container. (Like Cloud but with containers)
##### ECS IAM Roles Deep Dive
* EC2 Instance Profile:
    * Used by the ECS agent
    * Makes API calls to ECS service
    * Send container Logs to CloudWatch Logs
    * Pull Docker image from ECR
* ECS Task Role:
    * Allow each task to have a specific role
    * Use different roles for the different ECS Services you run
    * Task Role is defined in the task definition
    
##### ECS Task Placement
* Helpful to determine where the task of type EC2 will be placed, the CPU, memory and available port.
* When a service scales in, ECS need to determine which task to terminate.
* Task placement strategies are best effort
* Process:
  + Identify the instances that satisfy the CPU, memor and port req.
  + Identify the instances that satisfy the task placement constraints
  + Identify the isntances that satisfy the task placement strategies
  + Select the instances for task placement
##### ECS Task Placement Strategies
U can mix this strategies on the same placementStrategy
* Binpack
  * Place tasks based on the least available amount of CPU or memory
  * Minimizes the number of instances in use (cost saving)
* Random
  * Place the task randomly
* Spread
  * Task will be spread based on the specified value
  * Examples: InstanceID, attribute:ecs:availability-zone ...
##### ECS Task Placement Constraints
* distinctInstance
  * Place each task on a different container instance
* memberOf
  * Place task on instances that satisfy an expresion
  * Use the Cluster Query Language
##### ECS Service Auto Scaling
CPU and RAM is tracked in CloudWatch at the ECS service level
* **Target Tracking** 
  * target a specific average CloudWatch metric.
* **Step Scaling**
  *  scale based on CloudWatch alarms
* **Scheduled Scaling** 
  * based on predictable changes

ECS Service Scaling (task level) **!=** EC2 Auto Scaling (Instance level)  
Fargate Auto Scaling is much easier to setup (Serverless)
##### ECS Cluster Capacity Provider
Used in association with a cluster to determine the incrastructure that a task runs on
* For ECS and Fargate users, the FARGATE and FARGATE_SPOT capcity providers are added automatically
* For Amazon ECS on EC2, you need to associate the capacity provider with a auto-scaling group  

When you run a task or a service, you define a capacity provider strategy, to prioritize in which provider to run.
This allows the capacity provider to automatically provision infrastructure for you

##### ECS Summary
ECS is used to run Docker containers and has 3 flavors:
* ECS "Classic"
  *  Provision EC2 instances to run containers onto
  + EC2 instances must be created
  + We must configure the file /etc/ecs/ecs.config with the cluster name
  + EC2 instance must run an ECS agent to connect to cluster
  + EC2 instances can run multiple containers on the same type
    + You must **not** specify a host port
    + You should use an Application Load Balancer with the dynamic port mapping
    + The EC2 instance security group must allow traffic from the ALB on all port
  + ECS tasks can have IAM Roles to execute action against AWS
  + Security groups operate at the instance level, not task level
*  Fargate
   * ECS Serverless, no more EC2 to provision
   * AWS provisions containers for us and assigns them ENI
   * Fargate containers are provisioned by the container spec (CPU/RAM)
   * Fargate tasks can have IAM Roles to execute actions agains AWS
* EKS
  *  Managed Kubernetes by AWS  
  
ECR is used to store Docker Images
 * ECR is tightly integrated with IAM
 * AWS CLI V1 login command
   * $(aws ecr get-login --no-include-email --region eu-west-1)
   * "aws ecr get-login" generates a "docker login" command
  * AWS CLI v2 Login command (newer, may also be asked at the exam -pipe)
    * aws ecr get-login.password --region eu-west-1 | docker login --username AWS password-stdin 1234567890.dkr.ecr.eu-west-1.amazonaws.com
  * Docker Push & Pull:
    * docker push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
    * docker pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
* ECS does integrate with cloudWatch Logs:
  * You need to setup loggin at the task definition levle
  * Each container will have a different log stream
  * The EC2 Instance Profile need to have the correct IAM permissions
* Use IAM Task Roles for your tasks
* Task Placement Strategies: binpack, random, spread
* Service Auto Scaling with target tracking, step scaling or scheduled
* Cluster Auto Scaling through Capacity Providers
### AWS Elastic Beanstalk
##### Overview
* Elastic Beanstalk is a developer centric view of deploying an application on AWS
* Uses all the component's we've seen before: EC2,ASG,ELB,RDS, etc...
* Still have full control over the configuration
* Beanstalk is free but you pay for the underlying instances
##### Deep
* Managed service
  * Instance configuration / OS is handled by Beanstalk
  * Deployment strategy is configurable but performed by Elastic Beanstalk
* The application code is the responsibility of the developer
* Three architecture models:
  * **Single Instance deployment** -> good for dev
  * **LB + ASG** -> great for production or pre-production web applications
  * **ASG only** -> non-web apps (Workers or queue)
##### Components
* Elastic Beanstalk has three components
  * Application
  * Application version: each deployment gets assigned a version
  * Environment name (dev,test,prod...)
* You deploy application versions to environments and can promote application versions to the next environment
* Rollback feature to previous application version
* Full control over lifecycle of environments
#####  Beanstalk Deployment Options for Updates
* **All at once (deploy all in one go)** - fastest, but instances aren't available to serve traffic for a bit (downtime)
  * shutdown all instance and rise the news
  * Fastest deployment
  * Application has downtime
  * Great for quick iterations in development environment
  * No additional cost
* **Rolling**: update a few isntances at a time (bucket), and then move onto the next bucket one the fist bucket is healthy (_update slowly_)
  * Application is running below capacity
  * Can set the bucket size.
  * _Shut down half of ur system cause u are releasing the new verison and want to keep working your system with the rest of the old instances_
* **Rolling with addition batches** like rolling, but spins up new instances to move the batch (update minimal cost and maitaning full capacity)
  * Application is running at capacity
  * Can set the bucket size
  * Application is running both versions simultaneously
  * Small additional cost
  * Addiotional batch is removed at the end of the deployment
  * Longer deployment
  * Great for prod

* **Immutable**: spins up new isntances in a new ASG, deploys versions to these instances and then swaps all the isntances when everything is healthy (*Rollback inmediatly*)
  * Zero downtime
  * New Code is deployed to new isntances on a temporary ASG
  * High cost, double capacity
  * longest deployment
  * Quick rollback in case of failures
  * Great for prod
* **Blue / Green**
  * Not a "direct feature" of Elastic Beanstalk
  * Zero downtime and release facility
  * Create a new "stage" environment and deploy v2 there
  * The new environment (green) can be validated independently and roll back if issues
  * Route 53 can be setup using weighted policies to redirect a little bit of traffic to the stage environment
  * Using Beanstalk, "swap URLs" when done with the environment test
 _**Faltaria ficar les fotos**_ 
---
##### Beanstalk Lifecycle Policy
* Elastic Beanstalk can store at most 1000 aplications versions.
* Have to remove old versions to deploy more.
* To phase out old applications versions use lifecycle policy
  * Based on time
  * Based on space
* Versions that are currently used won't be deleted
* Option not to delete the source bundle in S3 to prevent data loss

##### Elastic Beanstalk Extensions
* A zip file containing our code must be deployed to Elastic Beanstalk
* All the parameters set in the UI can be configured with code using files
* Requirements
  * In the .ebextensions/ directory in the root of source code
  * YAML / JSON format
  * .config  extensions (example: loggin.config)
  * Able to modify some default settings using: option_settings
  * Ability to add resources such as RDS, ElastiCache, Dynamo DB, etc...
* Resources managed by .ebextensions get deleted if the environment goes away

##### Elastic Beanstalk Under the Hood
Under the hood, Elastic Beanstalk relies on CloudFormation
CloudFormation is used to provision other AWS services
You can define CloudFormation resources in your _.ebextensions_ to provision ElastiCache, S3 bucket...
##### Elastic Beanstalk Clonning
Clone an environment with the exact same configuration
Useful for deploying a "test" version of your applicaiton
* All resources and configuration are preserved:
  * Load Balancer type and configuration
  * RDS database type (but the data is not preserved)
  * Environment variables
##### Elastic Beanstalk Migration: Load Balancer
  After creating an Elastic Beanstalk environment, you cannot change the Elastic Load Balancer Type
  You have to migrate:
    * Create a new environment with the same configuration except the Load Balancer
    * Deploy your applicaiton onto the new environment
    * Perform a CNAME swap or Route 53 Update

##### Elastic Beanstalk - Single Docker
Run your application as a single docker container
Ether provide:
  * Dockerfile: Elastic Beanstalk will build and run the Docker container
  * Dockerrun.awsjson (v1): Describe where *already built* Docker image is:
    * image
    * ports
    * volumes
    * Loggin
  * Beanstalk in Single Docker Container **does not use ECS**
##### Elastic Beanstalk - Multi Docker Container
* Multi Docker helps run multiple containers per EC2 instance in EB
* Components:
  * ECS Cluster
  * EC2 instances, configured to use the ECS Cluster
  * Load Balancer (in high availability mode)
  * Task definitions and executions
* Requires a config Dockerrun.aws.json (v2) at the root of source code
* Dockerrun.aws.json is used to generate the ECS task definition
* Your Docker image must be pre-build and stored in ECR x exemple.
##### Elastic Beanstalk and HTTPS
* Beanstalk with HTTPS
  * _Load the SSL Certificate onto the LoadBalancer_
    * Can be done from the Console (EB console, load balancer configuration)
    * Can be done from the code: .ebextensions/securelistener-alb.config
    * SSL Certificate can be provisioned using ACM (AWS Certificate Manager) or CLI
    * Must Configure a security group rule to allow incoming port 443 (HTTPS port)
* Beanstalk redirect HTTP to HTTPS
  * Configure your instances to redirect HTTP to HTTPS:
  * Or configure the application Load Balancer (ALB only) With rule
  * Make sure health checks are not redirected (so they keep giving 200 ok)
  ##### Elastic Beanstalk - Custom Platform (Advanced)
  * Custom Platforms are very advanced, they allow to define from scratch:
    * The Operating System (OS)
    * Additional Software
    * Scripts that Beanstalk runs on these platforms
  * Use case: app language is incompatible with Beanstalk & doesn't sue DOcker
  * To create your own platform:
    * Define an AMI using Platform.yaml file
    * Build that platform using the Pacjer Software (open source tool to create AMIs)
  * Custom Platform vs Custom Image (AMI):
    * Custom Image is to tweak an **existing** Beanstalk platform (Python, Node.js, Java)
    * Custom Platform is to create **an entirely new** Beanstalk Platform
### AWS CICD:
##### Technology Stack for CICD
|  Code | Build   | Test   | Deploy   |  Provision |
|---|---|---|---|---|
| AWS CodeCommit  | AWS CodeBuild  |AWS CodeBuild   | AWS Elastic Beanstalk  | AWS Elastic Beanstalk |
|   Github Or 3rd party code repository| Jenkins CI Or 3rd party CI Servers   |   Jenkins CI Or 3rd party CI Servers | AWS CodeDeploy  | User Managed EC2 Instances Fleet (CloudFOrmation)  |

All integrated with Orchestrate: **AWS CodePipeline**
#### CodeCommit
##### CodeCommit Overview
* **Version Control** -> ability to underestand the various changes that happened to the code over time.
* Enabled using a version control system such as Git
* Git repository can live on one's machine or central online repository
* Benefits are:
  * Collaborate with other dev
  * Backed-up code
  * Viewable and auditable

##### CodeCommit AWS
* Private Git repositories
* No size limit on repositories (scale seamlessly)
* Fully managed, highly available
* Code Only in AWS Cloud Account
* Secure (encrypted, access control, etc)
* Integrated with jenkins / CodeBuild / Other CI Tools

##### CodeCommit Security
* Interactions are done using Git
* Authentication in Git:
  * SSH Keys: AWS Users can configure SSH keys in their IAM Console
  * HTTPS: Done through the AWS CLI Authentication helper or Generating HTTPS credentials
  * MFA can be enabled for extra safety
* Authorization in Git:
  * IAM Policies manage user / roles rights to repositories
* Encryption:
  * Repositories are automatically encrypted at rest using KMS
  * Encrypted in transit (can only use HTTPS or SSH - both secure)
* Cross Account access:
  * Do not share your SSH keys
  * Do not share your AWS credentials
  * Use IAM Role in your AWS Account and use AWS STS (with AssumeRole API)

##### CodeCommit Notifications
You can trigger notifications in CodeCommit using **AWS SNS** (Simple Notification Servie) or **AWS Lambda** or **AWS CLoudWatch Event Rules**
* Use Case SNS /AWS Lambda:
  * Deletion of branches
  * Trigger for pushes in master branch
  * Notify external Build System
  * Trigger AWS Lambda function to perform codebase analysis
* Use cases for CLoudWatch Event Rules:
  * Trigger for pull request updates
  * Commit comment events
  * CloudWatch Event Rules goes into an SNS topic

#### CodePipeline
##### CodePipeline Overview
* Continuous delivery
* Visual workflow
* Source: GitHub / CodeCommit / Amazon S3
* Build: CodeBuild / Jenkins /etc ...
* Load Testing: 3 party tools
* Deploy: AWS CodeDeploy / Beanstalk / CloudFormation / ECS ...
* Made of stages:
  * Each stage can have squential actions and / or parallel actions
  * Stage examples: Build / Test / Deploy / Load Test / etc ...
  * Manual approval can be defined at any stage

##### CodePipeline Artifacts

Each pipeline stage can create "artifacts", artifacts are passed stored in Amazon S3 and passed on to the next stage.

_Each stage will output artifacts to the next stage_

##### CodePipeline Troubleshooting

CodePipeline state changes happen in **AWS CloudWatch Events**, which can in return create SNS notifications.
  * Create events for failed pipelines
  * Create events for cancelled stages  

If CodePipeline fails a stage, the pipeline will stop and u will get the info in the console

AWS CloudTrail can be used to audit AWS API calls
Pipeline have to be attached an IAM Service Role to do some actions

#### CodeBuild
##### CodeBuild Overview 
* Fully managed build service
* Alternative to other build tools like Jenkins
* Continuous scaling (no servers to manage or provision - no build queue)
* Pay for usage: the time it takes to complete the builds
* Leverages Docker under the hood for reproducible builds
* Possibility to extend capabilities leveraging our own base Docker images
* Secure: Integration with KMS for encryption of build artifacts, IAM for build permissions, and VPC for network security, CloudTrail for API calls loggin
##### CodeBuild Properties
* Source Code from GitHub / CodeCommit / CodePipeline / S3...
* Build instructions can be defined in code (buildspec.yml file)
* Output logs to Amazon S3 & AWS CloudWatch Logs
* Metrics to monitor CodeBUilds statistics
* Use CloudWatch Alarms to detect failed builds and trigger notifications
* CloudWatch Events / AWS Lambda as a Glue
* SNS notifications
* Ability to reproduce CodeBuild locally to troubleshoot in case of errors
* Builds can be defined within CodePipeline or CodeBuild itself
##### CodeBuild BuildSpec
* **buildspec.yml** file must be at the **root** of your code
* Define environment variables:
  * Plaintext variables
  * Secure secrets: use SSM Parameter store
* Phases (specify commands to run):
  * Install: install dependencies you may need for your build
  * Pre build: final commands to execute before build
  * **Build: Actual build commands**
  * Post build: finishing touches (zip output for example)
* Artifacts: What to upload to S3 (encrypted with KMS)
* Cache: Files to cache (usually dependencies) to S3 for future build speedup
##### CodeBuild Local Build
* In case of need of deep troubleshooting beyond logs...
* You can run CodeBuild locally on tour desktop (after installing Docker)
* For this, leverage the CodeBuild Agent
##### CodeBuild in VPC
* By default, your CodeBuild containers are launched outside your VPC
* Therefore, by default it cannot access resources in a VPC
* You can specify a VPC configuration:
  * VPC ID
  * Subnet IDs
  * Security Group IDs
* Then your build can access resources in your VPC (RDS, ElastiCache, EC2, ALB)
* Use cases: integration test, data query, internal load balancers
#### CodeDeploy
##### CodeDeploy overview
* We want to deploy our applicaiton automatically to many EC2 instances
* These isntances are not managed by Elastic Beanstalk
* There are several ways to handle deployments using open source tools (Ansible, Terraform, Chef, Puppet, etc...)
* We can use the managed Service AWS CodeDeploy
##### CodeDeploy Steps to make it Work
* Each EC2 Machine (or On Premise machine) must be running the **CodeDeploy Agent**
* CodeDeploy sends appspec.yml file
* Application is pulled from GitHub or S3
* EC2 will run the deployment instructions
* CodeDeploy Agent will report of success / failure of deployment on the instance
##### CodeDeploy Other
* EC2 instances are grouped by deployment group (dev / test / prod)
* Lots of flexibility to define any kind of deployments
* CodeDeploy can be chained into CodePipeline and use artifacts from there
* CodeDeploy can re-use existing setup tools, works with any application, auto scaling integraiton
* Note: Blue / Green only works with EC2 instances (not on premise)
* Support for AWS Lambda deployments (we'll see this later)
* CodeDeploy does not provision resources
##### CodeDeploy Primary Components
* **Aplication** -> unique name
* **Compute platform** ->  EC2/On-Premise or Lambda
* **Deployment configuration** ->  Deployment rules for success / failures
  * EC2/On-Premise: you can specify the minimum number of healthy instances for the deployment.
  * AWS Lambda specify how traffic is routed to your updated Lambda function version.
* **Deployment group** -> group of tagged instances (allows to deploy gradually)
* **Deployment Type** -> In-place deployment of Blue/green deployment
* **IAM instance profile** -> need to give EC2 the permissions to pull from S3 / GitHub
* **Application Revision** -> application code + appsec.yml file
* **Service role** -> Role for CodeDeploy to perform what it needs
* **Target revision** -> Target deployment application version
##### CodeDeploy AppSpec
* File section -> How to source and copy from S3 / GitHub to filesystem
* Hooks -> set of instructions to do to deploy the new verion (hooks can have timeouts)
* The order of hooks:
  * **ApplicationStop**
  * **DownloadBundle**
  * **BeforeInstall**
  * **AfterInstall**
  * **ApplicationStart**
  * **ValidateService** *really important*

##### CodeDeploy Deployment Config
* Configs:
  * On a time -> one instance at a time, if one instance fails the deployment stops
  * Half at a time 50%
  * All at once quick but no healthy host, downtime. *Good for dev*
  * Custom min healhty host 75%
* Failures:
  * Instances stay in "failed state"
  * New deployments will first be deployed to "failed state" instance
  * To rollback: redeploy old deployment or enable automated rollback for failures
* Deployment Targets:
  * Set of EC2 isntances with tags
  * Directly to an ASG
  * Mix of ASG / Tags so you can build deployment segments
  * Customization in scripts with DEPLOYMENT_GROUP_NAME environment variables
##### CodeDeploy to EC2
* Define how to deploy the application using appspec.yml + deployment strategy
* Will do in-place update to your fleet of EC2 isntances
* Can use hooks to verify the deployment after each deployment phase
##### CodeDeploy to ASG
* In place updates:
  * Updates current existing EC2 instances
  * Instances newly created by an ASG will aslo get automated deployments
* Blue/green deployment:
  * A new auto-scaling group is created (settings are copied)
  * Choose how long to keep the old instances
  * Must use an ELB
##### CodeDeploy Rollbacks
You can specify the following rollback options:
  * Roll back when a deployment fails
  * Roll back when alarm thresholds are met
  * Disable rollbacks - Do not perform rollbacks for this deployment
_If a roll back happens, CodeDeploy redeploys the last known good revision as a new deployment_
##### CodeStar
CodeStar is an integrated solution that regroups: GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch
* Helps quickly create "CICD-ready" projects for EC2, Lambda, Beanstalk
* Supported languages: C#, Go, HTML 5, Java, Node.js, PHP, Pyhton, Ruby
* Issue tracking integration: JIRA / GitHub Issues
* Ability to integrate with Cloud9 to obtain a web IDE
* One dashboard to view all your components
* Free service, pay only for the underlying usage of other services
* Limited Customization
### AWS CloudFormation
##### What is CloudFormation
CloudFormation is a declarative way of outlining your AWS Infrastrcture.
Example of CloufRomation template:
  * I want a security group
  * I want two EC2 machines using this security group
  * I want two Elastic IPs for there EC2 machines
  * I want an S3 bucket
  * I want a load balancer in fron to these machones
CloudFormation creates those for u, in the **right order** with the **exact configuration**
##### Benefits of AWS Cloud Formation
* Infrastructure as code
  * No resources are manually created, which is excellent for contorl
  * The code can be version controlled for example using git
  * Changes to the intrastructure are reviewed through code
* Code
  * Each resources withing the stack is stagged with an identifier so you can easily see how much a stack cost you
  * You can estimate the costs of your reources using the CloudFormation template
  * Saving strategy: In Dev, you could automation deletion of templates at 5PM and recreated at 8 AM, safely
* Productivity
  * Ability to destroy and re-create an infrastructure on the cloud on the fly
  * Automated generation of Diagram for your templates
  * Declarative programming (no need to figure out ordering and orchestration)
* Separation of concern: create many stacks for many apps, and many layers.
  * VPC stacks
  * Network stacks
  * App stacks
##### How CloudFormation Works
Templates have to be uploaded in S3 and then referenced in CloudFormation
* Update template -> we can't edit previous ones. We have to reupload a new verions of the template to aws
* Stacks are identified by a name
* Deleting a stack deletes every single artifact that was created by CloudFormation
##### Deploying CloudFormation templates
* Manual way:
  * editing templates in the CLoudFormation Designer
  * Using the console to input paraments, etc
* Automated way:
  * Editing templates in a YAML file
  * Using the AWS CLI to deploy the templates
  * Recommended way when you fully want to automate your flow
##### CloudFormation Resources
Resources are the core of your CloudFormation Template
* They represent the different AWS components that will be created and configured
* Resources are declared and can reference each other
* AWS figures out creation, updates and deletes of resources for us
* There are over 224 types of resources
* Reosurces types identifiers are of the form:
  * **AWS::aws-product-name::data-type-name**
##### CloudFormation Parameters
* Parameters are a way to provide inputs to your AWS CloudFormation template
* They are important to know about if:
  * You want to reuse your templates across the company
  * Some inputs can not be determined ahead of time
  * Parameters are extremly powerful, controlled, and can prevent erros from happening in your templaters thanks to types.
* The **Fn::Ref** function can be leveraged to reference parameters
* Parameters can be used anywhere in a template
* The shorthand for this is YAML is !Ref
##### CloudFormation Mappings
* Mappings are fixed variables within your CloudFormation Template.
* They're very handy to differentiate between different environments, regions, AMI Types, etc...
* All the values are hardcoded within the template
* Mappings are great when you know in advance all the values that can be taken and that they can be deduced from variables such as:
  * Regions
  * Availability Zone
  * AWS Account
  * Environment
  ...
* They allow safer control over the template
* Use parameters when the values are really user specific.
* Accessing Mapping Values:
  * We use **Fn::FindInMap** to return a named value from a specific key
  * **!FindInMap [MapName, TopLevelKey, SecondLevelKey]**
##### CloudFormation Rollbacks
* Stack Creation Fails:
  * Default: everything rolls back (gets delelted). We can look at the log
  * Option to disable rollback and troubleshoot what happened-
* Stack Update Fails:
  * The stack automatically rolls bakc to the previous known working state
  * Ability to see in the log what happened and error messages
##### CloudFormation ChangeSets
* when you update a sstack, you need to know what hanges before it happens for greater confidence
* ChangeSets won't say if the update will be successful
##### CloudFormation Nested Stacks
* Nested stacks are stacks as part of other stacks
* They allow you to isolate repeated patterns  / common components in separate stacks and call them from other stacks
* Example:
  * Load Balancer configuration that is re-used
  * Security Group that is re-used
* Nested stacks are considered best practice
* To update a nested stack, always update the parent (root stack)
##### CloudFormation Cross vs Nested Stack
* Cross Stacks
  * Helpful when stacks have different lifecycles
  * Use Outputs Export and Fn::ImportValue
  * When you need to pass export values to many stacks (VPC id, etc...)
* Nested Stacks
  * Helpful when components must be re-used
  * Ex: re-use how to properly configure an Application Load Balancer
  * The nested stack only is important to the higher level stack (it's not shared)
##### CloudFormation StackSets 
* Create, update, or delete stacks acorss **multiple accounts and regions** with a single operation 
* Administrator account to create StackSets
* Trusted accounts to create, update, delete stack instances from StackSets
* When you update a stack set, all associated stack instances are update throughout all accounts and regions.

### AWS Monitoring & Audit: CloudWatch, X-Ray and CloudTrail
#### Monitoring in AWS
* AWS CloudWatch:
  * Metrics: Collect and track key metrics
  * Logs: Collect, monitor, analyze and store log files
  * Events: Send notifications when certain events happen in your AWS
  * Alarms: React in real-time to metrics / events
* AWS X-Ray:
  * Troubleshooting application performance and errors
  * Distributed tracing of microservices
* AWS CLoudTrail:
  * Internal monitoring of API calls being made
  * Audit changes to AWS reousrces by your users

#### AWS CloudWatch Metrics
* CloudWatch provides metrics for every services in AWS
* **Metric** is a variable to monitor (CPUutilitzaiton, Networkin..)
* Metrics belong to **namespaces**
* **Dimension** is an attribute of a metric (instance id, environment, etc...)
* Up to 10 dimensions per metric
* Metrics have **timestamps**
* Can create CloudWatch dashboards of metrics
##### AWS CloudWatch EC2 Detailed monitoring
* EC2 instance metrics have metrics "*every 5 minutes*"
* With detailed monitoring (for a cost), you get data "*every 1 minute*"
* Use detailed monitoring if you want to more prompt scale your ASG
* The AWS Free Tier allows us to have 10 detailed monitoring metrics
* Note: EC2 Memory usage is by default not pushed (must be pushed from inside the instance as a custom metric)
##### AWS CloudWatch Custom Metrics
* Possiblity to define and send your own custom metrics to CLoudWatch
* Ability to use dimensions (attributes) to segment metrics
  * Instance.id
  * Environment.name
* Metric resolution:
  * Standard: 1 minute
  * High Resolution: up to 1 second (**StorageResolution** API parameter) - Higher cost
* Use API call PutMetricData
* Use exponential back off in case of throttle errors
##### AWS CloudWatch Alarms
* Alarms are used to trigger notifications for any metric
* Alarms can go to AutoScaling, EC2 Actions, SNS notifications
* Various options (sampling, %, max, min, etc...)
* Alarm States:
  * OK
  * INSUFFICIENT_DATA
  * ALARM
* Period:
  * Length of time in seconds to evaluate the mtric
  * High resolution custom metrics: can only choose 10 sec or 30 sec

#### AWS CloudWatch Logs
* Applications can send logs to CloudWatch using the SDK
* CloudWatch can collect log from:
  * Elastic Beanstalk: collection of logs from application
  * ECS: collection from containers
  * AWS LAmbda: collection from function logs
  * VPC Flow Logs: VPC specific logs
  * API Gateway
  * CloudTrail based on filter
  * CloudWatch log agents: for example on EC2 machines
  * Route53: Log DNS queries
* CloudWatch Logs can go to:
  * Batch exporter to S3 for achival
  * Steam to ElasticSearch cluster for futher analytics
* CloudWatch Logs can use filter expressions
* Logs storage arhcitecture:
  * Log groups: arbitrary name, usually representing an application
  * Log stream: instances withing application / log files / containers
* Can define log expiration policies (never expire, 30 days, etc...)
* Expire after 7 days by default 
* Using the AWS CLI we can tail CloudWatch logs
* To send logs to CloudWatch, make sure IAM permission are correct
* Security: encryption of logs using KMS at the group level
##### CloudWatch Logs for EC2
* No logs from your EC2 mahine will go to CloudWatch by default
* You need to run a CloudWatch agent on EC2 to push the log files you want
* Make sure IAM permissions are correct
* The CloudWatch log agent can be setup on-premises too
##### CloudWatch Logs Agent & Unified Agent
* For virtual servers (EC2 isntances, on-premise servers...)
* **CloudWatch Logs Agent**
  * Old version of the agent
  * Can only send to CloudWatch Logs
* **CloudWatch Unified Agent**
  * Collect additional system-level metrics such as RAM, processes, etc...
  * Collect logs to send to CloudWatch Logs
  * Centralized configuration using SSM Parameters Store
##### CloudWatch Logs Metric Filter
* CloudWatch Logs can use filter expressions
  * for example, find a specific IP inside of a log
  * Or count occurences of "ERROR" in your logs
  * Metric filters can be used to trigger alarms
* Filters do no retroactively filter data. Filters only publish the metric data points for events that happen after the filter was created.
##### AWS CloudWatch Events
* Schedule: Cron jobs
* Event Pattern: Event rules to react to a service doing something
  * Ex: CodePipeline state changes!
* Triggers to Lambda fucntions, SQS/SNS/Kinesis Messages
* CloudWatch Event creates a small JSON document to give information about the change
##### Amazon EventBridge
* EventBridge is the next evolution of CloudWatch Events
* Default event bus: generated by AWS services (CloudWatch Events)
* Partner event bus: receive events from SaaS service or applications (Zendesk, DataDog, Segment, Auth0...)
* Custom Event buses: for your own applications
* Event buses can be accessed by other AWS accounts
* Rules: how to process the events (similar to CloudWatch Events)
##### Amazon EventBridge Schema Registry
* EventBridge can analyze the events in your bus and infer the schema
* The Schema Registry allows you to generate code for your application that will know in advance how data is structured in the event bus
* Schema can be versioned
##### Amazon EventBridge vs CloudWatch Events
* Amazon EventBridge builds upon and extends CloudWatch Events.
* It uses the same service API and endpoint, and the same underlying service infrastructure.
* EventBridge allows extensions to add event buses for your custom applications and your third-party SaaS apps.
* Event bridge has the Schema Registry capability
* Event Bridge has a different name to mark the new capabilities
* Over time, the CloudWatch Events name will be replaced with EventBridge
#### AWS X-Ray
* Debugging in Production, the good old way:
  * Test locally
  * Add log statements everywhere
  * Re-deploy in production
* Log formats differ across applications using CloudWatch and analytics is hard.
* Debugging: monolith "easy", distributed services "hard"
* No common views of your entire architecture!
##### AWS X-Ray advantages
* Troubleshooting performance (bottlenecks)
* Understand dependencies in a microservice architecture
* Pinpoint service issues
* Review request behavior
* Find errors and exceptions
* Are we meeting time SLA?
* Where I am throttled?
* Indentify users that are impacted
##### AWS X-Ray Leverages Tracing
* Tracing is an end to end way to following a "request"
* Each component dealing with the request adds its own "trace"
* Tracing is made of segments (+ sub segments)
* Annotations can be added to traces to provide extra-information
* Ability to trace:
  * Every request
  * Sample request (as a % for example or a rate per minute)
* X-Ray Security:
  * IAM for authorization
  * KMS for encryption at rest
##### AWS X-Ray Hot to enable it?
1. Your code (Java, Python, Go, Node.js) must import the AWS X-Ray SDK
    * Very little code modification needed
    * The applicaiton SDK will then capture:
      * Call to AWS services
      * HTTP / HTTPS requests
      * Database Calls (MySQL, PostgreSQL, DynamoDB)
      * Queue calls (SQS)  

2. Install the X-Ray daemon or enable X-Ray AWS Integration
    * X-Ray daemon works as a low level UDP packet interceptor (Linux / Windows / Mac)
    * AWS Lambda / other AWS services already run the X-Ray daemon for you
    * Each application must have the IAM rights to write data to X-Ray
##### AWS X-Ray Troubleshooting
* If X-Ray is not working on EC2
  * Ensure the EC2 IAM Role has the proper permissions
  * Ensure the EC2 instance is running the X-Ray Daemon
* To enable on AWS Lambda:
  * Ensure it has an IAM execution role with proper policy (AWSX-RayWriteOnlyAccess)
  * Ensure that X-Ray is imported in the code
##### AWS X-Ray Instrumentation in your code
* Instrumentation means the measure of product's performance, diagnose errors, and to write trace information.
* To instrument your application code, you use the X-Ray SDK
* Many SDK require only configuration changes
* You can modify your application code to customize and annotation the data that the SDK sends to X-Ray, using interceptors, filter, handlers, middleware ...
##### X-Ray Concepts
* Segments - each application / service will send them
* Subsegments - if you need more details in your segment
* Trace - segments collected together to form an end-to-end trace
* Sampling - decrease the amount of requests sent to X-Ray, reduce cost
* Annotations - Key Value pairs used to index traces and use with filters
* Metadata - Key Value pairs, not indexed, not used for searching  

* The X-Ray daemon / agent has a config to send traces cross account:
  * make sure the IAM permissions are correct - the agent will asume the role
  * This allows to have a central account for all your application tracing.
##### X-Ray Sampling Rules
* With sampling rules, you control the amount of data that you record
* You can modify sampling rules without changing your code

* By default, the X-Ray SDK records the first request **each second**, and **five percent** of any additional request.
* **One request per second is the reservoir**, which ensures that at least one trace is recorded each second as long the service is serving requests
* **Five percent** is the rate at which additional requests beyond the reservoir size are sampled.


* You can create your own custom rules with the **reservoir** and **rate**

##### X-Ray Write/Read APIs (used by the X-Ray daemon)
Policy name is **AWSXrayWriteOnlyAccess**
* **PutTraceSegments** Uploads segment documents to AWS X-Ray
* **PutTelemetryRecords** Used by the AWS X-Ray daemon to upload telemtry
  * SegmentsReceivedCount,SegmentsRejectedCounts,BackendConnectionErrors...
* **GetSamplingRules**: Retrieve all sampling rules (toknow what/when to send)
* **GetSamplingTargets** & **GetSamplingStatisticSummaries**: advanced
* The X-Ray daemon needs to have an IAM policy authorizing the correct API calls to funcion correctly  

Policy name is **AWSXrayReadOnlyAccess**
* **GetServiceGraph** main graph
* **BatchGetTraces** Retrieves a list of traces specified by ID. Each trace is a collection of segments documents that originates from a single request.
* **GetTraceSummaries** Retrieves IDs and annotations for traces available for a specified time frame using an optional filter. to get the full traces, pass the trace IDs to BatchGetTraces.
* **GetTracesGraph** Retrieves a service graph for one or more specific trace IDs

##### X-Ray with Elastic Beanstalk
* AWS Elastic Beanstalk platforms inlcude the X-Ray daemon
* You can run the daemon by setting an option in the Elastic Beanstalk console or with a configuration file (in .ebextensions/xray-daemon.config)
* Make sure to give your instance profile the correct IAM permissions so that the X-Ray daemon can function correctly
* Then make sure your application code is instrumented with the X-Ray SDK
* Note: The X-Ray daemon is not provided for Multicontainer Docker 
##### ECS + X-Ray integration options
* X-Ray **Container as a Daemon**
  * You have the EC2's and runs there a **X-Ray DaemonContainer** to manage all the logs from the APP containers  per EC2's
* X-Ray **Containers as a "Side Car****"
  * Every APP integrate a X-Ray Sidecar so X-Ray will work everywhere EC2 container
* Fargate Cluster use **Container as a Sidecar** 'cause we don't know the infrastructure behind

#### AWS CloudTrail
* Provides governance, compliance and audit for your AWS Account
* CloudTrail is enabled by default
* Get an history of events / API calls made within your AWS Account by:
  * Console
  * SDK
  * CLI
  * AWS Services
* Can put logs from CloudTrail into CloudWatch Logs
* If a resource is deleted in AWS, look first in CloudTrail

#### AWS CloudTrail vs CLoudWatch vs X-Ray
* CloudTrail:
  * Audit API calls made by users / services / AWS console
  * Useful to detect unauthorized calls or root cause of changes
* CloudWatch:
  * CloudWatch Metrics over time for monitoring
  * CloudWatch Logs for storing applications log
  * CloudWatch Alarms to send notifications in case of unexpected metrics
* X-Ray:
  * Automated Trace Analysiss & Central Service Map Visualitzation
  * Latency, Errors and Fault analysis
  * Request tracking across distributed systems
### AWS Integration & Messagin
When we start deploying multiple applications, they will inevitably need to communicate with one another  
There are two patterns of application communication:
* Synchronous communication (Application to application)
* Asynchronous / Event Based (Application to queue to application)
  
Synchronous between applications can be problematic if there are sudden spikes of traffic
* What happen if u need to suddenly encode 1000 videos but usulally it's 10?
  
In that case, it's better to **decouple** your applications
  * using SQS: queue model
  * using SNS: pub/sub model
  * using Kinesis: real-time streaming model
* These services can scale independenlty from our application

#### AWS SQS
##### AWS SQS - Standard Queue
* Oldest offering (over 10 years old)
* Fully managed
* Scales from 1 message per second to 10.000 per second
* Default retention of messages: 4 days, maximum of 14 days
* No limit to how many messages can be in the queue
* Low latency (<10 ms on publish and receive)
* Horizontal scaling in terms of number of consumers
* Can have duplicate messages (at least once delivery, occasionally)
* Can have out of order messages (best effort ordering)
* Limitation of 256KB per message sent
##### AWS SQS - Delay Queue
* Delay a message (consumers don't see it immediately) up to 15 minutes
* Default is 0 seconds (message is available right away)
* Can set a defualt at queue level
* Can override the default using the DelaySeconds parameter
##### AWS SQS - Producing Messages
* Define Body (up to 256kb)
* Add message attributes (metadata - optional)
* Provide Delay Delivery (optional)
* Get back
  * Message identifier
  * MD5 hash of the body
##### AWS SQS - Consuming Messages
Consumers Polls SQS for messages
* Receive up to 10 messages at a time
* Process the message within the visibility timeout
* Delete the message using the message ID & receipt handle
##### AWS SQS - Visibility timeout
When a consumer polls a message from a queue, the message is "invisible" to other consumers for a defined period... **The Visibility Timeout**
* Set between 0 seconds and 12 hours (default 30 seconds)
* If too high and consumer fails to process the message. We must **wait long time** until we can process the message again.
* If too low and consumers needs more time to process the message, **another consumer will receive the message** and the msg will be processed more that once
* **ChangeMessageVisiblity** API to change the visibility while processing a message
* **DeleteMessage** API to tell SQS the message was successfully processed
##### AWS SQS - Dead Letter Queue
* If a consumer fails to process a message within the Visibility Timeout... the message goes back to the queue.
* We can set a **threshold** of how many times a message can go back to the queue - it's called a "**redrive policy**"
* After the threshold is exceeded, the message goes into a dead letter queue (DLQ)
* We have to create a DLQ first and then designate it dead letter queue
* Make sure to process the messages in the DLQ before they expire

##### AWS SQS - Long Polling
* When a consumer requests message from the queue, it can optionally "wait" for messages to arrive if there are none in the queue
* LongPolling decreases the number of PAI calls made to SQS while increasing the efficiency and latency of your application.
* The wait time can be between 1 sec to 20 sec (Pref 20s)
* Long Polling is preferable to Short Polling
* Long polling can be enabled at the queue level or at the API level using **WaitTimeSeconds**
##### AWS SQS - FIFO Queue
* Name of the queue must end in .fifo
* Lower throiughput (up to 3,000 per second with batching, 300/s without)
* Messages are processed in order by the consumer
* Messages are sent exaclty once
* No per message delay (only per queue delay)
##### AWS FIFO - Features
* Deduplication: (not send the same message twice)
  * Provide a **MessageDeduplicationId** with your message
  * De-duplication interval is 5 minutes
  * Content based duplicatino: the MessageDeduplicationId is generated as the SHA-256 of the message body (not the attributes)

* Sequencing:
  * To ensure strict ordering between messages, specify a **MessageGroupId**
  * Messages with different Group ID may be recived out of order
  * E.g to order messages for a user, you could use the "user_id" as a group id
  * Messages with the same Group ID are delivered to one consumer at a time
##### AWS SQS Extended Client
* Message size limit is 256KB
* Using the SQS Extended Client (Java Library) you can send >256KB
1. Producer send small metadata message to SQS QUeue T=1
2. Producer also send large message to S3 T=1
3. Consumer pull metadata from SQS Queue T=2
4. Consumer knows that have to Retrive large message from S3 and pull it T=3

##### AWS SQS Security
* Encryption in flight using the HTTPS endpoint
* Can enable SS· (Server Side Encryption) using KMS
  * Can set the CMK (Customer Master Key) we want to use
  * Can set the data key reuse period (between 1 minute and 24 hours)
    * Lower and KMS API will be used often
    * Higher and KMS API will be called less
  * SSE only encrypts the body, not the metadata (message ID, timestamp, attributes)
* IAM policy must allow usage of SQS
* SQS queue access policy
  * Finer grained control over IP
  * Control over the time the requests come in
##### AWS SQS Must know API
* **CreateQueue, DeleteQueue**
* **Purge Queue**: Delete all the messages in queue
* **SendMessage**, ReceiveMessage, DeleteMessage
* **ChangeMessageVisiblity**: change the timeout
* **Batch APIs** for **SendMessage, DeleteMessage, ChangeMessageVisibility** help decrease your costs
#### AWS SNS Overview
If u want to send one message to many receivers u will use SNS Topic.
Works with the **Publish / Subscrive** strategy
* The "event producer" only sends message to one SNS topic
* As many "event receivers" (subscriptions) as we want to listen to the SNS topic notifications
* Each subscriber to the topic will get all the messages (note: *new feature to filter messages*)
* Up to 10,000,000 topics limit
* Subscribers can be:
  * SQS
  * HTTP / HTTPS (with delivery retries - how many times)
  * Lambda
  * Emails
  * SMS messages
  * Mobile Notifications
#### AWS SNS How to publish
* Topic Publish (within your AWS Server - using the SDK)
  * Create a topic
  * Create a subscription (or many)
  * Publish to the topic
* Direct Publish (for mobile apps SDK)
  * Create a platform application
  * Create a platform endpoint
  * Publish to the platform endpoint
  * Works with Google GCM, Apple APNS, Amazon ADM...

#### AWS SNS + SQS: Fan Out
* Push once in SNS, receive in many SQS
* Fully decoupled
* No data loss
* Ability to add receivers od data later
* SQS allows for delayed processing
* SQS alllows for retries of work
* May have many workers on one wueue and one worker on the other queue

### AWS Kinesis Overview
* **Kinesis** is a managed alternative to Apache Kafka
* Great for application logs, metrics, IoT, clickstreams
* Great for "real-time" big data
* Great for streaming processing frameworks (Spark, NiFi, etc...)
* Data is automatically replicated to 3 AZ

* **Kinesis Streams**: low latency streaming ingest at scale
* **Kinesis Analytics**: perform real-time analytics on stream using SQL
* **Kinesis Firehose**: load streams into S3, Redshift, ElasticSearch
##### AWS Kinesis Stream Overview 
* Streams are divided in ordered Shards/ Partitions *like roads*
* Data retentions is 1 day by default, can go up to 7 days
* Ability to reprocess / replay data
* Multiple applications can consume the same stream
* Real-time processing with scale of throughput
* Once data is inserted in kinesis, it can't be deleted (immutability)
##### AWS Kinesis Stream Shards
* One stream is made of many different shards
* 1MB/s or 1000 messages/s at write PER SHARD
* 2MB/s at read PER SHARD
* Billing is per shard provisioned, can have as many shards as you want
* Batching available or per message calls.
* The number of shards can evolve over time (reshard / merge)
* **Records are ordered per shard**
##### AWS Kinesis API - Put records
* PutRecord API + Partition key that gets hashed
* The same key goes to the same partition(helps with ordering for a specific key)
* Messages sent get a "sequence number"
* Choose partition key that is highly distributed (helps prevent "hot partition")
  * user_id if many users
  * **Not** country_id if 90% of the users are in one country
* Use Batching with PutRecords to reduce costs and increase throughput
* **ProvisionedThroughputExceeded** if we go over the limits
* Can use CLI, AWS SDK, or producer libraries from various frameworks
##### AWS Kinesis API - Exceptions
* Provisioned ThroughputExceeded Exceptions
  * Happens when sending more data (exceeding MB/s or TPS for any shard)
  * Make sure you don't have a hot shard (such as your patition key is bad and too much data goes to that partition)

* Solution:
  * Retries with backoff
  * Increase shards (scaling)
  * Ensure your partition key is a good one
##### AWS Kinesis API - Consumers
* Can use a normal consumer (CLI,SDK,etc...)
* Can use Kinesis Client Library (in Java, Node, Python, Ruby, .Net)
  * KCL uses DynamoDB to checkpoint offsets
  * KCL uses DynamoDB to track other workers and share the work amongst shards
##### AWS Kinesis KCL in Depth
Kinesis Client Library (KCL) is Java Library that helps read records from a Kinesis Streams with distributed appllications sharing the read workload  
* Rule: each shard is be read by only one KCL instance
* Means 4 shards = max 4 KCL instances
* Means 6 shards = max 6 KCL instances
* Progress is checkpointed into DynamoDB (need IAM access)
KCL can run on EC2, Elastic Beanstalk, on Premise Application
* Records are read in order at the shard level 
##### AWS Kinesis Security
* Control access / authorization using IAM policies
* Encryption in flight using HTTPS endopoints
* Encryption at rest using KMS
* Possibility to encrypt / Decrypt data client side (header)
* VPC Endpoints available for Kinesis to access within VPC
##### AWS Kinesis Data Analytics
* Perform real-time analytics on kinesis Stream using SQL
* Kinesis Data analytics:
  * Auto Scaling
  * Managed: no servers to provision
  * Continuous: real time
* Pay for actual consumption rate
* Can create streams out of the real-time queries
##### AWS Kinesis Firehose
Fully Managed Service, no administration
* Neas real time (60 seconds latency)
* Load data into Redshift / Amazon S3 / ElasticSearch / Splunk
* Automatic scaling
* Support many data format (pay for conversion)
* Pay for the amount of data going through Firehose
##### SQS vs SNS vs Kinesis
* SQS
  * Consumer "pull data"
  * Data is deleted after being consumed
  * Can have as many workers (consumers) as we want
  * No need to provision throughtput
  * No ordering guarantee (except FIFO queues)
  * Individual message delay capability
* SNS
  * Push data to many subscribers
  * Up to 10,000,000 subscribers
  * Data is no persisted (lost if not delivered)
  * Pub/sub
  * Up to 100,000 topics
  * No need to provision throughput
  * Integrates with SQS for fan-out architecture pattern
* Kinesis:
  * Consumers "pull data"
  * as many consumers as we want
  * Possibility to replay data
  * Meant for real-time big data, analytics and ETL
  * Ordering at the shard level
  * Data expires after X days
  * Must provisions throughput
##### Ordering data into SQS
  * For SQS stardard, there is no ordering.
  * For SQS FIFO, if you don't use a **Group ID**, messages are consumed in the order they are sent, **with only one consumer**
  * You want to scale the number of consumers, but you want messages to be "grouped" when they are related to each other
  * Then you use a Group ID (similar to Partition Key in Kinesis)
##### Kinesis vs SQS ordering
Let's assume 100 trucks, 5 kinesis shards, 1 SQS FIFO
* Kinesis Data Streams:
  * On Average you?ll have 20 trucks per shard
  * Trucks will have their data ordered within each shard
  * The maximum amount of consumers in parallel we can have is 5
  * Can receive up to 5 MB/s of data
* SQS FIFO
  * You only have one SQS FIFO queue
  * You will have 100 Group ID
  * You can have up to 100 Consumers (due to the 100 Group ID)
  * You have up to 300 messages per second (or 3000 if using batching)

### AWS Serverless: Lambda
##### Serverless in AWS
* AWS Lambda
* DynamoDB
* AWS Cognito
* AWS API Gateway
* Amazon S3
* AWS SNS & SQS
* AWS Kinesis Data Firehose
* Aurora Serverless
* Step Functions
* Fargate
##### AWS Lambda language support
* Node.js (JavaScript)
* Python
* Java (java 8 compatible)
* C# (.NET Core)
* Golang
* C# / Powershell
* Ruby
* Custom Runtime API (community supported, example Rust)
##### AWS Lambda Integrations Main Ones
* API Gateway - Create API REST
* Kinesis - data transformations on Fly
* DynamoDB - create some triggers
* Amazon S3 -  trigger events 
* CloudFront
* CloudWatch Events EventBridge
* CloudWatch Logs - to stream these logs wherever you want
* SNS -  react to notifications and your SNS topics
* SQS to ptrocess messages from your SQS queues
* Cognito - React whatever 
* ...
##### Lambda Synchronous Invocations
* Synchronous: CLI, SDK, API Gateway, Application Load Balancer
  * Results is returned right away
  * Error handling must happen client side (retries, exponential backoff, etc...)
* User Invoked:
  * Elastic Load Balancing (Application Load Balancer)
  * Amazon API Gateway
  * Amazon CLoudFront (Lambda@Edge)
  * Amazon S3 Batch
* Service Invoked:
  * Amazon Cognito
  * AWS Step Functions
* Other Services:
  * Amazon Lex
  * Amazon Alexa
  * Amazon Kinesis Data Firehose
##### Lambda Integration with ALB
To expose a Lamda function as an HTTP(S) endpoint ...
* You can use the Application Load Balacner (or an API Gateway)
* The Lambda function must be registered in a target group
##### ALB Multi-Header Values
ALB can support multi header values (ALB Setting)
  * **HTTP** -> http://example.com/path?**name**=*foo*&**name**=*bar*
  * **JSON** -> "queryStringParameters":{"**name**":["*foo*","*bar*"]}  
  
When you enable multi-value headers, HTTP headers and query string parameters that are sent with multiple values are shown as arrays withing the AWS Lambda event and response objects 
##### Lambda@Edge
You can use lambda to change CloudFront requests and responses:
1. After CloudFront Receives a request from a viewer (*viewer request*)
2. Before CLouFront forwards the querest to the origin (*origin request*)
3. After CloudFront receives the response from the origin (*origin response*)
4. Before CloudFront forwards the response to the viewer (*viewer response*)
##### Lambda@Edge Uses Case
* Website Security and Privacy
* Dynamic Web Application at the Edge
* Search Engine Optimizaiton (SEO)
* Intelligently Route Across Origins and Data Centers
* Bot Mitigation at the Edge
* Real-time Image Transformation
* A/B Testing
* User Authentication and Authorization
* User Prioritization
* User Tracking and Analytics
##### Lambda Asynchronous Invocations
* S3, SNS, CloudWatch Events...
* The events ar eplaced in an Event Queue
* Lambda attempts to retry on errors
  * 3 tries total
  * 1 minute wait after 1st, then 2 minutes wait
* Make sure the processing is **idempotent** (in cas of retries)
* If the function is retried, you will see duplicate logs entries in CloudWatch Logs
* Can define a DLQ (dead-letter queue) - SNS or SQS - for failed processing (need correct IAM permissions)
* Asynchronous invocations allow you to speed up the processing if you don't need to wait for the result (ex: you need 1000 files processed)
##### Lambda Asynchronous Invocations - Services
* AWS S3
* AWS SNS
* AWS CloudWatch Events / EventBridge
* AWS CodeCommit (CodeCommit Trigger: new branch, new tag, new push)
* AWS CodePipeline (invoke a Lambda function during the pipeline, Lambda must callback)
---- other ----
* Cloud Watch Logs (log processing)
* Amazon Simple Email Service
* AWS CloudFormation
* AWS Config
* AWS IoT
* AWS IoT Events
##### Lambda - Event Source Mapping
* Kinesis Data Streams
* SQS & SQS FIFO queue
* DynamoDB Streams
* Common denominator:
  * records need to be polled from the source
* Your Lambda function is invoked synchronously
##### Streams & Lambda (Kinesis & DynamoDB)
* An event source mapping creates an iterator for each shard, processes items in order
* Start with new items, from the beginning or from timestamp
* Processed items aren't removed from the stream (other consumers can read them)
* Low traffic: use batch window to accumulate records before processing
* You can process multiple batches in parallel
  * up to 10 batches per shard
  * in-order processing is still guaranteed for each partition key
##### Streams & Lambda Error Handling
By default if your function returns an error, the entire batch is reprocessed until the function succeeds, or the items in the batch expire.
* To ensure in-order processing, processing for the affected shard is paused until the error is resolved
* You can configure the event source mapping to:
  * Discard old events
  * restrict the number of retires
  * Split the batch on error (to work around Lambda timeout issues)
* Discarded events can go to a Destination
##### Lambda - Events Source Mapping SQS & SQS FIFO
Event Source Mapping will poll SQS (Long Polling)
* Specify batch size (1-10 messages)
* Recommended: Set the queue visibility timeout to 6x the timeout of your Lambda function
* to use a DLQ
  * set-up on the SQS queue, not Lambda (DLQ for Lambda is only for async invocations)
  * Or use a Lambda destination for failures
##### Queues & Lambda
Lambda also supports in-order processing for FIFO (first-in, first-out) queues, scaling up to the number of active message groups.
* For standard queues, items aren't necessarily processed in order.
* Lambda scales up to process a standard queue as quickly as possible.
* When an error occurs, batches are returned to the queue as individual items and might be processed in a different grouping than the original batch.
* Ocassionally, the event source mapping might receive the same item from the queue twice, even if no function error ocurred.
* Lambda deletes items from the queue after they're processed successfully
* You can configure the source queue to send items to a dead-letter queue if the can't be processed
##### Lambda Events Mapping Scaling
* Kinesis Data Streams & DynamoDB Streams:
  * One Lambda invocation per stream shard
  * If you use parallelization, up to 10 batches processed per shard simultaneously
* SQS standard:
  * Lambda adds 60 more instances per minute to scale up
  * Up to 1000 batches of messages processed simultaneously
* SQS FIFO:
  * Messages with the same GroupID will be processed in order
  * The Lambda function scales to the number of active message groups.

##### Lambda - Destinations
* Since Nov 2019 Can configure to send result to a destination
* Asynchronous invocations - can define destinations for successful and failed events:
  * Amazon SQS
  * Amazon SNS
  * AWS Lambda
  * Amazon EventBridge bus
  * AWS recommends you use destinations instead of DLQ now (but both can be used at the same time)
  * Event Source mapping: for discarded event batches
  * Amazon SQS
  * Amazon SNS
  * *Note*: you can send events to a DLQ directly from SQS

##### Lambda Execution Role (IAM Role)
* Grants the Lambda function permissions to AWS services / resources
* Sample managed policies for Lambda:
* AWS LambdaBasicExecutionRole - Upload logs to CloudWatch
* AWS LambdaKinesisExecutionROle - Read from Kinesis
* AWS Lambda DynamoDBExecutionRole - Read from DynamoDB Streams
* AWS LambdaSQSQueueExecutionRole - Read from SQS
* AWS LambdaVPCAccessExecutionRole - Deploy Lambda function in VPC
* AWSXRayDaemonWriteAccess - Upload trace data to X-Ray
* When you use an event source mapping to invoke your function, Lambda uses the execution role to read event data.
* Best practice: create one Lambda Execution Role per function
##### Lambda Resource Based Policies  
Use resource-based policies to give other accounts and AWS services permission to use your Lambda resources
* Similar to S3 bucket policies for S3 bucket
* An IAM principal can acces Lambda:
* If the IAM policy attached to the principal authorizes it (e.g. user access)
* OR if the resource-based policy authorizes (e.g. service access)
* When an AWS service like Amazon S3 calls your Lambda function, the resource-based policy gives it access

##### Lambda Environment Variables
* Environment variables = key / value pair in "String" form
* Adjust the function behavior without updating code
* The environment variables are available to your code
* Lambda Service adds its own system environment variables as well
* Helpful to store secrets (encrypted by KMS)
* Secrets can be encrypted by the Lambda service key, or your own CMK

##### Lambda Logging & Monitoring
* CloudWatch Logs:
  * AWS Lambda execution logs are stored in AWS CloudWatch Logs
  * Make sure your AWS Lambda function has an execution role with an IAM policy that authorizes writes to CloudWatch Logs
  * CloudWatch Metrics:
    * AWS Lambda metrics are displayed in AWS CLoudWatch Metrics
    * Invocations, Durations, Concurrent Executions
    * Error count, Success Rates, Throttles
    * Async Delivery Failures
    * Iterator Age (Kinesis & DynamoDB Streams)
* Lambda Tracing with X-Ray
  * Enable in Lambda configuration (Active Tracing)
  * Runs the X-Ray daemon for you
  * Use AWS X-Ray SDK in Code
  * Ensure Lambda Function has a correct IAM Execution Role
    * The managed policy is called AWSXRayDaemonWriteAccess
  * Environment variables to communicate with X-Ray
    * _X_AMZN_TRACE_ID: contains the tracing header
    * AWS_XRAY_CONTEXT_MISSING: by default, LOG_ERROR
    * AWS_XRAY_DAEMON_ADDRESS: the X-Ray Daemon IP_ADDRESS:PORT
##### Lambda VPCs
By default, your Lambda function is launched outside your oen VPC (in an AWS-owned VPC)
Therefore it cannot access resources in your VPC (RDS, ElastiCache, internal ELB...)
* To deploy Lambda in VPC:
  * You must define the VPC ID, the Subnets and the Security Groups
  * Lambda will create an ENI (Elastic Network Interface) in your subnets
  * AWSLambdaVPCAccessExecutionRole
* Permit Lambda in VPC access internet:
  * A Lambda function in your VPC does not have internet access
  * Deploying a Lambda function in a public subnet does not give it internet access or a public IP
  * Deploying a Lambda function in a private subnet gives it internet access if you have a **NAT Gateway / Instance**
  * You can use **VPC endpoints** to privately access AWS services without a NAT
