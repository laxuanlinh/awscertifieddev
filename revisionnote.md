## What is `SWF`?
  - It's a service to coordiate between multiple components, it ensures each task is only assigned once, the states of tasks are stored in SWF so the components don't have to care about it
## `CloudFormation` launches `ASG` and `ECS` clusters
  - Need to specify name of the cluster in `etc/ecs/ecs.config` file to launch the ECS cluster
## `S3` bucket policy, `IAM role` for cross account access?
  - Resource based policy nad IAM role can only be shared between accounts within the same partition, not cross partitions. Ex: `S3` bucket policies in `us-east` cannot be used in `ap-southeast`
## `CodeDeploy` when fails?
  - When a deployment fails, `CodeDeploy` will fall back to a last working version and deploy it to the failed instances
## `Beanstalk` when fails?
  - Unlike `CodeDeploy`, when deployment fails, `Beanstalk` will fall back to the last working version and deploy it to the new instances and replace failed instances
## `CodeDeploy Agent`, `Deployment Group`
  - `CodeDeploy Agent` is a software running on EC2 that allows it to be targeted by `CodeDeploy`
  - `Deployment Group` is a set of tagged EC2 instances, the group contains of some settings to be reused
## `AWS CLI CodeBuild` encryption
  - To encrypt a CodeBuild build using CLI, specify the param `CODEBUILD_KMS_KEY_ID` in the build command
## `DynamoDB` latency
  - DynamoDB global tables allow distribute replicas across regions that reduces latency
  - Eventually consistent read has less latency than strongly consistent read
  - Aside from that using DAX and reusing connection also help
## `CloudFormation` template `Outputs` section?
  - To export a resource, need to use `Export` field in `Outputs` section to specify the name of the resource
## Serverless app containers that share memory
  - Define multi containers in a task definition if we want these containers to share resources, lifecycle, data volume and can reference ports
  - If it's serverless then use Fargate, don't use EC2
## `Lambda` `Blue/Green` deployment:
  - One alias can point to 2 versions and assign weight to each version, it can rollback easily.
  - Don't use 2 aliases for 2 versions
## `ALB` and `SSL` - high CPU utilization
  - If serving traffic through HTTPS of `ALB` is under load then create an `HTTPS Listener` on ALB with `SSL termination` to use the certificate to force the request to be decrypted in client side before sending to `EC2` thus reduce the CPU utilization
  - Need to use `ACM`
## `ElastiCache` write-through?
  - Write through cannot write to the cache and the database at the same time but in sequence.
  - Writing to database then invalidating the cache is also a way to keep the cache up to date with database
## What is `Audit Trail` in `SSM`?
  - There is no such thing as `Audit Trail` in `SSM`, `SSM` can only be audited by `CloudTrail`
## `SQS` max messagge size
  - Max is 256KB
  - Can use `SQS Extended Client Library` to extend to 2GB
## What is `Organization Trail`? Does `CloudTrail` track only at bucket level? Access to `Organization Trail`
  - If we create an organization in `AWS Organizations` then we can have `Organization Trail` to track all accounts in it
  - `CloudTrail` can only track at bucket level by default, need to enable `S3 Event Stream` to trail object activities
  - Members can only see `Organization Trail` but cannot modify/delete or see trail log in `S3`
## What is the strategy that involves `ASG` to deploy applications to `EC2`?
  - To use `Blue/Green`, must use ALB so that ASGs of 2 versions can connect to the ALB
  - `In-place deployment` means deploy to existing EC2 instances, each instance is stopped then the new version is deployed and validated
## Where condition is used in `CloudFormation` template?
  - Conditions can be used in `Resources`, `Outputs`
  - Cannot be used in `Parameters`
## Input parameters in reuseable `CloudFormation` templates:
  - The input parameters is in `Parameters` section of the template
  - The reuseable template uses Ref to refer to the defined input `Parameters`
## `ALB` cross-zone balancing
  - `ALB` is always enabled with cross-zone balancing
  - `Sticky sessions` can cause unbalanced distribution because requests from 1 client keeps being routed to a single instance, also `long-lived TCP connection`
## Can ASG add volume to EC2 instance if the volume is approaching max capacity?
  - ASG cannot add volumes to EC2 instances, need to call EC2 API to do so, ASG can only add new instances
  - ASG is region bound so cannot span across regions, can span across AZs
