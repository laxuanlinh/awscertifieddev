# Lambda

- Servers are services that are managed
- We provide the code
- Limited by time, short execution
- Run on-demand, only runs when required
- Scaling is automated
- Pay per request and compute time
- Increase RAM also increase CPU and network
- Automatically uses `CloudWatch`
- Supports multiple languages
- Can run Lambda Container Image but the image must implement Lambda API, ECS and Fargate are still recommended to run Docker
## Lambda Integration
- Main
  - `API Gateway`: create rest API and invoke lambda functions
  - `Kinesis`
  - `DynamoDB`: data inserted and invoke lambda
  - `S3`
  - `CloudFront`: lambda at edge
  - `CloudWatch Event` or `EventBridge`: when state of infra changes
  - `CloudWatch Logs`
  - `SNS`
  - `SQS`
  - `Cognito`

## Pricing
- Pay per calls
  - First 1000000 requests are free
  - 0.2 per 1 requests thereafter
- Pay per duration
  - 400000 GB-seconds of compute time per month if free
  - == 400000 seconds if function is 1GB RAM
  - == 3200000 seconds if functions is 128MB RAM
  - After that $1 for 600000 GB-seconds
- It's very cheap

## Synchronous Invocations
- Source from `CLI`, `SDK`, `API Gateway`...
- The response is returned right away
- Error handling happens on client side (retry, backoff)

### Integration with ALB
- To expose Lambda with an HTTPS endpoint, we need API Gateway or ALB
- The Lambda function must be registered in a target group
- The request is transformed into a JSON
- The request should include:
  - ELB info
  - HTTP method, path
  - Query String Parameter as Key/Value pairs
  - Headers as Key/Value pairs
  - Body
- When Lambda returns a response, JSON is transformed into HTTP by ALB
- ALB can support multi header values which means if the client send multiple values with the same name, ALB will convert them into an array
- We need to create an ALB, create a target group and add the Lambda function to the target group, specify the group in the ALB, **DONE!!!**

### Lambda@Edge
- Deploy `Lambda` function along each region edge location around the world
- Can use `Lambda` functions to change CloudFront requests and responses
  - Modify request from user => CF
  - Modify request from CF => Origin
  - Modify response from origin => CF
  - Modify response from CF => user
- Use cases:   
  - Security and privacy
  - Web app at edge
  - SEO
  - Real time image transformation
  - User tracking / analytics, authentication, authorization

## Asynchronous Invocations
- The requests are not procesed right away
- The events can be put in a event queue and `Lambda` eventually processes
- This is used when we don't need the response right away, like analyzing or processing a batch file..
- When `Lambda` fails to process an event, it will retry 3 times
- The event is then put back to the queue, this will cause `Lambda` to process the same event over and over again and fails
- To fix this, can config Lambda to use `Dead Letter Queue` from SNS and SQS
- Source:
  - S3: notification to invoke
  - SNS
  - CloudWatch Event / EventBridge
  - CodeCommit / CodePipeline: code change event ...
  - CloudWatch Logs: log processing
  - Simple Mail Service
  - CloudFormation
  - Config
  - IoT/IoT Events

### CloudWatch Events / EventBridge
- Can schedule an event
- Can set up a rule to detect an event such as CodeCommit's code change, CodePipeline's status change...
- Send the event to Lambda function as destination

### S3 Notifications and Lambda
- Events such as ObjectCreated, ObjecttRemoved, ObjectRestore...
- Object name filtering possible (*.png)
- Typically S3 events are sent in a second but can take longer
- Use cases: generate thumbnails...

### Event Source Mapping
- Sources are:
  - `Kinesis Data Stream`
  - `SQS` & `SQS FIFO`
  - `DynamoDB` Streams
- Records need to be polled from the source
- The Lambda function is then invoked synchronously
- For `Kinesis Data Stream`:
  - An `Event Source Mapping` creates an iterator for each shard, processes items in order
  - Start with new items from the beginning or from the timestamp
  - Processed items aren't removed from the stream so other consumers can still read them
  - Low traffic: use batch window to accumulate records before processing
  - Can process multi batches in parallel, up to 10 batches per shard
  - If an error is returned the whole batch is reprocessed until successful
  - Cam discard old events, restrict number of retries, split the batch on error
  - Discarded events can go to `Destination`
- For `SQS` / `FIFO`
  - `Event Source Mapping` will long poll SQS
  - Specify batch size
  - Set up DLQ on SQS, not on Lambda (DLQ on Lambda is only for async)
  - Or can use Lambda Destination
 
## Event Destinations
- `Asynchonous invocation`: Can send result of both error and sucess to:
  - SQS
  - SNS
  - Lambda 
  - EventBridge
  - It's recommended to use `Destinations` instead of DLQ
- `Event Source Mapping`: For discarded event batches, can send to:
  - SQS
  - SNS

## IAM Roles 
- Roles have to be attached to Lambda functions to grant access to other services
- Whenever we use event source mapping to invoke function, Lambda uses the execution role to read event data
- Best practice is to create 1 Lambda Execution Role per function 
- Use resource-based policies to give othe services and accounts accesst to functions
- An IAM principal can access if:
  - The IAM policy attached to the pricipal allows it (user access)
  - The resource-based policies allow it (service access)

## Environment variables
- Key Pair values in String form
- Adjust the behavior without updating code
- Helpful to store secrets
- Secrets can be encrypted by Lambda service key or Customer Master Key

