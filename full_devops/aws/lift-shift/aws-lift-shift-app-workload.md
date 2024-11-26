

#  AWS LIFT and SHIFT APP
In this setup, I'll manually build and deploy a project (Vprofile projetc BY @hkhcoder ) using the following AWS services: 
EC2 instances, Elastic Load Balancer (ELB), Auto Scaling Group (ASG), ACM, and S3.
***
## The final result
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/webpagelogin_with_domain_name.png)
## previous 
Multi tier web app stack : Vprofile project running in VMs created by vagrant file
### Scenario
- app services running on physical/VM machines
- work load in same machine (many services running in the same DC, here my computer)

### problem of managing DC
- many teams (DC OPS, Monitoring, Sys Admin, etc...)
- complex management services and teams 
- scale complexity
- cap EX and regular maintenance cost
- difficult to automate
- time-consuming

## host & run this project on aws cloud prod (manual build and manual deploy)

### objective
- flexible infra
- no upfront cost
- modernize app effectively
- IAAC : automation
- 
### Aws services
- EC2 -> replace VM (tomcat rabbitMQ, MEMCACHE, MYSQL)
- ELB -> Replace NGINX LB
- ASG : automatically adding severs as we need or get rid of them
- S3/EFS : for shared storage (like images), or file system
- Amazon certificate manager : secure website (https) 
- Route 53 : private DNS service

### aws stack
- user access the site by using url (this url mentioned in goDaddy DSN)
- this endpoint connect to the LB using https, the certificate for https encryption will be in ACM
- the ALB SG allow only https traffic
- the ALB route to Tomcat in set of EC2 managed by ASG (these ec2 instances are in the same SG AND allow traffic on 8080 only from LB)
- information of backend services like mysql,... will be mentioned in Route 53
- MemCached, Rabbit MQ, Mysql will be in the same SG
- Software artifact will be in S3 bucket

## workflow (us-east-1)
- login to aws account (use iam user with admin access not root)
- create Certificate in ACM for my godaddy domain : barryit.xyz
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/certificateACM.png)

### create SGs (for LB, EC2s, and backend services: mysql, memcached, rabbit mq)
- LB SG  : allow http traffic and https from everywhere
- EC2 SG (tomcat) : allow http traffic from LB SG
    - 22 from my Ip Only (you ip might change each day, make sure to update the rule)
    - 8080 from my IP (for troubleshooting if needed)
- Back end SG : allow :
    - 3306 from EC2s: for mysql 
    - 11211 from EC2s : for memcached
    - 5672 from EC2s: for Rabbit MQ
    - all traffic : for the SG itself (internal traffic on all port)
    - 22 from my ip
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/SGs.png)
### Launch instances with user data (bash scripts)
- create Key Pairs to log into EC2 instances using ssh
- name : vprofile-db01 : with mysql.sh as user data. choose the backend security group and amazon linux
- name : vprofile-mc01 : with memcached.sh as user data. choose the backend security group and amazon linux
- name : vprofile-rmq01 : with rabbitmq.sh as user data. choose the backend security group and amazon linux
- name : vprofile-app01 : with tomcat_ubuntu.sh as user data. choose the app security group and ubuntu os
- connect to db01 using ssh (add permission to.pem)
    - ssh -i vprofile-prodd-key.pem ec2-user@98.83.154.170
    - sudo -i
    - systemctl status mariadb
    - mysql -u admin -padmin123 accounts
    - show tables;
- connect to mc01 using ssh
    - ssh -i vprofile-prodd-key.pem ec2-user@34.235.158.175
    - sudo -i
    - systemctl or run :  ss -tunlp | grep 11211 (to see mmcache port)
- connect to rmq01
    - ssh -i vprofile-prodd-key.pem ec2-user@52.91.236.155
    - sudo -i
    - systemctl status rabbitmq-server
- connect to app01
  - ssh -i vprofile-prodd-key.pem ubuntu@54.90.182.183
  - systemctl status tomcat9
  -  ls /var/lib/tomcat9/
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/ec2s.png)

### ip to name mapping in Route 53 (in vagrant we used to use /etc/hosts file to map name to a VM ip, it is local) 
- go to route 53 in aws console
- create "hosted zone"
- name : vprofile.in
- choose private hosted zone(not visible in the internet only in the region vpc)
- region : us-east-1, and choose the vpc (we have only one in the region, all VM in the region will see the hosts)
- click on create hosted zone
- create record -> simple record for db01,mc01, and rmq01 (use the private ips for records)
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/route53HostedZone.png)

### build app from source code (create artifact)
- cd vprofile-project
- mvn install
### push artifact to s3 bucket
- create iam user (s3admin with s3 full access policy)
- create access keys for s3admin user with CLI access
- configure aws cli with the access keys : aws configure
- create s3 with cli : aws s3 mb s3://aminatoucoder-vpro-arts
- copy artifact in s3 :  aws s3 cp target/vprofile-v2.war s3://aminatoucoder-vpro-arts
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/s3.png)
### copy artifact from s3 to tomcat ec2 instance
- create iam role for ec2 instance to Allows EC2 instances to use S3 service (full access) : aminacoder-vprofile-role
- attach aminacoder-vprofile-role to app01 via actions -> security
- ssh to app01
- sudo -i
- apt update
- apt install awscli -y
- test : aws s3 ls
- aws s3 cp s3://aminatoucoder-vpro-arts/vprofile-v2.war /tmp
- systemctl stop tomcat9
- rm -rf /var/lib/tomcat9/webapps/ROOT/
- cp /tmp/vprofile-v2.war /var/lib/tomcat9/webapps/ROOT.war
- systemctl start tomcat9 (ROOT will be extracted)
### setup elb with https
- create target group : name vprofile-app-TG, http : 8080, health check path : /login, override port : 8080, healthy threshold :2 , select instance : app01, click on pending below
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/paas/TG.png)
- create App LB : name vprofile-prod-elb, select all AZ, select  ELB SG, add https listener, and the certificate
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/LB.png)
- map elb to website name in godaddy dns
  - copy dsn name from elb and add record in godaddy -> DNS -> add record (with value : vprofile-prod-elb-1448622209.us-east-1.elb.amazonaws.com)
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/paas/domain.png)
### verify
- got to  https://vprofileapp.barryit.xyz/index
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/webpagelogin_with_domain_name.png)
### build autoscaling group for tomcat instances.
- create AMI from app01 : vprofile-app-image
- create launch template from AMI : vprofile-app-LC, create tag : name =Vprofile-app, resource types : {volumes, instances}. in advanced section choose the iam role (aminacoder-vprofile-role)
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/aws/lift-shift/LT.png)
- create ASG : vprofile-app-ASG, select all zones, add TG, tags,...
- terminate app01
- enable stickiness in TG
- test

### clean up
