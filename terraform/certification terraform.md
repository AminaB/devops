certification terraform
in aws we have an IAC called aws cloud formation, we can easily drag and drop infrastructure in this platform.
document 
https://github.com/zealvora/terraform-beginner-to-advanced-resource

it recommended to use terraform for approving and ansible for configuration

for example : we can use terraform to create an EC2 instance. when EC2 instance is running, terraform call ansible to configure and install application.

Install terraform, and IDE (Atom), and install the plugin language-terrafom in atom editor

---------------------
how terraform authenticate in aws:
there is many ways : static credentials, Environment variable,..
we use static credentials in this course : 
get the code to :https://registry.terraform.io/providers/hashicorp/aws/latest/docs

let's create a user with access key in aws and use it in terraform configuration.

create an aws instance with terraform : https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/instance
-----------------------------
access github with terraform
https://registry.terraform.io/providers/integrations/github/latest/docs
create a token in github
----------------------------
destroy 

we can destroy a specific resource 
terraform destroy -target type.ressource_name

-------------------------
State file 

if we create a resource, the details of the resource will in the state file
if we destroy this resource, it will be destroyed in state file 
-------------------------
Desire state vs current state

the resource in tf file is the desire state.
current state is the actually state currently deployed

-----------------------------------
provider versioning

~>2.0 it is 2.X
-------------------------------------

Associate two resources
EX : EIP and EC2 instances
