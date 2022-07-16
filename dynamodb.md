# DynamoDB

- NoSQL, distributed
- Limited support for joins
- All data is needed for a query is present in 1 row
- Scale horizonally
- Fully managgged highly available with replication arcross AZs
- Fast and consistent performance, millions of requests
- Made of Tables
- Each table has a `Primary Key`
- Each table can have infinite number of rows
- Each item has attributes, max size of an item is 400KB
- Support data types:
  - `ScalarTypes`: String, number, binary, boolean, null
  - `DocumentTypes`: List, Map
  - `SetTypes`: String Set, Number Set, Binary Set
## How to choose Primary Key
- **Option 1**: Partition key (Hash)
  - Unique for each item
  - Must be diverse so that data is distributed
- **Option 2**: Parition key + Sort key (Hash + rangge)
  - The combination must be unique 
  - Data is grouped by partition key
- Example: Building movie database, which one should be primary key?
  - `movie_id` => primary key because of unique and diverse
  - `producer_name`
  - `leader_actor_name`
  - `movie_language`
## Read/Write Capacity Modes
- `Provisioned Mode` (default)
  - Specify number of reads/writes per sec
  - Need to plan beforehand
  - Pay for provisioned capacity
- `On-Demand Mode`
  - Auto scale up and down
  - Pay for what you use 

### Provisioned mode
- Table must have provisioned `read` and `write capacity unit`
- Option to set up auto scaling of throughput to meet demand
- Throughput can be exceeded temporarily using `Burst Capacity`
- If `Burst Capacity` has been consumed, we get `ProvisionedThroughputExceedException`
- It's then advised to do exponential backoff retry
- RCUs and WCUs are spreaded across partitions, `ProvisionedThroughputExceedException` can happen in a hot partition even when RCUs or WCUs are still available

### Write Capacity
- One `Write Capacity Unit (WCU)` repsents one write per second of an item up to `1KB` in size
- If the item is larger than 1KB then more WCU are consumed
- **Example**: write 6 items/sec, each with size 4.5KB, then 6*5=30 WCUs

### Read Capcity 
- One `Read Capacity Unit` = one write per seconds of an item up to `4KB` in size

### Strongly Consistent Read vs Eventually Consistent Read
- When we write to DynamoDB, the data is then replicated in replicas
- `Eventually Consistent Read`: When we read from the replica, if the data has just been writen then it might be inconsistent, costs only half of normal RCU
- `Strongly Consistent Read`: We can set `ConsistentRead` param to `True` in API call (GetItem, BatchGetItem, Query, Scan) to get the latest data
- `Strongly Consistent Read` costs twice the RCUs

### Partition Internal
- Data is stored in partitions
- Partition key go through a hashing algo to know which partition they go to
- WCU and RCU are spreaded evenly across partitions

### Throuttling
- If we exceed provisioned RCU and WCU then we get `ProvisionedThroughputExceededException`
- Reasons:  
  - Hot Keys: 1 partition key is being read too many times
  - Hot Partitions
  - Very large item
- Solution:
  - Exponential backoff
  - Distribute parititon key evenly

### Capacity Mode - On-Demand
- Unlimited WCU, RCU, no throttle, more expensive
- Charged for reads and writes
- 2.5 more expensive
- Use cases: unknown, unpredictable workload
- Can use for development while provisioned with auto scaling for production

### DynamoDB Basic APIs
- `PutItem`: create or replace old item with the same primary key
- `UpdateaItem`: edit attributes of existing items
- `ConditionalWrites`: accept write/update/delete if conditions are met, otherwise return an error, helps with concurrent access to items
- `GetItem`: read based on primary key
- `Query`: still requires primary key return items based on 
  - `KeyConditionExpression`
  - `FilterExpression`
- `Returns`: the number of items specified in Limit
- Can paginate on the result
- Can query table, local secondary index, global secondary index
- `Scan` to scan the entire table, can filter data, consumes a lot of RCU *not efficient* can do parallel scanning to optimize, good to find items without primary key
- `DeleteItem`: delete on condition
- `DeleteTable`: delete the table
- Can batch to reduce API calls
- `BatchWriteItem`: 25 `PutItem` and/or `DeleteItem` in 1 call, can't `UpdateItem`
- `BatchGetItem`: return items from 1 or more tables, parallel to reduce latency

## Indexes
- `Local Secondary Index`: 
  - Primary key must be composite
  - The partition key is the same partition key of the base table, the sort key can be any scalar attr
  - Can query only 1 parition defined by the parition key in the query
  - Supports both eventually and strong consistent query
  - **Can** query attr not in the index
  - Up to 5 LSIs per table
  - Must be defined at table creation
  
