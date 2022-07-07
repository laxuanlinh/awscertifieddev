# Advanced IAM

## AWS STS - Security Token Service
- Grants limited and temporary access to AWS resources (up to 1 hour)
- `AssumeRole`: to assume role within account or cross account
- `AssumeRoleWithSAML`: retur credentials for users logged with SAML
- `AssumeRoleWithWebIdentity`: return credentials for users using third parties like FB, Google... AWS recommends against this, should use Cognito Identity Pools instead
- `GetSessionToken`: for MFA
- `GetFederationToken`: for federated users
- `GetCallerIdentity`: return details about IAM user or role in the API call
- `DecodeAuthorizationMessage`: decode error when API call is denied 

### Use STS to Asssume Role
- Create a role and define who can use this role
- Use STS to retrieve a credentials to impersonate the role
- The temp creds are valid between 15 mins to 1 hour

### STS with MFA
- Use `GetSessionToken` to get MFA
- Use `aws:MultiFactorAuthpresent:true` to set condition that the princials must use MFA 

## Advanced IAM
- If there is an explicit DENY, end decision and DENY
- If there is an ALLOW, end decision with ALLOW
- Else DENY
- Example: 
  - Decision starts at DENY
  - Eval all policies
  - Explicit Deny ? Final=DENY : ALLOW ? Final=ALLOW : DENY

### S3 
- IAM policies attached to users/roles/groups
- S3 policies attached to buckets
- We eval the union of both IAM policies + S3 policies = total policies
- IAM ALLOW + S3 DENY => DENY
- IAM ALLOW => ALLOW
- IAM DENY + S3 ALLOW => DENY

### Dynamic Policies
- Assgin each user to read/write to just 1 folder in S3 bucket
- **Option 1**: create IAM policy for each user to each folder (not efficient)
- **Option 2**: Dynamic policies assigned to a group of users
  ```JSON
    "Resource": ["arn:aws:s3:::my-company/home/${aws:username}/*"]
  ```

### Inline and Managed Policies
- `Managed Policy`: 
  - Maintained by AWS
  - Cannot change
- `Customer Managed Policy`:
  - Best practice to use because they're customizable
  - Version controlled + rollback
- `Inline Policy`:
  - Strict one to one relationship between policy and principal
  - Deleted if the principal is deleted, cannot version control or rollback

### Grant a User permission to pass a role to a service
- To config AWS services, must pass IAM role to these services, later the services will assume the role
- To do pass roles, must have permission `iam:PassRole`
- This role usually comes with `iam:GetRole` to view the role being passed


## AWS Directory Services
- MS Active Directory on Windows Server, database of objects
- Centralize security management, create accounts, permissions
- Objects are organized in trees
- A group of trees is a forest
- `AWS Managed MS AD`
  - AWS Directory Services create AD and manage users
  - Establish trust connections with on-premise AD
  - MFA supported
- `AD Connector`
  - Acts as proxy, on-prem AD still manages users instead of both AWS and on-prem managing
- `Simple AD`
  - AD compatible managed by AWS
  - Cannot be joined with on-prem AD