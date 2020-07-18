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