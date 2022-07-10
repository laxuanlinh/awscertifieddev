# Other Services

## SES - Simple Email Service
- Can send emails using SMTP interface or SDK
- Integrate with S3, SNS, Lambda
- Sometimes exam tricks into using SES over SQS or SNS, SES can only send emails
  
## Databases Summary
- RDS: relational 
  - MySQL, PostgreSQL
  - Aurora serverless 
- DynamoDB: NoSQL, key/value, serverless
- ElastiCache: in mem DB, Redis/Memcache
- Redshift: Analytic processing

## ACM - AWS Certificate Manager
- Provision, manage, deploy SSL/TSL certs
- Connect to ALB to provide HTTPS endpoints

## Cloud Map
- Managed resource discovery service
- Can register application components with Cloud Map
- Integrate health check
- Can query Cloud Map using SDK, API or DNS

## Fail Injection Simulator
- Managed service for runningg fault injection experiments on AWS workloads
- Stressing application by creating disruptive events (CPU, mem usage), observe how the system responds
- Supports EC2, ECS, EKS, RDS...