---
#AWS Developer Certification Notes
##Table of Contets
1. AWS Fundamentals: IAM + EC2
2. AWS Fundamentals: ELB + ASG
3. ECS2 Storage - EBS & EFS
4. AWS Fundamentals: RDS + Aurora + ElastiCache
5. Route 53
6. VPC Fundamentals
7. Amazon S3 Introduction
8. loreipsum
9. loreipsum
##AWS Fundamentals: IAM + EC2
####AWS Regions
AWS has Regions all around the world. -> *ap-southeast-2*
Each Region has many availability zones -> *ap-southeast-2*__(a,b,c,..)__
Each availability zone has one or more discrete data centers with redundant power, networking, and connectivity
#### IAM 
Users -> for physical persons
Groups -> Permit group Users per Functions or Teams applying permissions to groups 
Roles -> for machines
Policies -> JSON documents that defines what Users/Groups/Roles actions can/cannot do
#### IAM Federation
Integrate their own repository of users with AWS IAM.
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
| Mac/Linux[^1]      | OK |  |OK|
| Windows      |       |   OK |OK|
| Windows 10 | OK      |    OK |OK|

[^1]: Mac/Linux must remember to restrict your .pem key using chmod 400 