- `Global Secondary Index`:
  - Primary key can be simple or composite
  - The partition key and sort key (if present) can be any scalar attr 
  - Can query the entire table across all partitions
  - Only eventually consistent query
  - **Cannot** query attr not in the index
  - Must provision RCU and WCU for the index
  - Can be added/modified after table creation
  
- If the writes are throttled on the GSI then the main table is throttled as well, choose GSI parition key and WCU capacity carefully, can add more RCU and WCU to GSI since it uses them separately
- For LSI there is no special throttling consideration

## PatiQL
- Use SQL like syntax to manipulate tables
- Support some statements:
  - INSERT
  - UPDATE
  - SELECT 
  - DELETE
- Support batch operations

## Optimistic Locking
- Can do conditional writes, ensure data hasn't changed before you update/delete it
- Each item has an attr that acts as version number

## DynamoDB Accelerator (DAX)
- Fully managed highly available seamlessly in-mem cache, micro second latency
- Solve the Hot key problem (too many reads)
- 5 mins TTL by default, up to 10 nodes in the cluster, multi AZ, encrypted

## DynamoDB Streams
- Item level modifications create event stream
- This stream can be:
  - Sent to `Kinesis Data Stream`
  - Invoke and read by `AWS Lambda`
  - Read by `Kinesis Client Lib applications`
- Data retention up to 24 hours
- Use cases: react to changes in real time, analytics...
- When enabling, past events are not recorded
- If Lambda wants to read DynamoDB stream:
  - Create `Event Source Mapping` with source from DynamoDB
  - Provide sufficient permission to function
  - Invoke synchronously (by `EventBridge`)
- Can create a function to read from DynamoDB stream in console but the permission has to be added manually

## Time to live TTL
- Allows to remove items after an expiry timestamp
- Doesn't consume any WCU, no extra cost
- TTL is a number with Unix Epoch timestamp value, when enable TTL, need to specify the TTL attribute
- Expired items deleted within 48 hours after expiration, deleted from both GSI and LSI

## Transactions
- All-or-nothing opertaions to multiple items across one or more tables
- Read Modes - Eventual Consistency, Strong Consistency, Transactional
- Write Modes - Standrd, Transactional
- Consume 2x WCU, RCU
- 2 operations:
  - TransactGetItems
  - TransactPutItems
- Capacity calculation **important**
  - Example 1: 3 transcational writes per sec, item size 5KB = 3 * 5/1 = 15 WCU
  - Example 2: 5 transacitonal reads per sec, item size 5KB = 5 * 8/4 = 20 RCU

## Session State Cache
- Can use to store session state cache for apps
- vs `ElastiCache`:
  - EC fully in mem, Dynamo in serverless
  - Both are key value
- vs `EFS`: must be attached to EC2 instance and cannot attached to Lambda
- vs `EBS` and `instance store`: only be used for local caching and cannot share between instances
- vs `S3`: it's very slow 

## Write Sharding
- If we have a voting app with 2 candidate candidate_A and candidate_B
- If we use candidate_ID as partition key then all data will be inserted into 2 paritions (either candidate_A or B partition)
- To solve hot partition issue, can add surfix or prefix to partition keys
- 2 methods to create:
  - Random suffix
  - Calculated suffix

## Concurrent, conditional writes
- 2 user can override each other write
- To fix this can do conditional write, only update value if the value = something
- `Atomic write` is when 2 users increase a value by 1 and 2 and end up increase by 3
- `Batch write` is to write multi items in a call

## S3
- DynamoDB can only store up to 400KB per item
- Upload to S3, store the object path to DynamoDB

## Other operations
- To clean up a table:
  - Scan + DeleteItem: not efficient
  - Drop Table + recreate
- Copy table
  - Use Data Pipeline
  - Back up and restore to a new table
  - Scan + PutItem or BatchWriteItem

## Security
- VPC endpoints available to access DB without going through internet
- Access controlled by IAM
- Endcyption using KMS and SSL
- Can back up without performance impact
- Multi regions
- Can develop and test apps locally without accessing AWS
- `AWS Database Migration Service` can be used to migrate from MongoDB, Oracle, MySQL, S3...

## Calculation
- RCU:
  - Eventually consistent: 8KB/sec/object
  - Strongly consistent (standard): 4KB/sec/object
  - Transactional: 2KB/sec/object
- WCU
  - Standard write: 1KB/sec/object
  - Transactional write: twice as standard write
