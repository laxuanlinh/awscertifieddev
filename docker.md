# Docker in AWS

- AWS ECR is a repository for Docker images
- AWS ECS and EKS is a platform for containers to run on
- Fargate is a serverless container platform that works with both EKS and ECS

## ECS
- When launching ECS, we're launching ECS tasks on RCS Clusters
### EC2 Launch Type
- For EC2 launch type, need to provision and maintain the infra
- Each EC2 instance will have a ECS Agent to launch ECS tasks and register with the cluster
- AWS will takes care of stating/stopping the container

![](ecsec2launchtype.png)

### Fargate Launch Type
- Don't have to provision the infra or manage EC2 instances
- It's serverless
- Just need to create task definitions, can increase number of tasks to scale.
- AWS will run these tasks with the specified CPU/RAM 
  
![](ecsfargatelaunchtype.png)

### IAM Roles for ECS
- `EC2 Instance Profile` is for `EC2 Instance Launch Type` only
- It's used by ECS agent to call different services like `CloudWatch`, `ECR`,` Secret Manager`, `Param Store`
- `ECS Task Role` is for both `EC2 instance` and `Fargate` launch type.
- It's used by tasks 
- Each task uses different role because they need to call different AWS services like S3, RDS ...
- `Task Role` needs to be defined in `Task Definition`

### Load Balancer Integration
- Can create an ALB in front of the ECS Cluster
- NLB is only recommended when lots of throughput is required
- ELB is supported but there is no point to use it
  
### Data Volume (EFS)
- EFS works for both EC2 and Fargate
- Tasks running in different AZ will share the same data
- Farget + EFS = Serverless\
- S3 cannot be mounted as a file system

### Create ECS Cluster
- Go to ECS => Create Cluster and specify the name
- It's possible to launch both Fargate(selected as default) and EC2 launch type
- If EC2 Launch type is also selected then need to specify the OS, instance type, number of instances
- After the cluster is created, in the Infrastructure tab, there should be 3 `Capacity Provider`:
  - 1 is to launch Fargate containers
  - 1 is to launch Fargate Spot containers (same as EC2 Spot fleet, cheaper but can be interrupted)
  - 1 is for AGS to launch EC2 instances
### Create Task Definition
- Go to `Task Definition`, specify name
- In the Container - 1 (can have multiple containers), specify name of the container and the URL to its image, click `Next`
- In the config env page, select to launch this on `Farget` or `EC2`, or both
- Select `CPU/RAM` for the task
- Specify the `IAM Task role` if the container needs to call external services, before that need to create an IAM Role and attach the policies accordingly
- Can mount volume if needed
- Click `Next` then `Create` 

### Deploy a service
- After creating a definition, to deploy, we need an ALB.
- To have an `ALB`, we need to define 2 security group in `EC2`:
  - 1 is for the `ALB` to allow external traffic coming in
  - 1 is for the task/service to allow traffic from the `ALB`'s security group only
- Go to ECS cluster => `Deploy`
- Select `Service` for a bunch of tasks or `Task` for just 1 Task
- Select the container image and the revision (version)
- In Load Balancer, select Application Load Balancer.
- Can use an existing ALB or create a new one
- Specify number of tasks (default is 1, if 0 then no task is running)
- In `Networking`, select the Security group for the task, this option will also be applied for the load balancer we just created, which is **WRONG** and needs to be fixed later
- Click `Deploy`
- While the creation of `Task`/`Service` is running, go to EC2 and check the `Load Balancer` we created (or using from an existing one)
- The security group of this `ALB` is the same one of the `Task`/`Service` which is **WRONG** and not public.
- Change the security group to the `ALB`'s security group that we created
- Now we have the `ALB` with security group allowing the public traffic and the `Service`/`Task` with the security group allowing only traffic from the ALB's security group
- After the service is deployed, use the DNS of the ALB to access
- To scale the Service/Task, go to Edit and change the Desired Tasks, if it's 0 then there is no task running

