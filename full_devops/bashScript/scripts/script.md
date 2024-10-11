# bash
- if you want to make a permanent variable, add it in /etc/.bashrc file
- export season="monsoon"
- exit and login
-  if you want to create variable fo al user, add to /etc/profile


## scheduled script
- run : crontab -e *
- it opens a vi file 
- add in the file :
   # MM  HH DOM mm DOW 
   # 30  20 * * 1-5
   * * * * *  /opt/scripts/11_monit.sh &>> /var/log/monit_http.log

- DOW : day of the week
- MM : minutes
- HH : hour
- mm : month
-   * * * * * : every minute of every hour of every day of every month of every day
- &>> : redirect output


## remote command execution
- run different vm via vagrant
- change hostnames via /etc/hostname for each vm
- ssh vagrant@web01 or 'vagrant ssh web01' for ubuntu 
- ad devops user to all vms : useradd devops or 'adduser devops' in ubuntu
- passwd devops
- add devops to 
- add devops user to sudo with command : visudo

### ssh key exchange
- connect to web01
- gen ssh keys in web01:  ssh-keygen
- to connect by using ssh for user devops : ssh-copy-id devops@web01
- to connect by using ssh for user devops in web02 : ssh-copy-id devops@web02
- to connect by using ssh for user devops in web03 : ssh-copy-id devops@web03

- create host file in web01: vim hostname
  web01
  web02
  web03
- for $host in `cat $hostname`; do ssh devops@$host sudo yum install git; done 