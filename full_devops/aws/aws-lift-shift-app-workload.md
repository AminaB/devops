# AWS LIFT and SHIFt

## previous 
Multi tier web app stack : Vprofile project using vagrant
### Scenario
- app services running on physical/VM machines
- work load in data center (many services running in the same DC)
- many team : DC OPS, Monitoring, Sys Admin, etc...

### problem 
- complex management services and teams 
- Scale complexity
- Cap EX and regular maintenance cost
- difficult to automate
- time-consuming

## host & run this project on aws cloud prod

### objective
- flexible infra
- No upfront cost
- modernize app effectively
- IAAC : automation
- 
### Aws services
- EC2 -> replace VM (tomcat rabbitMQ, MEMCACHE, MYSQL)
- ELB -> Replace GINX LB
- ASG : automatically adding severs as we need or get rid of them
- S3/EFS : for shared storage (like images), or file system
- Amazon certificate manager : secure website (https) 
- Route 53 : private DNS service

### aws stack
- user access by using url (this url mentioned in goDaddy DSN)
- this endpoint connect to the LB using https, the certificate for https encryption will be in ACM
- the ALB SG allow only https traffic
- the ALB route to Tomcat in set of EC2 manged by ASG (these ec2 instances are in the same SG AND allow traffic on 8080 only from LB)
- information of backend services like mysql,... will be mentioned in Route 53
- MemCache, Rabbit MQ, Mysql will be in the same SG
- Software artifact will be in S3 bucket

### workflow
- login to aws account (use iam user with admin access not root)
- create Key Pairs to log into EC2 instances
- create SGs (for LB, EC2s, and backend services: mysql, memcache, rabbit mq)
- Launch instances with user data (bash scripts)
- update ip to name mapping in Route 53
- build app from source code
- upload s3 bucket
- download artifact to tomcat ec2 instance
- setup elb with https
- map elb to website name in godaddy dns
- verify
- build autoscaling group for tomcat instances.
- 
