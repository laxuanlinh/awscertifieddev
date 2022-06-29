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
- Source from CLI, SDK, API Gateway...
- The response is returned right away
- Error handling happens on client side (retry, backoff)

## Integration with ALB
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