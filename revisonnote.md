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

### What is IAM Identity-based policies and what resource-based policy that IAM supports?

### Differences between dedicated instances and dedicated hosts

### What is EC2 burst balance

### What is U2F security key?

### What is CDK?

### Policy types that can only limit but cannot grant permissions

### What are Access Advisor and IAM Access Analyzer?