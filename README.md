
# AWS Developer Certification Notes

Table of Contets  
[1. IAM + Security](#id1)  
[2. EC2 + ENI](#id2)  
[3. ELB](#id3)  
[4. ASG](#id4)  
3. ECS2 Storage:  EBS & EFS  
4. RDS + Aurora + ElastiCache  
5. Route 53  
6. VPC Fundamentals  
7. Amazon S3 Introduction  
8. loreipsum  
9. loreipsum  

### [IAM + Security]{#id1}

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
### EC2 + ENI {#id2}
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

### Elastic Load Balancers {#id3}
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
    
### Auto Scaling Group {#id4}
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








