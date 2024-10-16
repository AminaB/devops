# aws
- use static public ip to avoid ip changing when you powert off your instance (elastic ip)
- use cli : create user "awscli" with access key
- install as-cli 
- and run : aws configure

# aws command
- aws ets get-caller-identity
- aws ec2 describe-instances

# EBS
- create ebs volume and attach ec2 instance to it (they must be in the same AZ)
- ssh to ec2 instance : ssh -i key.pem ec2-user@ip 
- sudo -i
- fdisk -l : list all disk
- df -h : to see mount (/dev/xvdal mount to root user)
- you will see one disk (/dev/xvdf) which don't have partition (it is the ebs). 

## create partition
-  fdisk /dev/xvdf
  m
  n
  p
  1
  +gGB ( for last sector)
  p (print partition table)
  w (to write)
## formating 
- mkfs.ext4  /dev/xvdf1

## mount it
- cd /var/www/html
- 
- mount /dev/xvdf1 /var/www/html/images/
- df -h
- umount /var/www/html/images/

## permanent mount
- vi /etc/fstab
  /dev/xvdf1 /var/www/html/images/  ext4 default   0 0
- mount -a 
- mv /tmp/img-backup/* /var/html/images/
- systemctl restart httpd

## EBS Snapshot
- var unmount /var/www/html/images/
- yum install lsof -y
- mount -a 
- systemctl restart httpd
- cd /var/html/images
- lsof /var/html/images
- unmount /var/html/images

- detache ebs volume to ec2
- create volume, attache it to your instance
- log to ec2
- mount the volume permanently to /var/lib/mysql (create the directory before)
- install mariadb : apt install mariadb
- systemctl start mariadb
- go to aws console and create ebs snapshot of this vol

- in ec2 instance : remove everything in /var/lib/mysql
- stop mariadb
- unmount /var/lib/mysql
- detach volume in aws console

- create volume from snapshot
- attach volume to ec2
- mount -a (you see all data back)

# ELB

# cloudWatch
- monitoring service
- ec2 -> cloudWatch -> alarm -> alarm triggered -> emails notification (SNS)
## STRESS Installation ()
### Centos
- sudo yum install epel-release -y
- sudo yum install stress -y

### Amazon Linux 2
- sudo amazon-linux-extras install epel -y
- sudo yum install stress -y

### Ubuntu
- sudo apt update
- sudo apt install stress -y

## stress command : stress the linux os (cpu, ram,...)
- run stress to stress the cpu : stress -c 4 -t 60 && sleep 60 && stress -c 4 -t 60 && sleep 60 && stress -c 4 -t 360 && sleep  && stress -c 4 -t 460 && sleep 30 && stress -c 4 -t 360 && sleep 60
- go to see cpu monitoring in aws console
- you can create alarm on cpu utilisation in cloudWatch

# EFS
- shared storage
- create efs security group : add NFS inbound rules
- create file system
- customize : general purpose -> enable encryption -> choose the created SG  and remove the default one -> and do it in all AZ 
- create access point 
- install amazon-efs-utils
- mount efs (add this command to /etc/fstab) : file-system-id /var/www/html/img ............
- mount -fav
- every file i put in img folder will be in efs file system

# ASG
- alarm, scaling policy, instances 

- add target group, add LB for this target group
- create ASG -> select lunch template for IAM -> select all zones
- attach LB ->select TG ->