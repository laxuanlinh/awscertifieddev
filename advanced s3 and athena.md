# Advanced S3 & Athena

## S3 MFA Delete
- In order to MFA-Delete a S3 bucket, versioning must be enabled
- MFA is needed to permanently delete an object version or suspend versioning on the bucket
- MFA is NOT needed to enable versioning or list deleted versions
- Only the bucket owner (root account) is able to enable/disable MFA-Delete
- MFA-Delete can only be enabled via CLI

## S3 Default Encryption vs Bucket Policies
- It's a way to enforce bucket encyrption
- Bucket policies still take predence over S3 Default Encryption

## S3 Access Log
- When enabled, Access Logs will record all requests to access the bucket
- This log is stored in another logging bucket
- Later this data is used by tools like Amazon Athena to analyze
- ***Warning***: *DO NOT SET THE LOGGING BUCKET TO BE THE MONITORED BUCKET, IT WILL CREATE AN INFINITE LOOP AND THE SIZE WILL GROW EXPONENTIALLY*
- To create a logging bucket, create a regular bucket
- Go to the monitored bucket's Properties tab and enable Server Access Logging
- Specify the logging bucket, AWS will automatically update the bucket policies of the logging bucket
- The logging bucket will now have the information about access requests to the monitored bucket

## S3 Replication
- Versioning must be enabled in both source and destination bucket
- There are 2 types of replication:
  1. Cross Region Replication (```CRR```): compliance, lower latecy access, replication across accounts
  2. Same Region Replication (```SRR```): log aggregation, live replication between prod and test accounts
- Buckets can be in different accounts
- Copying is asynchronous in the background
- Must give proper IAM permission to S3
- After activating, only new objects are replicated
- If want to replicate the existing objects then use ```S3 Batch Replication```, this is also helpful to replica failed replications
- Can replicate detele markers optionally but cannot delete with a version ID
- If bucket 1 is replicated into bucket 2, bucket 2 is replicated into bucket 3 then objects created in bucket 1 is not replicated into bucket 3