## Terminate an container instance:
  - When terminate a `STOPPED` EC2 container instance, the instance is not deregistered from the cluster, need to manually deregister to remove
  - When terminate a `RUNNING` EC2 container instance, the instance is automatically removed from the ECS cluster

## `Cognito Use Pools` for `ALB` and `CloudFront`
  - `Cognito User Pools` can authenticate requests for `ALB` and `API Gateway` but not `CloudFront` and `Classic Load Balancer`
  - To use `Cognito User Pools` at `CloudFront`, need to use `Lambda@Edge`

## To migrate `Beanstalk` evironments from 1 account to another:
  - Create a saved configuration object from `Beanstalk`
  - Download the config object to local and edit parameters 
  - Upload to `S3` bucket of the other account
  - Create a new environment from the other account using the config in the bucket

## To improve `Lambda` performance, increase the RAM, increase timeout can help with long running tasks but not necessarily improve the performance

## `Lambda` Authorizer vs Cognito User Pools
  - Third party authentication => `Lambda` authorizer because it processes bearer token from OAuth or SAML
  - Cognito uses user db managed by AWS so it's not third party

## CDK vs SAM
  - CDK defines infra using programming languages like Python, Node, .NET ... 
  - SAM defines serverless infra using templates
  
## AWS Budget takes 5 weeks of usage data to generate budget forecast

## What is burstable performance on EC2?
  - It's a performance mode that allow t2, t3 ... instances to performe beyond the baseline.
  - If the account is less than 12 months old then t2 instance (or t3 in regions where t2 not available) burstable is free of charge

## The only resource-based policy that IAM supports
  - `Trust policy` is the only resource-based policy that IAM service supports
  - It specifies the condition for a principal to assume the role
  - An `IAM Role` usually contains `identity-based` prolicies and `trust policies`
  - `ACL` is a service to control which principals from another account can access resources, cannot control within the same account
  - `Permission boundary` is to set maximum permission an identity-based policy can grant to an IAM entity
  - `Organization SCP (service control policies)` is to set maximum permissions to an `Organization Unit`
## `CloudFront` Key Pair has to be created by root user

## `Beanstalk` vs CodeDeploy
  - `Beanstalk` can do in place deploy, it can do Blue Green but have to create a new environment and switch manually, `Beanstalk` also does not give much control over the deployment either
  - `CodeDeploy` can do Blue Green better, give much more control over the deployment
## What is `Access Advisor`?
  - It's to help audit service access, remove unnecessary permissions
  - `Trusted Advisor` is to advise you to follow best practices
  - `Amazon Inspector` auto detect security issues of the applications
  - `Access Analyzer` helps identify resources being shared with other accounts

## IAM users to access Billing and Budget
  - By default only root user can access Billing.
  - To give other users to access Billing, root user needst to first
    - Activate IAM user access
    - Attach IAM policies to the user
  
## `Rolling with additional batches`  vs `Immutable` 
  - Both ensure the availability
  - Rollin with additional batches takes more time but cheaper
  - `Immutable`  takes less time and can quickly rollback but more expensive

## `Dedicated Host`  vs `Dedicated Instance` 
  - `Dedicated Instance` is on a hardware dedicated to a single customer, isolated from `Dedicated Instance` s of other AWS account but can still shares with other instances of the same account
  - `Dedicated Host` is a physical dedicated server, allows running `license softwares` that bound to hardware, visibility to `sockets` and `physical cores`, more expensive than `Dedicated Instance`

## `Immutable`  and `Traffic splitting` deployment can cause burst balance lost due to the instances are replaced in some cases

## IAM and `ACM` on SSL certificate
  - `ACM` is used to manage SSL certificates
  - In some regions where `ACM` is not available, use `IAM`
  - `CloudFormation` cannot configure SSL

## MFA for root users
  - Root user supports the following MFA methods, except `MFA SMS`
    - `Hardware MFA`
    - `Virtual MFA`
    - `U2F security key`: a USB plugged into computer

## Scaling based on SQS
  - When it comes to scaling instances polling from SQS, there are 2 recommended ways:
    - `Target tracking with backlog per instance`
    - `Target trakcing with acceptable backlog per instance`
  - The number of messages in SQS might not change to the size of Auto Scaling group because number of instances based on many factors, not just number of messages but also time to process, thus we cannot use `Step scaling`

