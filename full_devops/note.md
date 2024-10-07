https://github.com/devopshydclub/vprofile-project
nuy a domain name : godaddy
create dockerhub account
login to sonarcloud


aws : 
 certificate the domain name which I bought in godaddy in aws certificate manager service 
- Fully qualified domain name = *.barryit.xyz
- add record to godaday dns by adding cname and cname value of certificate
- iam user alias : aminainfo


Software Development Life Cycle (SDLC).
SDLC Agile 
- small list iterations, each itr has dev, production, demo, and feedback

Dev are agile and do regular and quick change, ops are looking fo stability

Devops : 
- reduce time to market :
- fix code delivery issues
- ops and dev communicate
- train dev teams in ops concept
- train ops to be agile
- automation in every level

Devops lifecycle
- commit -> build -> integration test -> code analyse -> delivery -> db/sec changes ->  software testing -> deploy -> user approval -> keep monitoring

Linux :
- server linux os : red hat, ubuntu server, centos,..
- desktop linux : ubuntu, debian,...
- to see architecture of the cpu tape command: arch

vagrant vs terraform : both projects from HashiCorp.
- Vagrant is for development environments. Terraform is for more general infrastructure management.
- vagrant provisioner : shell. config.vm.provision "shell" (in vagrant file)
