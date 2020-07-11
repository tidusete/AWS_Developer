---
# AWS Developer Certification Notes
---
Table of Contets
1. AWS Fundamentals: IAM + EC2 {#id1}
2. AWS Fundamentals: ELB + ASG
3. ECS2 Storage :  EBS & EFS
4. AWS Fundamentals: RDS + Aurora + ElastiCache
5. Route 53
6. VPC Fundamentals
7. Amazon S3 Introduction
8. loreipsum
9. loreipsum

### [AWS Fundamentals: IAM + EC2] {#id1}

#### AWS Regions
AWS has Regions all around the world. -> *ap-southeast-2*  
Each Region has many availability zones -> *ap-southeast-2*__(a,b,c,..)__  
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
#### EC2 Introduction
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



