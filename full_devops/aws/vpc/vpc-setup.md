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