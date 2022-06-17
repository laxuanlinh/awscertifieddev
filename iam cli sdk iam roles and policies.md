# IAM CLI, SDK, IAM Roles and Policies

## Create Policies in IAM
- To create a policy in IAM, go to ```Policies``` => ```Create Policies```
- Choose a service
- Check the ```Actions``` boxes
- In ```Resources```, select the resource ***ARN*** or can select ***Any*** to select all Resources in the service
- Can create additional conditions
- Finally input policy's name and click ```Create Policy```.
- It's also possible to use ```Policy Generator```

## AWS CLI Dry Runs
To test a whether user can execute a command in aws cli, add --dry-run to the command to test it.

## STS Decode Errors
- When execute a command on aws cli and fail, we usually get a long error message.
- In order to decode this error message, we use ```sts``` command.
    ```
    aws stst decode-authorization-message --encode-message <the long error message>
    ```
- To decode a error message, the user needs to be authorized with ```sts:DecodeAuthorizationMessage``` permission

## AWS EC2 Instance Metadata
- EC2 Instance Metadata is powerful but least known
- It helps the instance to know about itself without using an IAM Role for that purpose
- The URL is http://169.254.169.254/latest/meta-data
- This URL is only accessible from the EC2 instance
- It returns metadata that includes IAM Rolename but cannot return IAM Policy.
- Metadata = Info about the EC2 instance
- Userdata = launch script of the EC2 instance

## MFA with CLI
- To use MFA with the CLI, must create a temporary session
- To do so, must run the ***STS GetSesssionToken*** API call
    ```
    sts get-seeion-token --serial-number <arn number from IAM> --token-code <code from MFA app> --duration-seconds 3600
    ```
- This will return a session token
- Copy this token to the field ```aws_session_token``` field in  credentials file

## AWS SDK Overview
- It's a SDK allowing you to interact with AWS without using the console or CLI
- If no region is specified, the SDK will choose us-east-1 by default

## AWS Limits (Quotas)
- AWS has limits on how many time we can call the API per sec.
- For example: EC2 allows 100 calls/sec, S3 allows 5500 calls/sec
- For ***Intermittent Error (5xx error)***, implement ```Exponential Backoff``` 
- For ***Consistent Errors***: request an API throttling limit increase
- For Service Quotas, by default we can have up to 1152 vCPU, if we want to incease, we can open a ticket

### Exponential Backoff
- Exponential Backoff should only be implemented when we get ```ThrottlingException```
- If we use AWS SDK then the retry mechanism is already included
- If we call API by ourselves the we need to retry manually 
- We should not implement when we get something other than 5xx error, such as 400 error

## CLI Credentials Provider Chain
- When execute a command in CLI, will look for the credentials in this order
  1. **Command line options**: example --region, --output, --profile
  2. **Environment variables**: example AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY...
  3. **CLI credentials file**
  4. **CLI configuration file**
  5. **Container credentials**: for ECS task
  6. **Instance profile credentials**: for EC2 Instance Profiles

## SDK Default Credentials Provider Chain
- The Java SDK will look for credentials in this order
  1. **Java system properties**: aws.accessKeyId and aws.secretKey
  2. **Environemnt variables**: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
  3. **The default credentails file**
  4. **Amazon ECS container credentials**: for ECS containers
  5. **Instance profile credentials**: for EC2 instances

## Example for default credentials provider chain:
- A Java application using AWS SDK running on an EC2 instance
- The Java app has properties file that store credentials of an IAM users that has full access to S3
- Because the requirement for the app is to access just 1 S3 bucket, therefore a EC2 Instance Profile with a Role that is limited to that one S3 bucket is created and assigned to the EC2 instance
- But the EC2 instance still has full access to the S3 despite of its Instance Profile
- This is because the IAM user stored in the Java system properties takes precedence over the EC2 instance profile

## AWS Credentials Best Practices
- Never store AWS credentials in code, it should be inherited from the credentials chain
- Within AWS, use IAM Roles, example if working with EC2 then use EC2 Instance Roles, for ECS use ECS Roles...

## Signing AWS API requests
- Requests to AWS HTTP API should be signed using AWS credentials
- If you use SDK or SLI, the requests are automatically signed
- Requests should be signed with SigV4
- SigV4 means that the request is signed with AWS credentials