# vpc setup
Network ACL : control traffic out and in 
if there is a need to control all networking, create your own vpc.
create our logical data center with vpc, our own private network

## address ipv4
### my ip
- my ip :  192.168.0.27/24 (class C)
- mask : 24 bits : 255.255.255.0
- br : 192.168.0.255
- mask : 255.255.255.0 : 3 first octets are for network
- 198.168.0.0 => network
- first usable ip : 198.168.0.1
- last usable ip : 198.168.0.254
- total usable ip : 254

### 172.20.0.0
- 172.20.0.0 /16
- mask : 255.255.0.0
- divide to smaller subnet
  - 172.20.1.0/24
  - 172.20.2.0/24
  - ...
  - 172.20.255.0/24

### CIDR 
- CIDR notation : /24 or /16, ...

## VPC
- public subnet : web servers inside it have public ip, and can go to the internet, and internet can come too, by internet gateway
- private subnet : 
  - app server, db. Only private ips. 
  - if we try to download package from the internet, it will connect to Nat gateway from public subnet, 
  - the traffic from internet cannot go into private subnet
- Route table
  - with route tables ec2 for ex can know if it has to connect to nat gateway or internet gateway
### high availability
at least 2 zones
- all subnets can communicate each other, either they are public or private
- one nat gateway by public subnet (aws will charge you on nat gateway)
- a subnet is assigned to a zone
- 2 public subnet (zone1 and 2), and 2 private subnets
- each subnet has its own route table

## VPC hands-on 
### Setup Details
- Region : us-west-1
- VPC Range 172.20.0.0/16
- 4 subnets : 2 public 2 private
- 2 zones : us-west-1a us-west-1b
- 172.20.1.0/24 => pub-sub1 [us-west-1a]
- 172.20.2.0/24 => pub-sub2 [us-west-1b]
- 172.20.3.0/24 => priv-sub1 [us-west-1a]
- 172.20.4.0/24 => priv-sub2 [us-west-1b]
- 1 internet gateway
- 2 nat gateway
- 2 EIP for each nat gateway
- 2 routes tables : 1 pub, 1 private subnet
- 1 Bastion host in pub subnet : we can access instances ine public subnet
- NACL : firewall for subnet, in the opposite of SG(just allow) we have allow/deny rules
- 1 more VPC => VPC Peering
- never delete default vpc and default igw
### create VPC 
- vpc only
- name : vprofile-vpc
- CIDR : 172.20.0.0/16
### create subnets
- select the right vpc not default one
- create all 4 sub at the same time 

### make public subnet public by adding internet gateway to their route table
#### crete internet gateway 
- name : vprofile-IGW
- create
- action -> attach to vpc 
#### connect IGW to public subnet through route table
- create route table, name : vpro-pub-RT -> create
- edit subnet association 
- select the two public subnets
- edit routes : 0.0.0.0/0, target : internet gateway, select the IGW
#### enable public ip on pub subnets
- go to public subnet
- action -> Edit subnet settings -> check "Enable auto-assign public IPv4 address"
- save

### add NAT gateway to private subnets
- Create EIP to associate to NAT gateway
#### Create NAT gateway
Nat gateway lives in the pub subnet and connect to internet through the internet gateway
- name : vpro-NAT-GW
- subnet : pub-sub1
- select the EIP
- create
#### create route table for the private subnet
- name : vpro-priv-RT
- select vpc
- create
- subnet association
  - select two private subnet
- routes tables
  - Edit routes
  - 0.0.0.0/0, target : Nat Gateway, select the nat gateway

### enable hostname in vpc
it allows ec2 to have hostname when they are lunched inside vpc.
- got to vpc
- edit settings
- check "Enable DNS hostname"

### bastion host
bastion host is a single point entry to access resources inside private subnet (like ssh to private ec2,...)
#### create SG for the bastion host
- name : vpro-bastion-sg
- select the vprofile-vpc
- ssh allows from my ip
#### create key pair
- name : pro-bastion-key
#### lunch EC2
for the bastion host use ami that are very secure
- go to aws marketplace
- search "cis"
- for learning purpose I will use free tier ami
  - ubuntu 22
  - name : bastion
  - t2.micro
  - select key-pair
  - edit network : select the public subnet
  - lunch
  
#### test the pub subnet
- test if we can ssh to ec2 "bastion"

### lunch instance into private subnet and add website on it, create LB in pub subnet to access to the website 
#### ssh to private subnet
for that we need to ssh into bastion and then ssh to the private subnet
- create key pair and copy it inside bastion server
  - name : web-key
  - create
  - copy : scp -i pro-bastion-key.pem web-key.pem ubuntu@54.176.102.58:/home/ubuntu/ 
  - ssh into bastion and verify if the key is there 

- lunch instance into private subnet
  - name : web01-priv
  - select the key : web-key
  - security group : web-01-sg, add ssh from bastion-sg
  - lunch
