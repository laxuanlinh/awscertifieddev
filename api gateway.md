# API Gateway

- Provide APIs for Lambda
- Support WebSocket
- Can handle different environment, security
- Handle request throttling
- Swagger/Open API import defined APIs
- Transform and validate requests and responses
- Cache API responses

## Integrations at high level
- Invoke Lambda functions
- Expose HTTP endpoints not only for Lambda but to ALB, on premises, add cache, validation, authentication, API keys ...
- Expose any AWS API

## Deploy API Gateway
- `Edge-optimized` (default) requests routed through CloudFront Edge, API Gateway still lives in 1 region
- `Regional`: for clients within the same region
- `Private`: can only be accessed from VPC using VPC endpoint, use resource-based policy

## Create API 
- Go to `API` => `Create API` => `Build REST API`
- Specify name of resource
- Add method, specify endpoint (HTTP, Lamdba ARN ...)
- Endpoint types: Regional, Edge, Private
- Add paths and methods
- After testing the API, select the API and deploy to a stage (dev, prod, test ...)
- If want to deploy to another stage, can do again and select another stage
  
## Stages
- Can config stages to point to different lambda function alias
- Ex: `api.example.com/v2` => Lambda function `v2`, `/dev` =>` $LATEST` ...
- Stage variables are like environment variables, can be passed to Lambda
- Since Lambda aliases can have weights for different versions, stages can point to these versions of Lambda functions with weights

### Create stage for each alias
- Add method => Add Lambda ARN
- The ARN should include a suffix `:${<name of resource>.<name of alias variable>}`
  - Ex: `:${stagevariables.lambdaAlias}`
- In Lambda, create aliases for different environemnts
- In each alias, go to `Permissions` and add permission `InvokeFunction` for API Gateway resource's ARN
- To test, go to the resource, specify the `lambdaAlias` variable (`dev`, `test`, `prod`), the request from API Gateway should redirects to the alias accordingly
  - Ex: `https://lambda.us-east-1.amazonaws.com/2015-03-31/functions/arn:aws:lambda:us-east-1:659287870720:function:lambda-api-gateway:PROD/invocations` => PROD alias/evironment
- Deploy the method to stage, go to Stage Variable and specify the env variable
  - Ex: lambdaAlias = DEV

### Stage configuration
- For each stage can config cache, throttling, firewall, tracing, variables ...
- Can enable `Canary Deployment`

## Canary Deployment
- Enable canary dpeloyment for any stage
- Choose a small % of traffic to test first and then move 100% to the new version
- Metrics and Logs are separated
- Can override stage variables for canary
- When deploy the resource, select Canary stage

## Integration Types
- `Mock`: for mocking
- `HTTP/AWS` (Lambda and other services)
  - Must config both integration request and integration response
  - Set up data mapping using mapping templates
  - Mapping template can transform JSON to XML with SOAP
- `AWS_PROXY` (Lambda Proxy) the request is the input to Lambda, cannot change the request, no headers, query string ... passed as arguments
- `HTTP_PROXY`: pass directly to the backend and client

## Swagger / Open API spec
- Common way of defining REST API using API definitions as code
- Swagger can be written in YAML or JSON

## API caching
- Default TTL 300 secs, max 3600 secs
- Caches are defined per stage, possible to override per method
- Expensive
- Can invalidate by flushing the entire cache
- Client can invalidate cache with header: `Cache-Control:max-age=0` with proper IAM permission
- If not impose policy then anyone can invalidate cache, not good

## Usage plan
- Can create API keys to offer to customer 
- Usage plan:
  - Who can access
  - How fast, how much
  - Use API keys to identify clients and meter access
  - Config throttling limits and quota limits
- API keys:
  - String value to distribute to customer
  - Throttling limits are applied to API keys
  - Quota limits is the overall number of max requests
- To create a usage plan:
  - Create 1 or more APIs with required API keys, deploy to stages
  - Generate and distribute to customer (app developer)
  - Create a usage plan with desired throttle and quota limits
  - Associate API stages and API keys with the plan

## Logging and Tracing
- Can enable `CloudWatch Logs` at stage level
- `X-Ray` can be enabled 
- `CloudMetrics` are at stage level
- `CacheHitCount` and `CacheMissCount`: efficiency of the cache
- `Count`: The total number of API requests in a period
- `IntegrationLatency`: time between passing request to backend and when getting the response
- `Latency`: time between receving request from a client and returning the response to the client
- 4XXError and 5XXError howm many errors at client and server side

## Throttling
- By default **10,000 r/s** per account across all API
- Soft limit can be increased upon request
- Error `429 Too Many Requests` (retriable error)
- Set Stage limit and Method limits to make sure they don't consume all throughput if they're under attack
- Or can define Usage Palns to throttle per customer
  
## Errors
- 502 Bad Gateway Exception usually incompatible outputs returned from Lambda or out-of-order invocations due to heavy loads
- 503 Service Unavailable
- 504 Integration Failure backend timeout, API Gateway timeout after 29 seconds

## CORS
- Can enable CORS at resource level to allow requests from other origins

## Authentication and Authorization
- `IAM Permission`: 
  - If it's accessed by othe AWS services, users
  - Resource Policies: who can access API Gateway, cross account access, specific IP, VPC endpoints
  - Signature v4
- `Cognito User Pools`: 
  - Manage user lifecycle, token expires auto
  - API Gateway verify user via Cognito
  - No custom implementation required
  - Authentication = Cognito User Pools, Authorization = API Gateway Methods
- `Lambda Authorizer`: 
  - Bearer token
  - Lambda evals the token and return an IAM Policy for the user, result is cached
  - Authentication = 3rd party, Authorization = Lambda function

## HTTP API and REST API
- HTTP APIs: low latency, proxy, no mapping, OIDC and OAuth2, no usage plan
- REST API: same but no OAuth, OpenID Connect

## WebSocket
- Client connects to Gateway with a `connectionId`
- This `connectionId` is sent to Lambda, Lambda can store it in DynamoDB
- Later Lambda can use this `connectionId` to get user info as long as client still connects to Gateway
- Client sends frames to Gateway on the same connection and receives response
- The incoming data is routed to different backend based on `route select expression` `$request.body.action`