## Security Group vs ACL
  - `SecGroup` are stateful, if it allows an incoming request, the response to that request is also allowed, regardless of Outgoing rules
  - `ACL` are stateless, the incoming and outgoing request/response are independent
  - By default, a `VPC` is attached to a default `NACL`, this default `NACL` `allows` all inbound and outbound traffic
  - If we create a custom `NACL` to attach to a subnet, by default it `denies` all inbound and outbound traffic

## EBS volume cannot be used in another AZ?
  - EBS volumes are AZ locked, it's not possible to use in another AZ

## Maximum `S3` upload size?
  - Max size is 5TB but per upload is 5GB

## What is Firehose sink type?
  - `Firehose` can support streaming data to services like `Redshift`, `S3`, `Splunk`, `Amazon Elastic Search` but cannot stream to `ElastiCache`

## CodeBuild log
  - By default `CodeBuild` logs about status of builds, this log is in `CloudWatch Logs`, if integrated with `S3` then the log can be moved to `S3` and later analyzed by `Anthena`
  - `CloudTrail` can also integrate with `CodeBuild` but it only records the action and API from users, not the statuses of builds

## How to backup DynamoDB locally?
  - `Hive`, `Glue`, `Data Pipeline` and `EMR` can be used to extract some tables of `DynamoDB` and store in your `S3` bucket for backup
    - `AWS Glue`: serverless extract, transform and load (ETL) data
    - `Apache Hive`: data warehouse, read, write, manage large amount of data
    - `AWS Data Pipeline`: process, transform and move data between services like `S3`, `DynamoDB`, `RDS` ...
    - `AWS EMR`: big data
  - `DynamoDB` has 2 built-in backup methods `on-demand` and `point-in-time` but both of these write to an `S3` bucket that you don't have access to so cannot download to local

## Benefits of Reserved Instances
  - Each reserve instance is discounted in the first 60 mins.
  - If we run multiple instances but have only 1 reserved instance then the benefit time is divided across running instances

## Set default detailed monitoring for EC2?
  - By default, basic monitoring for EC2 is enabled when launch from `Management Console`
  - Detailed monitoring is enabled by default when use `CLI` or `SDK`

## Critical instances? Zonal and regional
  - Purchase reserved instance for an AZ => `Zonal instance`, discount for a fixed size and `reserved capacity`
  - Purchase reserved instance for a region => `Regional instance`, discount for flexible size but `NO reserved capacity`

## `Lambda` concurrency modes?
  - `Reserved Concurrency`: since `Lambda` limits 1000 concurrent instances running per account, `Reserved Concurrency` ensures that a function has an amount of concurrency that no other functions will use, this also stops a function from bottlenecking other functions' concurrency
  - `Provisioned Concurrency`: init a number of instances when invoked, when there are too many requests, instead of spawning new instances to process, `Lambda` can just use the init instances available, this helps with latency to serve requests

## `CloudFormation` parameter types?
  - `String`, `Number`, `List<Number>`, `CommaDelimitedList`
  - `AWS::EC2::KeyPair`, `SecurityGroup`, `SubNet`, `VPC`
  - `List<AWS::EC2::VPC::Id>`, `List<AWS::EC2::SecurityGroup::Id>`, `List<AWS::EC2::SubNet::Id>`

## EC2 SSD types? IOPS?
  - For `gp2` SSD, more volume = more IOPS, IOPS increase linearly from 0 -> 5,334GB (16.000IOPS), after that, larger volume is capped at 16.000 IOPS, cannot go higher
  - For `io1`, the provisioned max throughput ratio to volume size is 50:1, which means 50GB has a max IOPS 50*50=2500IOPS, 200GB = 10000IOPS, no cap

## `CloudFront` signed URL
  - CF uses `Signed URL` to control who can access the content with expiration, it can only manage access of `a single file`
  - `Signed Cookies` can also control access but for `multiple files`.
  - `Signed URL` is more specific and takes prededence over `Signed Cookies`
  - Can only have up to 2 key pairs
  - The `public key` is with `CloudFront` and `private key` is to sign portion of the URL, the other portion is signed by `key ID`
  - The `public key` uploaded to `CF` must be assigned to a `trusted signer`

## Internet Gateway? Subnet? Route table? Public internet?
  - `Internet Gateway` is a service to connect VPC to public internet
  - `Route Table` contains of rules (routes) of where traffic from subnet, gateway goes
  - A subnet can only be associated with 1 table at a time, by default if not explicitly associated with any table, it's associated with the `Main Route Table`
  - To access `Internet`, the `Subnet` of an `EC2` should be routed to an `Internet Gateway` in `Route Table`
  - If a `Subnet` is routed to a `Internet Gateway`, it's a `public Subnet`, if not then it's a `private Subnet`

