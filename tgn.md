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
* **Rolling**: update a few isntances at a time (bucket), and then move onto the next bucket one the fist bucket is healthy
  * Application is running below capacity
  * Can set the bucket size.
  * _Shut down half of ur system cause u are releasing the new verison and want to keep working your system with the rest of the old instances_
* **Rolling with addition batches** like rolling, but spins up new instances to move the batch (So that the old application is still available)
  * Application is running at capacity
  * Can set the bucket size
  * Application is running both versions simultaneously
  * Small additional cost
  * Addiotional batch is removed at the end of the deployment
  * Longer deployment
  * Great for prod

* **Immutable**: spins up new isntances in a new ASG, deploys versions to these instances and then swaps all the isntances when everything is healthy
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