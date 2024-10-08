# Scenario

- variety services : php jquery wordpress joomla ubuntu ...

# problem
- not comfortable making changes in real severs
- local setup by using VM is complex, time-consuming, not repeatable
# solution 
- automate local setup
- make it repeatable
- code : iac
## tools 
- hypervisor : virtualbox
- automation : vagrant
- cli : git bash
- ide

# Stack of Vprofile project
- nginx : to create the load balancing experience, to route the req to tomcat server
- NFS : shared storage
- Rabbit MQ connected to tomcat : Message broker, queueing agent
- Meme cache : connected to mysql to cache user information
- MariaDb

# manual installation
- run vagranfile based on your os to install VMS
## db01
- see doc into : /home/aminatou/Documents/cours/devops/full_devops/VProfileProject/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.pdf
- yum install epel-release -Y : for extra repo
- install maria db and git into db01 vm 
- start and enable mariadb 
- config security my running  : mysql_secure_installation
- setup db : mysql -u roo -padmin123, grant privilege to admin to connect remotely, flush privileges
- use script accountsdb.sql to create db 
- restart serice

## memecahe : mc01
- see doc : /home/aminatou/Documents/cours/devops/full_devops/VProfileProject/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.pdf

## rabbitmq : rmq01
- see doc : /home/aminatou/Documents/cours/devops/full_devops/VProfileProject/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.pdf

## tomcat : app01
- see doc : /home/aminatou/Documents/cours/devops/full_devops/VProfileProject/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.pdf

## nginx : web01