## API Gateway security methods?
  - `Resource-based policy`: allows or denies AWS accounts or IP addresses that can invoke APIs
  - `IAM roles and policies`: allows which principals and execution units can create, deploy, manage and call APIs, can use with `IAM tags`
  - `Lambda Authorizer` and `Cognito User Pools`
  - `Endpoints policies for interface VPC endpoints`: attach resource-based policies to interface VPC endpoints to improve security of private APIs

## KMS Encryption using `Lambda`
  - To encrypt data larger than 4KB, `Lambda` needs to use `Envelop Encryption` and store the encrypted data as a file because the max size of env variable in `Lambda` is 4KB (probably to use KMS' Encrypt API)

## VPC FLow Logs can be created for both VPC and subnets, which ever it's created for is monitored by the flow log

## What is dynamodb::updateItem?
  - Update an existing item or create a new item if it doesn't exist

## What is signing API request?
  - When sending requests to AWS, we need to sign the request using access key
  - If we're using `CLI` or `SDK`, they will sign for us automatically
  - Don't need to sign anonymous requests to `S3 endpoints`
  - We need to manually sign `HTTP requests` when the programming language does not have `SDK` support

## `Lambda` image max size?
  - 10GB
  - Containers running on `Lambda` must implement `Lambda` API
  - The images must be created in ECR by the same AWS account as `Lambda`'s account
  - Can test locally using `Lambda Runtime Interface Emulator`

## API Gateway HTTP
  - HTTP is for `Lambda` and other services of AWS like HTTP endpoints of applications
  - `API Gateway` uses `Lambda Authorizer`, `Cognito` and `IAM policies` but not `AWS WAF`

## Firehose vs Data Stream
  - `Firehose` is to load data stream to other AWS services like `Redshift`, `S3`, `ElasticSearch`, it's fully managed and cheaper
  - `Data Stream` is to ingest data stream and pass down to customized applications, it's managed by AWS but still needs config for shards

## `Step Function` Express and Standard workflow
  - `Standard `Step Function` Workflow `supports all service integrations, design pattern ... long running workload up to 1 year
  - `Express `Step Function` Workflow` does not support all integrations or pattern, short burst workload for 5 mins

## `S3` Object Ownership setting
  - By default when another account upload to your bucket, that account own the object and can use `ACL` to grant access to that object
  - `Object Ownership` is a bucket-level setting that allows you to disable this and own all objects inside the bucket thus centralize the access using policies

## `GenerateDataKey` vs `GenerateDataKeyWithoutPlainText`
  - Their operations are the same except `GenerateDataKeyWithoutPlainText` returns an encrypted version of the key, this is useful when you don't want to encrypt rightaway and save it for later
  - When use encrypted data key, call `Decrypt` API to decrypt the data key first, the rest is similar to `GenerateDataKey`
  - `GenerateDataKeyWithPlaintext` doesn't exist, by default `GenerateDataKey` is already plaintext

## Query authentication is useful to authenticate signed URL to `S3`

## `S3` bucket replication
  - `Same-Region Replication` and `Cross-region Replication` can be config at `bucket level`, at `shared prefix level` or at `object level` using tag
  - The object lifecycle actions are not replicated

## `CloudWatch` resolutions
  - `Basic Monitoring` sends data to `CloudWatch` every `5 mins`
  - `Detailed Monitoring` sends every `1 min`
  - `Custom metric` high-resolution can send metrics every `5/10/30/60 secs`
  - `Basic` and `Detailed Monitoring` cannot send CPU and Memory usage, needs to use `CloudWatch Agent`

## State Machine and Activities in `Step Function`
  - `State Machine` is a JSON file to orchestra tasks
  - `Activity` is a feature that allows tasks in a S`tate Machine` to have work done somewhere else like `EC2` instances

## `CodeDeploy` Agent usage
  - Allows `EC2` instances to be used by `CodeDeploy`
  - It can store app revision, logs and also can clean up

## `CodeDeploy` auto rollback default behaviors?
  - By default `CodeDeploy` deploys the last working version to a new deployment with a new ID

## `Lambda` Custom Sources? CW Event Rules?
  - If an AWS service does not have direct connection to `Lambda`, can create a rule in CW Event to invoke `Lambda` functions
  - There is no such thing as `Lambda` Custom Sources

## `CodeDeploy` lifecycle event order?
  - App stop => Download Bundles => BeforeInstall => Install => AfterInstall => AppStart => ValidateService => BeforeBlockTraffic => BlockTraffic => AfterBlockTraffic => BeforeAllowTraffic => AllowTraffic

## CodeCommit encryption?
  - Repositories are encrypted at rest by default

## Kinesis Agent vs Kinesis Producer Library
  - Both Agent and KPL are to send data to Data Stream but Kinesis Agent is better

## CloudTrail `S3` cross-account
  - The owner of a bucket can only receive `CloudTrail` access log if he is also the owner of the object

## DynamoDB scan large table
  - It's recommended to use `Parallel Scan` when query a large table.
  - `Filter Expression` does not help speed this up because it only filters after the `Scan` is complete

## CodePipeline automation?
  - When create source of `CodePipeline` in console, `CodePipeline` automatically creates a `CW Event` to watch for code changes in `CodeCommit` or `S3`

## Enable KMS encryption in requests to `S3`
  - When SSE:KMS encryption is enforced in `S3`, the request to put objects needs to include header `'x-amz-server-side-encryption:'aws:kms'`
  - *NOT* `'x-amz-server-side-encryption:'SSE:KMS'`

## Redis with cluster mode
  - `Redis` is region bound regardless of cluster enabled or disabled
  - `Redis` cluster mode allows data to be partitioned into `1-500` shards, if the mode is disabled then there is only 1 shard
  - But when `enabled`, modifiability is more limited, can't manually promote a replica node to a primary node
  - Data sync between primary and replica nodes is done `asynchronously` regardless of cluster mode `enabled` or `disabled`

## Cross account permissions
  - To grant cross account permissions for Account B to access Account A's resources:
    - `Account A` creates a new role and attaches an `identity-based policy` to it to access the resource
    - `Account A` adds a `trust policy` to identify `Account B` as the princile that can assume the role
    - `Account B` then delegates permission to assume the role to any user of this account

## Options for real-time application backend?
  - Use websocket on `Gateway API`, `DynamoDB` with `Lambda`
  - Use `AppSync` because it can create backend using schemas and also support real-time

## SES vs SNS
  - SES can only send email
  - SNS can send email and SMS

## Elastic IP?
  - Elastic IP is a reserved public IP
  - If we run a DNS server on an EC2, we need a public IP

## What is CodeStar?
  - It's a service that group all CI/CD services together
  - Can provide a CI/CD pipeline from start to finish in a few mins and provides a monitor dashboard

## How to deploy X-Ray to Fargate
  - Create a Docker image and upload to ECR and deploy to ECS cluster
  - Provide the container with proper role

## Which property ensure X-Ray Daemon is discovered on ECS?
  - `AWS_XRAY_DAEMON_ADDRESS` this is to set the host and port of daemon listener

## How to centralize X-Ray cross-account management?
  - Install `X-Ray deamon`
  - Create a role in target unified account and allow all users to assume it
  - Configure `X-Ray deamon` to assume the role

## What is `S3` transfer acceleration?
  - Transfer Accelerator is a service that helps speed up the upload processing using edge locations

## How to define `Lambda` functions in `CloudFormation` templates?
  - To define `Lambda` functions in CF templates:
    - Upload `Lambda` code as zip to `S3` and refer to its key in field `AWS::Lambda::Function`
    - Write the `Lambda` code inline in field `AWS::Lambda::Function` as long as it does not have dependencies

## SQS `MessageGroupID` vs `MessageDeduplicationID`?
  - If a message with `MessageDeduplicationID` is sent successfully, any following messages with the same `MessageDeduplicationID` are accepted but will not be sent in the next 5 mins
  - `MessageGroupID` is to group messages together and order them, messages in the same group are processed one by one

## How to paginate `AWS CLI` API calls?
  - `--starting-token` and `--max-items`
  - `--page-size` specifies the number of result each CLI request retrieves, CLI still makes multi calls to retrieve full list thus this should only be done on a small data set.

## How to enable `S3` web hosting?
  - We need to enable `Public access` of the bucket
  - Beside that we may also need to create a bucket policy to allow public access

## Gateway endpoint vs Interface endpoint in VPC
  - `Endpoints` are virtual devices and components of VPC, they are highly available, they helps connect VPC to other AWS services
  - There are 2 types of endpoints:
    - `Gateway endpoint`: connect to `S3` and `DynamoDB`
    - `Interface endpoint`: connect to everything else
  - There is no such thing as `S3 API`, only `API Gateway` has API

## ELB 5xx errors
  - `500 Internal error`: application errors or ACL errors
  - `501 Not implemented`: Transfer-Encoding header with invalid value
  - `502 Bad gateway`: connected to server but having error with the connection
  - `503 Service unavailable`: has not registered with any target group

## Does MySQL offer storage auto scaling?
  - Yes, all DBs in RDS does

## API gateway authentication using DynamoDB
  - Use `Lambda Authorizer`.
  - `Cognito` uses `LiteSQL`, not `DynamoDB`

## Can run CodeBuild locally using `CodeBuild Agent` to build and debug

## Can build locally using `CodeBuild Agent` as a `cost-effective` way to test the build before building on server

## `S3` strongly consistent data model?
  - By default `S3` applies `Strongly consistent` data model to objects in all buckets for `PUT` and `DELETE` operations
  - If we delete an object and immediately access it, `S3` will return nothing
  - If we delete a bucket and list all buckets, the deleted bucket still appears because buckets have `Eventual consistency model`

## Replacing data and immediately access it in `S3`?
  - Since objects in buckets have strongly consistent data model by default, replacing or deleting objects then immedately accessing will always returns the latest version

## `CloudFront` Origin Group failover?
  - To set up `Origin Group`, need to create at least `2 origins` and add them to the group, 1 is primary and 1 is secondary
  - When a cache hit, `CF` returns the file but when cache miss, it routes to `Primary origin`, if the `Primary origin` fails then it routes to `Secondary origin`
  - All traffic to `CF` is routed to `Primary origin` if cache miss even when the previous request failed over to `Secondary origin`

## How CloudTrail manages EBS in EC2 ?
  - If EBS is created along with EC2 then the event `CreateVolume` is not recorded by `CloudTrail`

## Set DeleteOnTermination on EBS while EC2 is running
  - Can use DeleteOnTermination attribute in CLI command while the instance is running

## EBS encryption
  - If enable encryption for a region then there is no way to change it for individual EBS volume

## How to configure redirect for ALB using CLI?
  - Can use `query-string` with key value pairs to match the URL

## RDS multi AZ features
  - `RDS` performs OS updates on the `standby` instance then promote it to `primary`, the old `primary` is now a new `standby` and OS updated 
  - `Primary` and `standby` are `synchonously` replicated so that `standby` always has the latest data
  - `Read replicas` are replicated `asynchronously`

## EFS classes
  - Standard: frequently accessed
  - Standard Infrequent Access (IA): infrequent access
  - One zone: also frequenly access but not as highly available
  - One zone Infrequent Access (IA): also infrequent access but not as highly available
  - File storage + concurrently accessed by EC2 => EFS

## Max number of SQS messagges can be retrieved at the same time?
  - 10

## How to use Swagger file?
  - Can deploy SAM template with inline Swagger definition 
  - Or can define a Swagger file and refer it in SAM template
  - Basically SAM deploys Swagger

## Access key best practices
  - Remove all or not create any root account access keys
  - Use IAM Role where possible
  - Don't embed it in code, rotate periodically, remove unused keys, use MFA

## How to securely transfer data between EC2 and EBS?
  - Enable EBS encryption
  - Use IAM role to limit access
  - **Creating encrypted EBS volume is useful to move between AZs but has nothing to do with transfer data between EC2 and EBS, don't select this**

## How to define a `Beanstalk` multi container Docker environment?
  - A `Beanstalk multi container Docker env` requires a `Dockerrun.aws.json` file, which is an `ECS Task Definition` file even though `ECS` has nothingg to do with it

## What to do when get ProvisionedThroughputExceededException?
  - Modify the application to use AWS SDK 
  - Modify the app to use exponential backoff
  - **DO NOT** increase write and read throughput if the issue is only write or read

## How to use X-ray in ECS containers?
  - Build the containers from `amazon/aws-xray-daemon` base image

## `Lambda` function's iterator age metric growing and processing time more than expected?
  - Increase the performance by `increasing RAM` of `Lambda`
  - Increase stream throughoutput by `increasing shards`

## `CloudWatch` filter only works for the events that happens after it's created

## To check the impact on resources when updating `CloudFormation` templates, use change sets

## `CloufFormation` `EC2` instances slow to set up?
  - Create an `AMI` with installed software to reduce boot time
  - Update the template to include the `AMI`

## What `FIFO` queues have anything to do with deduplication?
  - Probably nothing, other answers are just bad

## `API Gateway` generates API key => User receives 403 Forbidden ?
  - `createUseagePlanKey` to associate the newly created API key with the useage plan
  - You create keys using `createApiKeys` or `importApiKeys`, cannot be both, so this option is not correct

## `KMS` key rotation
  - Key rotation is only supported for custom keys if they're generated within `AWS KMS HSM`s
  - Imported custom keys, asymmetric or keys generated in `CloudHSM` clusters are not supported
  - If use imported or asymmetric keys then must manually rotate

## How to enable `CORS` when integrate `S3` web with `API Gateway`
  - Needs to enable `CORS` on `API Gateway`

## Can restrict access to some items in DynamoDB table using primary keys

## How to validate IAM access to a service?
  - Use `--dry-run` or `IAM policy simulator`

## What is cache aside strategy?
  - It's lazy-loading

## To access a `CodeCommit` git repo need a `Git credentials helper` to use an `AWS credential profile` to access the repo path

## `Lambda` with `X-Ray`?
  - Provide execution role with permissions
  - Enable auto tracing for the `Lambda` function

## To put multiple `CloudWatch` `metrics` on a graphical presentation screen, use `namespace`

## If asked about install `Memcache` on `EC2` vs `ElastiCache Redis`?
  - Select `Redis`, manually installing `Memcache` on `EC2` takes lots of effort

## Use `AWS DocumentDB` to migrate `MongoDB` to AWS

## `Beanstalk` deployment no downtime and exclusive access logs on existing servers?
  - Rolling

## Reduce Lambda cold start time?
  - Reduce the dependencies
  - Increase RAM to increase CPU

## Encrypt fields in CloudFront?
  - Use `field-encryption`
  - Specify the set of fields to encrypt in the POST request
  - Can encrypt up to 10 fields

## For `Lambda` to create custom metrics from `CloudWatch`?
  - Use `CloudWatch` metric filter to search for phrases in the logs
  - These metrics are `custom metrics` and can be used to graph or create `Alarms`
  - By default `Lambda` writes log to CW so no need to config that

## `S3` encrypted with `KMS` and throttled?
  - Perform exponential backoff
  - Contact `AWS Support` to increase `KMS rate limit`

## `Immutable` update policy on `Beanstalk` fails?
  - During immutable deployment, another ASG is created which can cause running out of on-demand instance limit

## The provisioned write capacity for each GSI must be equal or greater than the base table, otherwise throttling can occur

## To assume role in crossed account using assume command, need to config the access key, secret and token in the environment variables

## Services that manage users and allow to change password?
  - `Cognito User Pool` and `Cognito Identity Pool`

## Ordering in `Kinesis Data Stream`?
  - FIFO if processing 1 shard
  - Not guarantee if processing multi shards in parallel

## How to trigger `CodeDeploy` when the application is upgraded?
  - Update the version of the app in `appspec.yml` to the latest
  - Can use env variables so that it can do it automatically, not sure why the question does this manually, not the best practice

## How DB from Account B allows access for app from Account B?
  - Create role in Account A that allows the DB and add Account B as a trusted entity
  - In Account B allow the `EC2` `IAM` role to assume the DB access role with prefined SCP
  - Include `AssumeRole` API in the app so the app can assume the role

## To config web hosted on S3 to have HTTPS endpoints
  - Create `CloudFront` distribution and add S3 as an origin
  - Configure `CloudFront` with SSL/TLS cert
  - CANNOT config ELB and S3

## `Kinesis Data Stream` vs `SNS`+`SQS`
  - `Kinesis Data Stream `is recommended when the requirement is multi apps consume same stream concurrently
  - `SQS` when messaging semantics, visibility timeout, message delay, dynamically increase throughput at read time, scale transparently

## Move stateful web app to AWS 
  - Store session data in `DynamoDB`
  - Use `ELB` and `CloudFront`

## In case of failure, `CodePipeline` can send notification to `CloudWatch Event`

## To include `Input` in the `Output` of `Step Functions`?
  - Use `ResultPath` in `Catch` statement to include error with input

## Choosing the polling timeout 
  - If `short polling` for 10s, if the message comes at 15s then the message can only be consumed by the next polling which is at 20s
  - IF `long polling` for 20s and the message comes at 15s, the message is consumed at 15s and connection closed, thus faster than `short polling` for 10s

## Execute large number of `CLI` commands => use pagination

## Route `API Gateway` based on action key in the JSON request?
  - If the request contains action key like: `get`, `create`, `update`, `API Gateway` can route based on this key by setting the value of route selection to `$request.body.action`

## `Lambda` functions of `Lambda@Edge` can only be in `US East Region`

## System with different components that scale separately on `Beanstalk`?
  - Best to deploy components to different `Beanstalk` environments because each environment is optimized for a type of application.
  - By specifying different tier for different components, `Beanstalk` can use the right capacity and scale accordingly

## To set alert on `CloudWatch` using `CLI`, use `put-metric-data` command

## Update application in `Beanstalk`?
  - Zip the application and upload to `Beanstalk` console or use `CLI`
  - No need to rebuild or build new environment

## How many keys are used in `Envelop Encryption`?
  - 2 keys, `Customer Master Key` is to encrypt/decrypt `Data key`
  - `Data key` is to encrypt data

## Which service uses JWT?
  - After a user logs in, `Cognito User Pool` returns a JWT
  - `Cognito Identity Pool` can use this JWT to grant access to other services

## `X-Ray daemon` vs `X-Ray SDK` vs `X-Ray Agent`
  - `X-Ray SDK` is to put segments in API requests
  - `X-Ray Daemon` is to send data back to `X-Ray` API for visualization
  - `X-Ray Agent` is a standalone Java app which auto instrument the code with minimum effort, it can also assume role to send data to a centralize account for management
  - When deploying to EC2 or on-premises, install `X-Ray daemon/agent` and instrument the code, no need to install on `ECS` and `Beanstalk`

## `Cognito User Pool` customize logo
  - Create a login page in `Cognito User Pool`
  - Customize the logo there
  - No need to create a custom login page

## Parallel different functions in `Step Function`?
  - Use `Map state`
  - `Parallel state` can also do parallel but all functions have to have the same input while `Map` can also parallel but not does not have to take the same input

## Can scale Redis by simply adding read replicas, no need to do cluster

## Push notification about new update between mobile devices?
  - Use `push synchronization` with appropriate `IAM Role`
  - This is a feature by Cognito using SNS to inform users about new data.

## How to order SQS?
  - Configure each sender to have a unique `MessageGroupID`, this would help order all messages from a sender
  - Do not configure each message with a `MessageGroupID`

## Centralize `Lambda functions` from multiple accounts?
  - `Lambda functions` from multi accounts can subscribe to 1 `SNS` topic, the topic will invoke functions
  - Cannot use `SQS` because `SQS` does not invoke `Lambda functions`

## CodeDeploy hooks useages?
  - `DownloadBundle` and `Install`: agent doing stuff, users can't do anything
  - `AfterInstall`: can configure app or change file permissions
  - `ApplicationStart`: can restart services that were stopped during `ApplicationStop`
  - `ValidateService`: can verify 

## Config lifecycle policy of `Beanstalk` to a limited number of version but zip files in `S3` still deleted anyway?
  - In `Beanstalk` console => `Application version lifecycle settings` => `Retention` => select not to delete the bundle when the app version is deleted

## Redirect objects in S3 to another URL?
  - It's possible to redirect an `S3` object to another object or URL by setting redirect location in the metadata of the object
  - No need to use `Route53` to route `S3` objects

## When successful, `Lambda` authorizer should return:
  - Primary identity
  - Policy document with desired permissions

## `Context objects` in `Lambda` functions provide the info about the functions including name, version and unique identifier

## `Kinesis Data Stream` with `Lambda` when fails?
  - When an event fails Lambda will retry the entire batch of a shard over and over again unti it succeeds or the event expires in the source mapping
  - This may lead to duplicates in transactions

## What are `service-linked roles`?
  - They are special roles that link to an AWS service and contain all permissions required to call this service
  - Sometimes we don't need to create a `service-linked role`, the service can create that role for us automatically

## Grant access for `CloudFormation` to access `Lambda` code in `S3` in the most secure way?
  - Grant permission List and GetPermission for `CloudFormation` execution role
  - Add a bucket policy to `S3` to allow the principal 
  - Do not grant access for principal *
  - Use a service-linked role is not a good idea either because it contains too many permissions

## CloudFront enforce HTTPS?
  - Can set `View Protocol Policy` to `Redirect HTTP to HTTPS` and need to change `Origin Protocol Policy` to `Match Viewer`
  - Or in `View Protocol Policy` can select `HTTPS Only`
  - Install SSL cert in ALB
