# Advanced S3 & Athena

## S3 MFA Delete
- In order to MFA-Delete a S3 bucket, versioning must be enabled
- MFA is needed to permanently delete an object version or suspend versioning on the bucket
- MFA is NOT needed to enable versioning or list deleted versions
- Only the bucket owner (root account) is able to enable/disable MFA-Delete
- MFA-Delete can only be enabled via CLI

## S3 Default Encryption vs Bucket Policies
- It's a way to enforce bucket encryption
- Bucket policies still take precedence over S3 Default Encryption

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
  1. Cross Region Replication (```CRR```): compliance, lower latency access, replication across accounts
  2. Same Region Replication (```SRR```): log aggregation, live replication between prod and test accounts
- Buckets can be in different accounts
- Copying is asynchronous in the background
- Must give proper IAM permission to S3
- After activating, only new objects are replicated
- If want to replicate the existing objects then use ```S3 Batch Replication```, this is also helpful to replica failed replications
- Can replicate detele markers optionally but cannot delete with a version ID
- If bucket 1 is replicated into bucket 2, bucket 2 is replicated into bucket 3 then objects created in bucket 1 is not replicated into bucket 3

### To create a replica for a bucket in S3:
- Create 2 buckets, origin and replica, enable `Bucket Versioning`
- In the origin bucket, go to Management to create new replication rule
- Specify the destination bucket, cross account can be enabled
- For IAM role, can either select an existing role or select Create new role and leave it to AWS
- Delete marker replication if selected will replicate the marker every time an object is deleted in the origin bucket
- Upon creating a new replication rule, AWS will ask whether to replicate the existing objects with Batch operation

## S3 pre-signed URLs
- Can generate pre-signed URLs using SDK or CLI
- By default valid for 3600 seconds, can change timeout with --expires-in argument
- Users given a pre-signed URL inherit the permissions of the person generating the URL for GET/PUT
- To generate a pre-signed from console, go to the file and select `Share with a pre-signed URL` in the `Object Actions`

## S3 Storage Classes
- S3 has many classes and can manually move between objects manually or use S3 Lifecyle configurations
- S3 has high Durability and Availability, if we store 10,000,000 objects in S3, we expect will lose 1 object once every 10.000 years. Or S3 is not available 53 minutes a year.

### S3 Standard - General Purpose
- 99.99% availability
- Used for frequence access data
- Low latency and high throughput 
- Sustain 2 concurrent facility failures
- Good for big data analytics, gaming apps, content distribution

### S3 Infrequent Access classes
- For data that is less frequently accessed but rapid access when needed
- Lower cost than S3 standard, charged over retrieval
- **S3 Standard-Infrequent Access (S3 Standard-IA):**
  - 99.9% Availability
  - Used for disaster recovery, backups
- **S3 One Zone-Infrequent Access (S3 One Zone-IA)**
  - 99.5% Availability, not really available and resilient
  - High durability
  - Storing secondary backup copies of on-premise data or data you can be easily recreated

### S3 Glacier
- Low-cost object storage meant for archiving/backup
- Charge for storage + object retrieval
- **S3 Glacier Instant Retrieval:**
  - Milisecond retrieval, great for data accessed once a quarter
  - Minimum storage duration of 90 days
- **S3 Glacier Flexible Retrieval:**
  - Has 3 classes with different retrieval time: Expendited(1-5 mins), Standard (3 to 5 hours), Bulk (5 to 12 hours, free)
  - Minimum storage duration of 90 days
- **S3 Glacier Deep Archive**
  - Cheaper and has 2 options Standard (12 hours) and Bulk (48 hours)

### S3 Intelligent-Tiering
- Small monthly monitoring fee
- Move objects automatically between tiers based on usage
- No retrieval charge
- Tiers include:
  1.  Freequent access tier
  2.  Infrequent access tier
  3.  Archive instant access tier
  4.  Archive access tier
  5.  Deep archive access tier

### Configure storage classes
- To configure storage classes, go to object properties and Edit Storage Class
- To configure storage rule to move objects to different classes, go to bucket's Management, create Life Cycle Rule

## S3 Lifecycle Rules
- Scenario 1: EC2 creates thumbnails after profile images are uploaded to S3, the thumbnails can be easily recreated and only need to be kept for 45 days. The source images should be able to be immediately retrieved for 45 days and afterward, the users can wait up to 6 hours.
- `Design`: 
  - The source images should be stored in S3 standard for the first 45 days, after that they will be moved to Glacier Flexible.
  - The thumbnail can be stored in One Zone IA

- Scenario 2: A rule in your company states that should be able to recover your deleted S3 objects immediately for 15 days, although this may happen rarely. After this time and for up to 365 days, deleted objects shold be recoverable within 48 hours.
- `Design`:
  - Enable versioning so that deleted objects can be retrieved
  - For the first 15 days, the non-current versions should be stored in S3 Infrequent Access
  - After that, up to 365 days, the non-current versions can be moved to Glacier Deep Archive

## S3 Baseline Performance
- S3 automatically scales to high request rates with latency 100-200ms
- The app can achieve at least 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD per prefix/folder in a bucket
- If you spread across all 4 prefixes evenly, you can achieve 22000 requests/sec for GET/HEAD

## KMS Limitations
- KMS is used then the requests can be bottlenecked
- When uploading using SSE-KMS, it calls GenerateDataKey from KMS API
- When downloading it calls Decrypt from KMS API
- Based on the region, KMS request rate can vary from 5500 to 30000.
- Can requets a quota increase using Service Quotas Console

## Uploading files
- Multi-part upload is recommended for files larger than 100MB and mandatory for files larger than 5GB. This can be done parallelly 
- To speed up uploads, can use S3 Transfer Acceleration to upload file to an edge location and then it will be forwarded to S3

## Downloading files
- We can request byte-range fetches in HTTP requests to read just a portion of a file to reduce size and time
- This processed can be done parallelly to retrieve the whole file with faster
- Better resilience in case of failure
- Can be used to retrieve only partial of data

## S3 and Glacier Select
- Retrieve less data using SQL queries and performing server side filtering
- Can filter by rows, columns with simple SQL queries

## S3 Event Notifications
- When an event happens in S3 such as access or upload/download objects, an event is sent to Amazon EventBridge and then forwarded to up 18 AWS services to perform actions
- To set up S3 Event Notifications:
  - Go to bucket's Properties tab
  - Go to Event Notifications and Create Event Notifications
  - Specify name, event types, Destination
  - If choose SQS as destination then make sure to create policy in SQS to allow S3 to write to the queue
- For example, when an image is uploaded to S3, it sends an event notification to SQS to create a message in the queue. An EC2 app subscribing to the queue receives this message and generate a thumbnail from the image in S3

## Amazon Athena
- Serverless query service to perform analytics against S3 objects
- Uses standard SQL 
- Support CSV, JSON, ...
- Charges $5 per TB of data scanned
- Used for business intelligence/analytics/reporting analyze and query all kinds of log from different services of AWS
- When a service produces log and send to S3, this log can be analyzed by Anthena
- To use Athena:
  - Go to Settings of Athena to specify a bucket that stores the result
  - Go to Editor and create a new database and tables
  - Queries to create table to analyze `S3 access log` can be found in Google