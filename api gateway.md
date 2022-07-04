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