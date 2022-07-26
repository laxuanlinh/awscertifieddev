### What is `SWF`?
  - It's a service to coordiate between multiple components, it ensures each task is only assigned once, the states of tasks are stored in SWF so the components don't have to care about it
### `CloudFormation` launches `ASG` and `ECS` clusters
  - Need to specify name of the cluster in `etc/ecs/ecs.config` file to launch the ECS cluster
### `S3` bucket policy, `IAM role` for cross account access?
  - Resource based policy nad IAM role can only be shared between accounts within the same partition, not cross partitions. Ex: S3 bucket policies in `us-east` cannot be used in `ap-southeast`
### `CodeDeploy` when fails?
  - When a deployment fails, CodeDeploy will fall back to a last working version and deploy it to the failed instances
### `CodeDeploy Agent`, `Deployment Group`
  - `CodeDeploy Agent` is a software running on EC2 that allows it to be targeted by `CodeDeploy`
  - `Deployment Group` is a set of tagged EC2 instances, the group contains of some settings to be reused
### `AWS CLI CodeBuild` encryption
  - To encrypt a CodeBuild build using CLI, specify the param `CODEBUILD_KMS_KEY_ID` in the build command
### `DynamoDB` latency
  - DynamoDB global tables allow distribute replicas across regions that reduces latency
  - Eventually consistent read has less latency than strongly consistent read
  - Aside from that using DAX and reusing connection also help
### `CloudFormation` template `Outputs` section?
  - To export a resource, need to use `Export` field in `Outputs` section to specify the name of the resource
### Serverless app containers that share memory
  - Define multi containers in a task definition if we want these containers to share resources, lifecycle, data volume and can reference ports
  - If it's serverless then use Fargate, don't use EC2
### Lambda `Blue/Green` deployment:
  - One alias can point to 2 versions and assign weight to each version, it can rollback easily.
  - Don't use 2 aliases for 2 versions
### `ALB` and `SSL` - high CPU utilization
  - If serving traffic through HTTPS of `ALB` is under load then create an `HTTPS Listener` on ALB with `SSL termination` to use the certificate to force the request to be decrypted in client side before sending to `EC2` thus reduce the CPU utilization
  - Need to use `ACM`
### `ElastiCache` write-through?
  - Write through cannot write to the cache and the database at the same time but in sequence.
  - Writing to database then invalidating the cache is also a way to keep the cache up to date with database
### What is `Audit Trail` in `SSM`?
  - There is no such thing as `Audit Trail` in `SSM`, `SSM` can only be audited by `CloudTrail`
### `SQS` max messagge size
  - Max is 256KB
### What is `Organization Trail`? Does `CloudTrail` track only at bucket level? Access to `Organization Trail`
  - If we create an organization in `AWS Organizations` then we can have `Organization Trail` to track all accounts in it
  - `CloudTrail` can only track at bucket level by default, need to enable `S3 Event Stream` to trail object activities
  - Members can only see `Organization Trail` but cannot modify/delete or see trail log in `S3`
### What is the strategy that allows `ASG` to deploy applications to EC2?
  - To use `Blue/Green`, must use ALB so that ASGs of 2 versions can connect to the ALB
  - `In-place deployment` means deploy to existing EC2 instances, each instance is stopped then the new version is deployed and validated
### Where condition is used in `CloudFormation` template?
  - Conditions can be used in `Resources`, `Outputs`
  - Cannot be used in `Parameters`
### Input parameters in reuseable `CloudFormation` templates:
  - The input parameters is in `Parameters` section of the template
  - The reuseable template uses Ref to refer to the defined input `Parameters`
### `ALB` cross-zone balancing
  - `ALB` is always enabled with cross-zone balancing
  - `Sticky sessions` can cause unbalanced distribution because requests from 1 client keeps being routed to a single instance, also `long-lived TCP connection`
### Can ASG add volume to EC2 instance if the volume is approaching max capacity?
  - ASG cannot add volumes to EC2 instances, need to call EC2 API to do so, ASG can only add new instances
  - ASG is region bound so cannot span across regions, can span across AZs
### Terminate an container instance:
  - When terminate a `STOPPED` EC2 container instance, the instance is not deregistered from the cluster, need to manually deregister to remove
  - When terminate a `RUNNING` EC2 container instance, the instance is automatically removed from the ECS cluster

