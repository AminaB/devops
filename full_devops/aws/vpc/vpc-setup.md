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

# VPC
- public subnet : web servers inside it have public ip, and can go to the internet, and internet can come too, by internet gateway
- private subnet : 
  - app server, db. Only private ips. 
  - if we try to download package from the internet, it will connect to Nat gateway from public subnet, 
  - the traffic from internet cannot go into private subnet
- Route table
  - with route tables ec2 for ex can know if it has to connect to nat gateway or internet gateway
