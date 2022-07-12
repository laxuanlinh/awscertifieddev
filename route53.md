**Amazon Route53**

- Highly available, scalable, fully managed and authoritative DNS sevice
- Route 53 is also a Domain Registrar
- Create records and config how they connect to IP addresses
- Supports the following DNS types:
  - A / AAAA / CNAME / NS
  - CAA / DS / MX /NAPTR / PTR / SOA /TXT / SPF / SRV

- A maps to IPv4
- AAAA maps to IPv6
- CNAME maps a hostname to another hostname, the target must be A or AAAA, cannot create CNAME records for the top node of a DNS namespace, exp: can create wwww.example.com but cannot create example.com
- NS - Name Servers for the Hosted Zone

***Hosted Zones***
- A container for records that defines how to route traffic to a domain and its subdomains
- Public Hosted Zones contains records that specify how to route traffic on the internet
- Private Hosted Zones contains records that speficy how to route traffic in VPC

***Time-to-live***
- The time that the records exist in client's cache

***CNAME vs Alias***

AWS resources exposes an AWS hostname
Ex: lbl-12234.us-east-2.elb.amazonaws.com but we want myapp.mydomain.com

CNAME:
- Points a hostname to any other hostname but does not work for Non root domain. Example: Root domain is example.com, it's not possible to create a CNAME from example.com to an EC2 instance, has to be something like dev.example.com.
- But CNAME can map to third party resources

Alias:
- Points a hostname to any other hostname including root domain. For example we have an EC2 instance and a root domain example.com, we can create an alias of example.com to point to this EC2 instance.
- Can only work with AWS resources.
- Free of charge and native health check

***Routing Policies***
It's how Route53 responds to DNS queries.

*Simple*
- Can route to a single record or multiple records
- If multiple values are returned, one is selected randomly by client
- When Alias is enabled, specify only 1 AWS resource
- Can't be associated with Health Check

*Weighted*
- Control the % of the requests that go to each specific resource
- Assign each record a relative weight.
- DNS records must have the same name and type
- Can be associated with Health Check
- Assign a weight of 0 to stop sending requests to a record.
- If all records have weight of 0 then they are returned equally.

*Latency-based*
- Redirect to the resource that has the least latency 
- Latency is based on traffic between users and AWS Regions
- Route 53 will determine which region has the least latency to users and redirect the requests to there.

*Geolocation*
Can config users from a certain countries to be redirected to a region. Example: users from the US are redirected to an EC2 instance in Europe

*Geoproximity*
- Routes requests based on geolocation of users and resources.
- You can set the bias number of shrink or expand the region where the resources are located.
- Can only use when use Traffic Flow

*Multi-value*
- Use when routing to multiple resources
- Route 53 returns multiple values/resources
- Can be associated with Health Checks to only return health resources
- Up to 8 values can be returned for each Multi-value query
- Multi-value is not subtitute for having an ELB

*Health Checks*
- Can only health check public resources
- Health check is automatically integrated with Cloud Watch.
- Health Check EC2 instances using Route 53 will automate DNS Failover.
- Route 53 can health check:
  - Health checks that monitor an endpoint like AWS resources, app, servers
  - Health checks that monitor other health checks, example: can combine the result of multiple health checks
  - Health checks that monitor Cloud Watch alarms, example: to monitor a private resources, we can create a CW alarm so that Health Check can check and failover.

*Failover*
If health check finds that the resource is unhealthy, it automatically switch to another instance

Domain Registrar vs DNS Service
- When buying a domain from a Domain Registrar, we're usually provided with a DNS service to look up the domain.
- It's possible to buy a domain from a third party like GoDaddy and use Amazon Route 53 as a DNS service and via versa.
- To change from third party DNS service to Route 53, go to the nameservers option and replace all nameservers with the DNS servers of Route 53.



