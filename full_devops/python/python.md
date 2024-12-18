# python in ci cd pipeline
- NB : always prioritize ansible and terraform to python, when it's come to do configuration management.
- Use python when terraform ans ansible are not satisfying your work.
- we can use python to execute task in jenkins, like OS task.
- there is many python libraries for jenkins, to execute task on jenkins from python.
# python libraries
- python-jenkins : you can create job, copy existing job, delete job, install plugins,...
- python boto : for aws services
- 
# Tasks
- Add Users
- Add Groups
- Add users to group
- Create directory
- Assign user & group ownership to directory
- Tes if user or dir exists, if not, create it
- ssh python
- Fabric library
- Webserver provisioning with python fabric
- Python virtualenv
- Python for various other tasks

## Create VMS 
- cd python 
- vagrant up
## VMs setup
- vagrant ssh scriptbox
- mkdir /opt/pyscripts
- cd /opt/pyscripts
- mkdir ostasks
- cd ostasks
- create python script ostasks.py to add user, group, directory and Assigning permission and ownership to the directory
## Fabric : is a python library
` fabric is going to use ssh to connect to remote machine`
`and run function from the shell`
- pip : is a tool to install python library in the internet
- install pip : 
  - wget https://bootstrap.pypa.io/pip/get-pip.py
  - we need python2 for this script
  - apt install python
  - chmod 777 get-pip.py.py
  - python get-pip.py.py
- install fabric : pip install 'fabric<2.0'
- mkdir fabric
- vi fabfile.py
- to lis functions : fab -l
- run greetings function : fab greeting:Evening
- config web01 and web02
  - vagrant ssh web01
  - useradd devops
  - password
  - give root privilege : visudo 
    - add line : devops All=(ALL) NOPASSWD:ALL
  - enable pwd based login  
    - vi /etc/ssh/sshd_config
      - PasswordAuthentication yes
  - systemctl restart sshd
- login to scriptbox : vagrant ssh scriptbox
- ssh-keygen for vagrant user and root user
  - copy ssh key to web01 : ssh-copy-id devops@192.168.10.3 
  - copy ssh key to web02 : ssh-copy-id devops@192.168.10.4
  - now we can log from scriptbox to web01 and web02 without pwd asking
- execute from remote : fab -H 192.168.10.3 -u devops remote_exec
- Setup web server from remote : fab -H 192.168.10.3 -u devops web_setup:https://www.tooplate.com/download/2137_barista_cafe.zip,2137_barista_cafe.zip
