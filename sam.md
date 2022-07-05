# SAM

- Serverless Application Model
- All config in YAML
- Compatible with `CloudFormation`
- An easy way to deploy serverless applications like Lambda, API Gatway, S3, DynamoDB...
- CloudFormation can also deploy these but not as easy
- 2 commands to deploy to AWS
- Recipe: `Transform Header` indicates it's SAM template
  - Transform: 'AWS::Serverless-2016-10-31'
- Write Code
  - AWS::Serverless::Function
  - AWS::Serverless::Api
  - AWS::Serverless::SimpleTable
- Package & Deploy
  - aws cloudformation package / sam package
  - aws cloudformation deploy / sam deploy
- SAM Deployment:
  - Build the application locally and send to `CloudFormation`: sam build
  - Package the application and send to S3: sam package
  - Deploy the application: sam deploy
- To run and debug Lambda functions locally, use SAM CLI

## Create SAM project
- Run command in local
  ```
    sam init
  ```
- Create `template.yaml` file at the root of the project
- Each SAM template must have
  ```
    Transform: 'AWS::Serverless-2016-10-31'
  ```
- Specify the `Resources` such as Lambda functions
- Run command `sam build` to build the file
- Run command `sam deploy` to deploy to `CloudFormation`

## API Gateway
- To create REST API, in the `Properties`, specify `Events`
  ```yaml
    Resources:
        helloworldpython:
        Type: 'AWS::Serverless::Function'
        Properties:
            Handler: app.lambda_handler
            Runtime: python3.6
            CodeUri: src/
            Events:
                Api:
                    Type: Api
                    Properties:
                        Path: /MyResource
                        Method: GET
  ```
## DynamoDB
- To create a DynamoDB table, add Table as a resource
  ```yaml
        Table:
            Type: AWS::Serverless::SimpleTable
            Properties:
                PrimaryKey:
                    Name: userID
                    Type: String
                ProvisionedThroughput:
                ...
  ```
- Add Environemnt variables and IAM policies to the Lambda function resource
  ```yaml
            Environemnt:
                Variables:
                    TABLE_NAME: !Ref Table
                    REGION_NAME: !Ref AWS::Region
            Policies:
                - DynamoDBCrudPolicy:
                    TableName: !Ref Table
  ```
## SAM Policy Templates:
- List of templates to appoly permissions to lambda functions
- Examples:
  - `S3ReadPolicy`: give read only permissions to objects in S3
  - `SQSPollerPolicy`: allow to poll SQS
  - `DynamoDBCrudPolicy`: CRUD permissions

## CodeDeploy
- SAM natively uses CodeDeploy to update Lambda functions
- When we use `sam build` and `sam deploy`, SAM automatically uses `CodeDeploy` to update the function, the new version is automatically published
- Traffic shifting, pre and post traffic hooks to validate deployment
- Rollback using `CloudWatch Alarms`
- To use Canary dpeloyment, add `DeploymentPerference` to the `Properties`
  ```yaml
            DeploymentPreference:
                Type: Canary10Percent10Minutes
  ```