# Step Functions and AppSync

## Step Functions
- Model your workflows as state machines (1 per workflow)
- Define workflow in JSON
- Visualize the workflow
- Every step could be Lambda function, DB insert ...
- To start a workflow, can use SDK, `API Gateway`, `EventBridge`

### Task States
- Each task state can invoke a service like Lambda, ECS, SQS ... or launch the next Step Function workflow
- A task can also be defined in an Activity
- An activity can poll step functions for work then sends result back to step functions

### Error Handling
- If an error happens we should not catch it inside the code to make the code simpler
- Should use `Retry` and `Catch` in the State Machine to handle instead
- Predefined error codes:
  - `State.ALL`: matches any errors
  - `State.Timeout`: task runs longer than `TimeoutSeconds`
  - `State.TaskFailed`: execution failure
  - `State.Permissions`: insufficient privileges
- `Retry`: defines how and number of times to retry
- `Catch`: handles errors after retries
  - `ErrorEquals`: match a specific type of error
  - `Next`: State to send to
  - `ResultPath`: specify what input is sent to the state of the `Next`
  - For example: `"ResultPath": "$.error"` will include the error in the input of the next step

### Standard vs Express
- `Standard`: 
  - Max Duration: 1 year
  - Execution start rate: 2000/sec
  - State transition rate: 4000/sec/account
  - Charge per state transition, expensive
- `Express`
  - Max Duration: 5 mins
  - Execution start rate: 100000/sec
  - State transition rate: unlimited
  - Charge per number of executions, cheaper

## AppSync
- Managed service that uses **GraphQL**
- If asked about GraphQL, it's `AppSync`
- Easy to query exactly data needed
- Can combine data from 1 or more sources (RDS, APIs, Dynamo, Aurora...)
- Retrieve data in real-tome with WebSocket or MQTT on WebSocket
- For mobile apps: local data access & data sync
- Can start by uploading 1 **GraphQL** schema

### Security
- `API_KEY`: generate token for apps
- `AWS_IAM`: allow users/roles to acccess
- `OPENID_CONNECT`: use open ID provider
- `AMAZON_COGNITO_USER_POOLS`: use existing `Cognito` user database
- For custom domain & HTTPS, recommend to use `CloudFront` in front of `AppSync`

## AWS Amplify
- Create mobile and web applications, like Beanstalk but for mobile
- Amplify Studio: visually build a full stack app, front and backend
- Amplify CLI: config Amplify backend with guided CLI workflow
- Amplify Libraries: connect apps to AWS services (Cognito, S3 ...)
- Amplify Hosting: host web apps
- Authentication out of the box 
  ```
  amplify add auth
  ```
- Auth uses `Cognito` 
- Datastore uses `AppSync` and `DynamoDB` 
- Build and host
  ```
  amplify add hosting
  ```
  