- ssh to bastion host
  - ssh to private ec2 web01-priv : ssh -i web-key.pem ec2-user@privateip
  - sui -i
  - yum install httpd wget unzip -y
  - wget https://www.tooplate.com/zip-templates/2136_kool_form_pack.zip
  - unzip 2136_kool_form_pack.zip
  - cp -r 2136_kool_form_pack/* /var/www/html/
  -  systemctl restart httpd
#### create LB inside public subnet
- create TG (do not forget to add web-01-priv as pending below)
- create web-elb-sg for LB : allow 80 from everywhere
- create lb
  - application
  - name : vpro-web-elb
  - select vpc
  - select pub subnet for two zones
  - select tg
- edit inbound rule in web01-sg
  - allow http (80) from web-elb-sg
- check if instance is healthy in tg

#### test

### clean up chargeable resources
- delete elb
- terminate instances
- delete nat gateway
- release EIP

### Vpc Peering
to connect VPC each other
- create one more VPC in a different region or account 
  - create it in oregon region
  - name : vpro-db
  - CIDR range (make sure it not overlap other vpc): 172.22.0.0/16
  - a route table wil automatically created for that vpc

#### connect vpro-db(oregon) vpc with vprofile-vpc(california)
- create peering connections (in california region)
  - name : vproNC_peering
  - select local vpc : vprofile-vpc
  - select other vpc in oregon : vpro-db
  - create 
#### accept the peering connection
- go to vpc service in oregon region
- we will see a request connection to peering connection
- action -> accept request

#### route table (both ways : two vpcs here)
for now subnets do not know how to route the traffic, to fix that :
- add rules inside routes (ex with : priv-sub1) in california
  - got to : vpro-priv-RT
  - routes -> edit routes
  - destination : 172.22.0.0/16
  - target : select the peering connection
- add rules inside vpc routes table vprodb-rt in oregon
  - routes -> edit routes
  - destination : 172.20.0.0/16
  - target : select the peering connection
#### adding authorization in SG
we need to give ip ranges 
ex :
- go to vpro-bastion-sg in california
- add inbound rules
- add a specific ip address or subnet ip range or 172.22.0.0/16
- here we allow ssh from 172.22.2.0/24

#### clean up,
- delete peering connection in oregon
- delete vpcs (do not delete default vpc) in oregon and california


# EC2 logs
- lunch ec2 with amazon linux2
- and add this template : wget https://www.tooplate.com/zip-templates/2136_kool_form_pack.zip 
- test website on browser
## httpd logs
- cd /var/log/httpd
- cat access_log
### we can archive this log and send it to s3 buckets 
- create bucket
  - name : wave-web-log-3103
- create archive
  - tar czvf wave-web01-httpdlogs19122024.tar.gz *
  -  mkdir /tmp/log-wave
  - mv wave-web01-httpdlogs19122024.tar.gz /tmp/log-wave/
  - empty the log file : cat /dev/null > access_log error_log
- install aws cli : yum install awscli
- aws s3 help : to see command option
- create iam user with right to access and modify s3 bucket (you can also do it by iam role)
  - name : s3admin with s3 full access policy
- aws s3 cp /tmp/log-wave/wave-web01-httpdlogs19122024.tar.gz s3://wave-web-log-3103
or 
- aws s3 sync /tmp/log-wave/wave-web01-httpdlogs19122024.tar.gz s3://wave-web-log-3103
- rm -rf /tmp/log-wave/
- it is better to do these tasks with script( bash, python, ansible,...) ad scheduled it in a cron job

# logs dashboard : cloudwatch logs
use cloudwatch to stream the log, analyse, create alarms, send its to elastic search,...
## stream acces_log file to cloudwatch
- create iam role log-admin-role : CloudWatchLogsFullAccess and AmazonS3FullAccess
- remove s3admin credential in ec2 : rm -rf .aws/credentials
- got to my ec2 -> action ->security -> attach role
- test : aws s3 ls
- install cloudwatch agent
  - yum install awslogs -y
  - vim /etc/awslogs/awslogs.conf
  - systemctl restart  awslogsd
  - systemctl enable awslogsd
  - we will see logs stream in aws console
- send file to cloud watch 
  - vi /var/awslog/etc/awslogs.conf
- you can send it to s3 bucket
  - in the aws console cloudwatch directly -> action -> export to s3
- you can create our own metric and alarm
  - Create filter pattern like hacker ip, ... 

## enable logs for classic LB in S3 buckets
- we cannot attach role to LB, so you need to attach policy to s3 bucket to giv LB the authorization to put logs
<pub>
      
      {
          "Version": "2012-10-17",
          "Statement": [
          {
          "Effect": "Allow",
          "Principal": {
          "AWS": "arn:aws:iam::elb-account-id:root"
          },
          "Action": "s3:PutObject",
          "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/your-aws-account-id/*"
          },
          {
          "Effect": "Allow",
          "Principal": {
          "Service": "delivery.logs.amazonaws.com"
          },
          "Action": "s3:PutObject",
          "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/your-aws-account-id/*",
          "Condition": {
          "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
          }
          }
          },
          {
          "Effect": "Allow",
          "Principal": {
          "Service": "delivery.logs.amazonaws.com"
          },
          "Action": "s3:GetBucketAcl",
          "Resource": "arn:aws:s3:::bucket-name"
          }
          ]
      }