
aws cours 

ARN : amazon ressource name 
CLI version : Commande line interface 
aws --version

aws ec2 describe-instances 

region 
aws configure get region

help:
aws help 

create user :
aws --endpoint http://aws:4566 iam create-user --user-name jack

create-group
aws --endpoint http://aws:4566 iam create-group --group-name project-sapphire-developers

add policy to user :
aws --endpoint http://aws:4566 iam attach-user-policy --user-name mary --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

add policy to group:
aws --endpoint http://aws:4566 iam attach-group-policy --group-name project-sapphire-developers --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess




add user to a group :
aws --endpoint http://aws:4566 iam add-user-to-group --user-name jill --group-name project-sapphire-developers

see policies attached to a user:
aws --endpoint http://aws:4566 iam list-attached-user-policies --user-name jack

see policies attached to a group
aws --endpoint http://aws:4566 iam list-attached-group-policies --group-name project-sapphire-developers

--------------------------------------------
provisionning iam ressources with terraform
resource "aws_iam_user" "admin-user"{
	name="lucy"
	tags={
	Description="Lead"
	}
}
NB : berfore to create resource, we have to add provider with credentials arguments to allow terraform to deploy our ressource
provider "aws"{
	region=""
	acces_key=""
	secret_key=""

}

we can also configure a credential file .aws/config/credentials :
[default]
aws_access_key_id=ffsdffdg..
aws_secret_acess_key=jjjjjjj...

or create an enviro variables, and use them by export ...

add policy : 
- first create iam policy 
resource "aws_iam_policy" "adminUser"{
	name="AdminUser"
	policy=?/<<EOF
	{
	"Version": "202....."
	"Statement":[
		{
			"Effect": "Allow"
			"Action" : "*"
			"Ressource": "*"

		}
	]

	}
	EOF
}
we can also move json on file admin-policy.json and do policy=file(admin-policy.json). in this cas we don't need EOF
- attach to user 
ressource "aws_iam_user_policy_attachement" "lucy-admin-acess"{
	user = aws_iam_user.admin.user.name
	policy_arn=aws_iam_policy.adminUser.arn
}
------------------------
AWS S3 : simple storage service in aws platform
data in s3 storage in the form of S3 bucket (container or directory)
in the bucket every thing is object
when we create a bucket, aws also create automacally a dsn for it
bucket name : must be unique (no upper case or _ )

bucket with terraform 

-create a bucket  
ressource "aws_s3_bucket" "finance"{
	bucket ="finance-2100000"
	tags={
	Description=""
	}
}
-create a object in bucket 
ressource "aws_s3_bucket_object" "finance-2020"{
	source ="/root/finance/fincance-2020.doc"
	key="finance-2020.doc"
	bucket="aws_S3_bucket.finance.id"
}
-who can acess in this bucket (policy)
	create iam group aws_iam_group with name finance-data, 
	s3 bucket policy like iam policy with :
	policy=<<EOF
		{
		"Version": "202....."
		"Statement":[
			{
				"Effect": "Allow",
				"Action" : "*",
				"Ressource": "arn:aws:s3:::${aws_s3_bucket.finance.id}/",
				"Principal":{
					"AWS": [
						"${data.aws_iam_group.finance-data.arn}"
					]
				}
			}
		]

		}
	EOF
----------------------
DynamoDB terraform : nosql
resource "aws_dynamodb_table" "cars"{
	name ="cars"
	hash_key="VIN"
	billing_mode="PAY_PER_REQUEST"  // 
	attribute{
		name="VIN"
		type="S"
	}

}
add items :