## Lambda with X-Ray
- `Lambda` is integrated with `CW Logs` and `Metrics`
- To integrate with `X-Ray`, need to enable in `Lambda` config (`Active Tracing`)
- Use `X-Ray` SDK in code
- Ensure function has correct `IAM Execution Role`
- Environment variables to communicate with X-Ray:
  - `_X_AMZN_TRACE_ID`
  - `AWS_XRAY_CONTEXT_MISSING`
  - `AWS_XRAY_DAEMON_ADDRESS`

## Lambda and VPC
- By default `Lambda` functions launch outside of VPC so it cannot access resouces in VPC
- Need to define VPC ID, subnet and Security Group
- `Lambda` will create an ENI in subnets
- Functions need to have role `AWSLambdaVPCAcessExecutionRole`
- Other security groups also need to allow Lambda's sec group access
- But if we put functions in VPC, they will not have public internet access
- We can connect the functions to a `NAT Gateway` / `Instance`

## Performance
- RAM 128MB -> 10GB
- More RAM = more vCPU
- Timeout 3sec -> 900 seconds (15 mins)
- Any task last longer than 15 mins should be carried out by other services like `Fargate`
- `Lambda Execution Context` is a temporary runtime environment that init any external dependencies of lambda code
- It's good for DB connection, https client, sdk client
- The context can maintain for some time so that the next function can reuse it
- `/tmp` temprorary dir to write files to
- **Example**: DB connection should not be init inside handler code, it should be init outside so that the next function can reuse
  ``` python
  import os

  DB_URL = os.getenv("DB_URL")
  db_client = db.connect(DB_URL)

  def get_user_handler(event, context):
    return db_client.get(event["user_id"])
  ```
- Whatever takes too long to init should be left outside of the handler to be resused
- `/tmp` max size 512MB, it remains when the context is frozen so that can be reused, useful for checkpoint your work

## Concurrency
- Up to 1000 concurrent executions if too many functions are spawned
- Can set `reserve concurrency`
- Each invocation over the concurrency limit will trigger a Throttle
  - Sync => return ThrottleError 429
  - Async => retry auto then go to DLQ
- Open support ticket to higher limit
- If we don't set the limit then 1 application can eat up all 1000 executions while other apps will be throttled and cannot scale

## Cold starts and provisioned concurrency
- `Cold start` means every time we start a function, we have to init from the beginning, first request can be slow for the first request
- `Provisioned concurrency` means concurrency is allocated before the function is invoked, the cold start will never happen and latency is low
- `App Auto Scaling` can manage concurrency, schedule or target utilisation

## Lambda Function Dependencies
- If the code depends on external dependencies then zip them with the code 
- If the file is less than 50MB, then upload to Lambda, else upload to S3 and reference to Lambda
- Native libraries need tot be compiled on `Amazon Linux`
- `AWS SDK` comes default with Lambda

## Lambda and CloudFormation
- We can define Lambda functions inline in CF YAML file, this is for simple functions without dependencies, use `Code.ZipFile`
- We can zip and upload to S3 and refer in the YAML file with the following properties:
  - `S3Bucket`
  - `S3Key`: full path to zip
  - `S3ObjectVersion`: if versioned bucket
- If you update the code in S3 but not the version / path in YAML then CF won't update the function
- If versioning in S3 is enabled, then the new version will have the same path and CF will pick up the changes
- If cross account, CF of `account A` needs to access S3 bucket of `account B`, we can create an access policy to allow `account A` or we can update the execution role for S3 of `account B` of `account A`

## Lambda Layers
- Custom Runtimes: C++, Rust
- Externalize dependencies to reuse
- When create a function, go to `Layers` => `Add a Layer` and select the layer template so that we don't have to zip the dependencies with the code

## Lambda Container Images
- Allow Lambda functions as container up to 10GB from `ECR`
- Pack all dependencies in a container
- Can create own images as long as they implement `Lambda Runtime API`
- `Lambda Runtime API` varies across languages

## Lambda Versions
- Each version has its own ARN
- When we change code, we're working on `$LATEST`
- When we publish a version, we create a new incremental version
- Versions are immutable, incremental and can be accessed
- Can create aliases like `dev`, `test` version of a version, they're mutable
- Aliases enable `Blue/Green` deployment by assigning weights to lambda functions
- When assigned weights, an alias can return response from 2 versions according to the weight ratio
- Aliases have thier own ARN, aliases cannot reference other aliases

## Lambda and CodeDeploy
- Can help automate traffic shift for Lambda aliases
- For example: we want to upgrade PROD from V1 to V2, CodeDeploy can increment the weight of V2 over time until it's 100% (`Linear`), deploy both and test V2 in small rate then switch 100% (`Canary`), or it can switch immediately to V2 (`AllAtOnce`) 
- Can create Pre & Post Traffic hooks to check the health of the function to rollback
  
## Lambda Limits (**important**)
- Mem allocation 128MB -> 10GB
- Max execution time 15 mins
- Environment variable 4KB
- Disk capacity in /tmp 512MB
- Concurrent execution 1000 (can be increased)
- Deployment of zip file size < 50MB and when unzipped < 250MB
- Can use /tmp to load other files at startup

## Best practices
- Perform heavy duty work outside of the functions handler to enable Lambda Execution Context
  - Connect to DB
  - Init AWS SDK
  - Pull dependencies or datasets
- Use environment variables for
  - DB string, S3 bucket... don't put these in code
  - Password, sensitive values encrypted using KMS
- Mimize deployment package size 
  - Break down functions
  - Use layers to load dependencies
- Avoid having a function call another function