### `Cognito Use Pools` for `ALB` and `CloudFront`
  - `Cognito User Pools` can authenticate requests for `ALB` and `API Gateway` but not `CloudFront` and `Classic Load Balancer`
  - To use `Cognito User Pools` at `CloudFront`, need to use `Lambda@Edge`

### To migrate `Beanstalk` evironments from 1 account to another:
  - Create a saved configuration object from `Beanstalk`
  - Download the config object to local and edit parameters 
  - Upload to `S3` bucket of the other account
  - Create a new environment from the other account using the config in the bucket

### To improve Lambda performance, increase the RAM, increase timeout can help with long running tasks but not necessarily improve the performance

### Lambda Authorizer vs Cognito User Pools
  - Third party authentication => Lambda authorizer because it processes bearer token from OAuth or SAML
  - Cognito uses user db managed by AWS so it's not third party

### CDK vs SAM
  - CDK defines infra using programming languages like Python, Node, .NET ... 
  - SAM defines serverless infra using templates
  
### AWS Budget takes 5 weeks of usage data to generate budget forecast

### What is burstable performance on EC2?
  - It's a performance mode that allow t2, t3 ... instances to performe beyond the baseline.
  - If the account is less than 12 months old then t2 instance (or t3 in regions where t2 not available) burstable is free of charge

### The only resource-based policy that IAM supports
  - `Trust policy` is the only resource-based policy that IAM service supports
  - It specifies the condition for a principal to assume the role
  - An `IAM Role` usually contains `identity-based` prolicies and `trust policies`
  - `ACL` is a service to control which principals from another account can access resources, cannot control within the same account
  - `Permission boundary` is to set maximum permission an identity-based policy can grant to an IAM entity
  - `Organization SCP` is to set maximum permissions to an `Organization Unit`
### CloudFront Key Pair has to be created by root user

### Beanstalk vs CodeDeploy
  - Beanstalk can do in place deploy, it can do Blue Green but have to create a new environment and switch manually, Beanstalk also does not give much control over the deployment either
  - CodeDeploy can do Blue Green better, give much more control over the deployment
### What is Access Advisor?
  - It's to help audit service access, remove unnecessary permissions
  - `Trusted Advisor` is to advise you to follow best practices
  - `Amazon Inspector` auto detect security issues of the applications
  - `Access Analyzer` helps identify resources being shared with other accounts

### IAM users to access Billing and Budget
  - By default only root user can access Billing.
  - To give other users to access Billing, root user needst to first
    - Activate IAM user access
    - Attach IAM policies to the user
  
### Rolling with additional batches vs Immutable
  - Both ensure the availability
  - Rollin with additional batches takes more time but cheaper
  - Immutable takes less time and can quickly rollback but more expensive

### Dedicated Host vs Dedicated Instance
  - `Dedicated Instance` is on a hardware dedicated to a single customer, isolated from dedicated instances of other AWS account but can still shares with other instances of the same account
  - `Dedicated Host` is a physical dedicated server, allows running `license softwares` that bound to hardware, visibility to `sockets` and `physical cores`, more expensive than `Dedicated Instance`

### Immutable and Traffic splitting deployment can cause burst balance lost due to the instances are replaced in some cases

### IAM and ACM on SSL certificate
  - `ACM` is used to manage SSL certificates
  - In some regions where `ACM` is not available, use `IAM`
  - `CloudFormation` cannot configure SSL

### MFA for root users
  - Root user supports the following MFA methods, except `MFA SMS`
    - `Hardware MFA`
    - `Virtual MFA`
    - `U2F security key`: a USB plugged into computer

### Scaling based on SQS
  - When it comes to scaling instances polling from SQS, there are 2 recommended ways:
    - `Target tracking with backlog per instance`
    - `Target trakcing with acceptable backlog per instance`
  - The number of messages in SQS might not change to the size of Auto Scaling group because number of instances based on many factors, not just number of messages but also time to process, thus we cannot use `Step scaling`

### Security Group vs ACL
  - `SecGroup` are stateful
  - `ACL` are stateless and need to allow both inbound and outbound traffic, by default `ACL` allow all inbound and outbound traffic 

### Maximum S3 upload size?

### What is Firehose sink type?

### What is AWS Glue? Data Pipeline? EMR?

### Set default detailed monitoring for EC2?

### Critical instances? Zonal and regional

### CloudFormation parameter types?

### EC2 SSD types? IOPS?

### Internet Gateway? Subnet? Route table? Public internet?

### What is dynamodb::updateItem?

### What is signing API request?

### Lambda image max size?
