# CloudFront

- Content Delivery Network (CDN)
- Improves read performance, content is cached at the edges
- DDoS protection
- Sits in front of S3 buckets to distribute and cache files at the edges
- Enhances security with CloudFront Origin Access Identity(OAI)
- Can be used as an ingress to upload files to S3
- Can be used with a custom origin

## Cloud Front with S3 and Custom Origin
- When a request to S3 is made using CloudFront URL, CF will forward this request to S3 and return the object but also cache it for later use
- When an edge location accesses S3, CF OAI will authenticate whether the edge is allowed to access the resource
- When use EC2 instance as a custom origin, its IP must be public and its security group must allow CF to access
- But if use EC2 instances with an ELB then only the ELB's IP needs to be public and its sec group to allow CF

## CloudFront Geo Restriction
- Can restrict or allow users from countries using Whitelist and Blacklist
- The user is determined by a 3rd party geo IP DB

## CloudFront vs S3 Cross Region Replication
- CloudFront:
  - Global Edge network
  - Files are cached for a TTL
  - Great for static content that must be available everywhere
  - The content might be outdated a little bit
- S3 replication
  - Must manually set up in some regions
  - The content is updated in almost realtime
  - Read only
  - Great for dymanic content that need to be available at low-latency in few regions

## Create CloudFront Distributions
- Go to CloudFront and `Create Distribution`
- Specify the `Origin domain`, could be an S3 bucket or ELB ...
- Select `Yes use OAI` so that only CloudFront can access this bucket (in case the bucket is private)
- Create new `Origin access identity` for CF
- Select `Yes, update the bucket policy` so that AWS will automatically allow CF to read and write to the bucket
- Optionally, can specify the default returned object if users access the root path /
- Click `Create Distribution`

## CloudFront Caching
- Cache based on headers, session cookies, query string params
- Cache lives in edge locations
- Edge locations are places where data is cached to reduce latecy
- You want to maximize cache hit rate to minimize requests on the origin
- Control the TTL (0 sec to 1 year) can be set by the origin using the `Cache-Control` header, `Expire` header...
- You can invalidate part of the cache using the `CreateInvalidation` API
- We should separate the static and dynamic content to maximize the cache hit rate
- When uploading new content but CF still caches the old content, can go to the `Distribution` in CloudFront => `Invalidation` and `Create Invalidation`, specify the object path, could be `/*` to apply for all objects

## CloudFront Security
- CloudFront can restrict or allow users from countries using *Whitelist* or *Blacklist*
- Can enforce HTTP or HTTPS using Viewer Protocol Policy to:
  - Redirect HTTP to HTTPS
  - Or use HTTPS only
- Origin Protocol Policy (HTTP or S3):
  - HTTPS only
  - Or Match Viewer (HTTP=>HTTP, HTTPS=>HTTPS)
  - *Note*: S3 bucket websites don't support HTTPS
- To edit the policies, go to Distribution => `Behaviors` tab, enforce HTTP and HTTPS
- To restrict or allow users from countries, go to `Geographic restrictions` tab

## CloudFront signed URLs
- It's commended to use Trusted Key Group to store public keys
- If you have a private CF distribution and only want to share it with a number of users then can use signed URLs or Cookies with attached policies with:
  - URL expiration
  - IP range to access from
  - Which AWS users to sign
- Signed URLs: access to individual files
- Signed cookies: access to multiple files
- **S3 pre-signed URL** vs **CF signed URL**:
  - **CF signed URL** allows access to a path, no matter the origin (could be S3, EC2, ELB)
  - **CF signed URL** uses account wide key-pair so only the root can manage it
  - Can filter by IP, path, date, epxiration
  - Can leverage caching features
  - **S3 pre-signed** cannot do those things but it can be managed by any authorized users
  - If a user has a **S3 pre-signed** URL, it can access objects with the same privileges as the signer
  - **S3 pre-signed** URLs have limited lifetime

### To create signed URL
- Generate a key-pair using whatever tool but make sure it's 2048 bit size
- Go to `CloudFront` => `Public Keys` => `Create public key`
- Paste public key just created
- Go to `Key Groups` => `Create key group`, select the public key
- Use AWS SDK to generate the signed URLs

## CloudFront Pricing
- The more data used, the less expensive
- To reduce the cost, we can reduce the number of edge locations
- There are 3 classes:
  - Price Class All: all regions - best performance
  - Price Class 200: all regions except expensive ones
  - Price Class 100: only cheap regions

## CloudFront Multiple Origins
- Can specify the origin based on path.
- Example: /api/* then redirect to an ELB, /content/* then redirect to S3 buckets

## CloudFront Origin Groups
- This feature is to increase high-availability and do failover
- Each Origin Group has a primary and a secondary origin.
- If the primary fails then CF will try the secondary
- For EC2, if the primary fails then CF will call the 2nd instance's APIs. 
- For S3, if the primary bucket fails then the replica takes place

## CloudFront Field Level Encryption
- Protect users sensitive information through application stack
- Use asymmetric encryption
- Need to specify set of fields in POST requests that you want to encrypt (up to 10 fields)
- Specify public key
- When a user sends a POST request to Edge location, the data is encrypted with public key, this request is forwarded to CF, ELB and to the server where it's decrypted using the private key that the application has access to.

  
  