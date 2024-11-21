# Ansible
Idempotency: They ensure that the desired state is maintained. 
Running the tool multiple times won’t make changes if the system is already in the correct state.
- Configuration management tools : ansible, puppet, chef, salt stack, bash scripting, python, perl, ruby,...
- Ansible written in python
- No complex setup (python library): you can install its by using pip
- No agent : ssh connections on ubuntu, WINRM on windows, APIs for clloud
- Yaml
- With ansible, we can do API calls, execute shell commands, script
## use cases
- automation
- change management : production servers change
- previsioning
- orchestration : combine many automations tools

## lunch VMS
- lunch ubuntu EC2 : 22 allowed from my ip
- lunch 3 centos9 EC2s: name web01(02) and db01, create client-key-pair, ssh from my ip, 22 from ansible-sg

## setup ansible 
- ssh to ansible-server
- install ansible : https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu
- ansible --version
- create project directory : mkdir vprofile
- cd vprofile
- mkdir exercise1
- cd exercise1
## Inventory file and test with ping command
the inventory file defines the hosts (servers or devices) that Ansible manages.

` # Inventory file: inventory.yaml
all:
    hosts:
        web01:
            ansible_host: 192.168.1.10(private ip)
            ansible_user : ec2-user
            ansible_ssh_private_key_file: clientkey.pem`

- copy key : cat clientkey.pem
- ssh to ansible server
- cd vprofile/exercise1
- vim clientkey.pem
- past key content and save
- ansible web01 -m ping -i inventory

To avoid interaction with console like "fingrtprint when ssh", we need to create ansible configuration file
- cd /etc/ansible
- run : ansible-config init --disabled -t all > ansible.cfg
- vim ansible.cfg
- uncomment and modify : host_key_checking=False
- save and quit
- chmod 400 clientkey.pem
- test : - ansible web01 -m ping -i inventory

- test for web02 and db01

- add group in inventory file 

`children:
webservers:
  hosts:
      web01:
      web02:
dbservers:
  hosts:
      db01:
dc_oregon:
  children:
      webservers:
      dbservers:`
- test : ansible webservers -m ping -i inventory
- test : ansible dc_oregon -m ping -i inventory
- test : ansible all -m ping -i inventory
- test : ansible '*' -m ping -i inventory

- test with vars (delete those lines at host level)

` vars:
    ansible_ssh_user: ec2-user
    ansible_ssh_private_key_file: clientkey.pem`

## Ad Hoc Commands
good for quick task we repeat rarely ex : reboot machines
- run : ansible web01 -m ansible.builtin.yum -a "name=httpd state=present" -i inventory --become
- absent state : ansible web01 -m ansible.builtin.yum -a "name=httpd state=absent" -i inventory --become : remove package
- ansible web01 -m ansible.builtin.service -a "name=httpd state=started" -i inventory --become 
- ansible web01 -m ansible.builtin.copy -a "src=index.html dest=/var/www/html" -i inventory --become (create index.html file before)
- test by adding port 80, and verify to browser

## Playbook and modules
Playbooks provide a way to orchestrate complex tasks and manage configurations across multiple systems in a structured
- vim web-app.yml
`
-name: Webserver setup
hosts:
become:yes
tasks:
  - name : Instal httpd
    ansible.builtin.yum
        name: httpd
        state: present
  - name : Start service
    ansible.builtin.service
        name: httpd
        state: started
        enabled:yes
name: DBserver setup`
hosts:
become:yes
tasks:
  - name: Install mariadb-server
    ansible.builtin.yum:
        name: mariadb-server
        state: present
`
- remove httpd: ansible webservers -m yum -a "name=httpd state=absent" -i inventory --become
- run : ansible-playbook -i inventory web-db.yaml (-v -vv -vvv -vvvv  for debugging information) (--syntax-check to check playbook syntax)
- ansible modules : https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html (ex: ansible.builtin.yum)
- test copy file : https://docs.ansible.com/ansible/2.9/modules/copy_module.html#copy-module

`- name: Copy file with owner and permissions
  copy:
  src: files/index.html
  dest: /var/www/html/index.html
  backup: yes`
- tes db modules : https://docs.ansible.com/ansible/2.9/modules/mssql_db_module.html#mssql-db-module

`# Create a new database with name 'accounts'
name : create db
mssql_db:
name: accounts
state: present`
  - require python >= 2.7 or pymssql
    - check in db01 server the exact package name : yum search python | grep -i mysql 
      
    `-name : Instal pymysql
          ansible.builtin.yum
          name: python3-PyMySQL
          state: present`
    - resolve access denied for user root@localhost : ansible can't connect to mysql server
    
    `name : create db
      community.mysql.mssql_db:
      name: accounts
      state: present
      login_unix_socket: /var/lib/mysql/mysql.sock`
    
    - add mysql user
    
    `name : create db
      community.mysql.mssql_db:
      name: vprofile
      password: 12345
      priv :'*.*:ALL
      state: present
      login_unix_socket: /var/lib/mysql/mysql.sock`

## Ansible configuration
ex : change the port of ssh connection
- ANSIBLE_CONFIG (environment variable if set)
- ansible.cfg (in the current directory)
- ~/.ansible.cfg (in the home directory)
- /ect/ansible/ansible.cfg (System level)
### read /ect/ansible/ansible.cfg
- ssh to ansible server
- cd vprofile
- sudo -i 
- vim /ect/ansible/ansible.cfg

### create our own ansible configuration
- vim ansible.cfg

`[default]
host_key_checking =False
inventory=inventory
forks =5
log_path =/var/log/ansible.log
[privilege_escalation]
become=true
become_method=sudo
become_ask_pass=False`
- save
- sudo touch /var/log/ansible.log
- chown ubuntu. ubuntu /var/log/ansible.log
- ansible-playbook db.yml (don't need to mention inventory, already in the config file)

## Variable & debug
we can define variable in playbooks, group vars, hosts vars, role, ...
- vars :
    http_port:80
    sqluser:admin
- Facts variables : ansible_os_family, ansible_processor_cores, ansible_kernel, ...
- Store Output 
- variable in playbook
- before tasks add :

`
vars : 
    dbname: electric
    dbuser: current
    dbpass: tesla
`
to call the variable : "{{dbname}}"
- we can create task to print the variable
  debug:
    msg: "{{dbname}}"

## variable outside playbook
- mkdir group_vars
- vim group_vars/all
  dbname: sky
  dbuser: pilot
  dbpass: aircraft
- run the playbook : comment the vars section inside playbook (playbook have higher priority)

- host var
- vim host_vars/web02
USRNM: web02user
COMM : variables from host_vars/web02 file

- command line varibales (higher priority) : ansible-playbook -e USRNM=cliuser -e COMM=cliuser vars_precedence.yaml 