resource "aws_dynamodb_table_item" "car-items"{
	table_name = aws_dynamodb_table.cars.name
	hash_key= aws_dynamodb_table.cars.hash_key
	item = <<EOF
		{
			"Manufacturer" :{"S": "Toyota"},
			"VIN" :{"S": "4Y1S..."},
			"Year" :{"N": 2004},
		}
		EOF
}
---------------------
Remote state 

 do note push state file in git repo
 do not run terraform file in same time
 ---------------------
 Remote backend

 we need S3 bucket for backend storage
 terraform{
 backend "s3" {

 	bucket =""
 	key =""
 	region = ""
 	dynamodb_table =""

 	}
 }
 affter running terraform apply, we can delete the state file. the state file will not be in the local configuration anymore
 to view state file : terraform state pull 
 --------------------------------
 terraform state command

 terraform state list : list f createtd ressource
 terraform state show resource : details of ressource
 terraform state mv source destionation : move items to one state file to another (rename table for example)
 terraform state pull : to view state file in s3 storage
 terraform state rm ressource_adress : remove only in terrraform mangement.

 ----------------------------------
 EC2 instance : elastic instance cloud , it is virtual machine in the cloud 

deploy EC2 instnacr with terraform 
ami : amazon image

ressource "aws_instance" "webserver"{
	ami = "ami-..."
	instance_type = "t2.micro"
	tags={
	Name=""
	Description=""
	}
	user_data=<< -EOF
		#!/bin/bash
		sudoe apt update 
		sudo apt install ..
		systemcl enable ..
		EOF

	key_name = aws_key_pair.web.id
	vpc_security_group_ids=[aws_security_group.ssh-access.id]

}
create ssh key :
ressource "aws_key_pair" "web" {
	public_key =file (.../web.pub) or 
	public_key ="ssh-
	rsa AAAAAAAAAAAAAAAAAbbiyhghxcjhvbjk................."
}

create a ssh accss
ressource "aws_security_group" "ssh-access"{
	name =""
	description="Allow ssh access from the internet"
	ingress{
		from_port=22
		to_port=22
		protocol="tcp"
		cidre_blocks=["0.0.0.0/0"]
	}
}

-----------------------------
terraform provionners : it is inside the ressource block 
provisionner "remote-exec"{
	inline=[
		"sudoe apt update ",
		"sudo apt install ..",
		"systemcl enable ..",
	]
}

to save the public ip : 

provisionner "local_exec"{
	command="echo ${aws_instance.webserver.public_ip} >> /tmp/ips.txt"
}

we cans also have connaction block inside ressource block : 
connection {
	
	type ="ssh"
	host= self.public.ip
	user=""
	private_key=file ()
}

local vs remote exec  : remote need ssh connection 
---------------------------------
Modules 

https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest/submodules/iam-user

to resolve duplacated code issues

module "dev-webserver" {
	source ="../aws-instance"
}

module from public registry 
after copy and past the code, run :
terraform get . to get module 
----------------------------------------------
functions in terraform


list(), toset(), file(), ...

lunch interractive console to try some functions: 
terraform console 
--------------------------------
Conditional expressions 

condition ? true_val : false_val
EX : 
resource "radom_psw" "psw-generator"{
	legth= var.length < 8 ? 8 : var.length

}


create sum user with string 

resource "aws_iam_user" "cloud" {
     name = split(":",var.cloud_users)[count.index]
     count = length(split(":",var.cloud_users))

}

resource "aws_s3_bucket_object" "upload_sonic_media" {
     bucket = aws_s3_bucket.sonic_media.id
     source =each.value
     key = substr(each.value, 7, 20)
     for_each = var.media 
  
}

resource "aws_instance" "mario_servers"{
	ami = var.ami
	instance_type = var.name == "tiny" ? var.small : var.large
	tags={
	Name=var.name

	}

}
---------------------------
workspace

avoid repetition steps in different configurations directory

terraformm workspace new ProjectA
terraformm workspace list

create resource based a workspace :
configured variable.tf to have different value for variables 

variable ami {
	type =map
	default={
		"ProjectA"="ami-....."
		"ProjectB"="ami-....."
	}
}
and use 
 
 tags ={
 	Name=terraform.workspace
 }


