# Cognito

- Give users an identity to interact with apps
- `Cognito User Pools`: sign in functionality, integrate with API Gateway
- `Cognito Indentity Pools (Federated Identity)`: 
  - Provide AWS credentials to users so they can access AWS directly
  - Integrate with `User Pools`
- `Cognito Sync`:
  - Sync data from devices to `Cognito`
  - Deprecated and replaced by `AppSync`
- `Cognito` vs `IAM`: "**hundreds of users**", "**mobile users**", "**authenticate with SAML**" => use `Cognito` 

## Cognito User Pools (CUP)
- Create serverless database of users for apps
- Simple login using username and password
- Password reset
- Email, Phone verification
- MFA
- Federated identity: use Google, Facebook, SAML...
- Login sends back JWT
- Lambda Triggers: can invoke Lambda functions on authentication events (pre, post login, signup ...)
- Hosted Authentication UI: provide a page for login/signup, can customize logo, CSS
- Can authenticate requests to `API Gateway` and `ALB` but not `Classic Load Balancer`

## Cognito Identity Pools (Federated Identity)
- External users want to access AWS resources
- Temporary credentials so cannot use IAM because there are a lot of them
- Users could be:
  - Public provider Google, Facebook, SAML, OpenID ...
  - Users in a User Pool
  - Developer Authenticated Identities
  - Could allow unauthenticated (guest) access
- Users can access directly or through API Gateway
- IAM policies applied to credentials defined in Cognito
- Users' permissions can be customized based on `user_id`
- To access AWS with Identity Pools:
  - *Step 1*: Users get access token from `User Pool` or public providers like FB, Google
  - *Step 2*: Users send the token to `Identity Pool` in exchange for IAM credentials
  - *Step 3*: `Identity Pool` validates with `User Pool` or public provider
  - *Step 4*: Get temp credential from `sts`
  - *Step 5*: Users access AWS using this credential
- Define default IAM roles for authenticated and guest users
- Define rules to choose roles based on user ID
- Can partition users' access using policy variable
- The role must have a trust policy of `Cognito Identity Pools`

## Cognito User Pools vs Identity Pools
- `User Pools`:  
  - Have a user database
  - Allow federate users
  - Can customize hosted UI for authentication
  - Has triggers for Lambda
- `Identity Pools`
  - Obtain AWS credentials for users
  - Users can login through public provider (federate users) and Cognito User Pools
  - Users can be unauthenticated
  - Users are mapped to IAM roles & policies
- CUP + CIP = manage user / password + access to AWS services

## Cognito Sync
- Replaced by `AppSync`
- Store preferences, config, state of app
- Cross device sync
- `Cognito Stream`: stream data from Cognito to Kinesis
- `Cognito Events`: execute Lambda functions in response to events