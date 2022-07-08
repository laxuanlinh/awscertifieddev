# AWS Security

## 1. Encryption
- Encryption in flight protects from man in the middle attack
- Encryption at rest is when data is encrypted after the server receives the data, the key must be managed by KMS
- From client, data is encrypted in flight using SSL, server decrypts it then encrypt again using KMS. 
- When sends it back to client, server decrypts using KMS key, encrypts again using SSL and sends to client, client decrypt using SSL
- Client side encryption is when client manages the key and encrypts data and the server cannot decrypt it

### KMS (Key Management Service)
- "encryption for any AWS service" => most likely the answer is KMS
- Fully integrated with IAM for authorization
- Seamlessly integrated into:
  - EBS: encrypt volume
  - S3: server side encryption of objects
  - Redshift: encryption of data
  - RDS: encryption of data
  - SSM: Parameter store
  - ...
- Can use CLI, SDK

#### Key types
- `Symmetric`: single key for both encryption and decrytion, must call API to use, used for AWS services integrated with KMS
- `Asymetric`: public and private key pair, public can is downloadable, can't access private key, used for encryption outside of AWS by users who cannot call KMS API

#### Features
- Can create, rotate, disable, audit key 
- Prices:
  - AWS Managed Service Default CMK: free
  - User keys created in KMS: $1/month
  - User keys imported: $1/month
- The encryption key can never be accessed and rotated for extra security
- Can only encrypt 4KB per call, if more then need to use envelop encrytion
- Can only access KMS if `Key Policy` (resource based) and `IAM Role` allow

#### Copy encrypted snapshots to another region
- KMS keys can be single or multi regions
- Create a snapshot of the volume (already encrypted by the key)
- Copy the snapshto to another region
- Specify a new KMS key to encrypt this volume
- When create new volume from snapshot, use the new key

#### Key Policies (Resource based)
- Similar to S3 bucket policies
- Difference: cannot copntrol access without them, MUST specify policies
- Default KMS Key Policy:
  - Auto created if don't provide policies
  - Complete access to KMS for the root user, meaning the root user can then delegate the permission to other users in the account
  - To give access to KMS using the default Key Policy, just need to create IAM role to get access to KMS
- Custom Key Policy
  - Define user, roles that can access KMS
  - Define who can admin the keys
  - Useful for cross account access to KMS

#### Cross-account Access
- When copying encrypted volume to another account
- Create a custom Key Policy to allow other account to have access to the key
- Share the snapshot
- The other account will use the KMS key to decrypt and restore the volume
- 