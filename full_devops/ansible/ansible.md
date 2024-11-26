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
- provisioning
- orchestration : combine many automations tools

## lunch VMS
- lunch ubuntu EC2 for ansible: 22 allowed from my ip
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
<sub>

    # Inventory file: inventory.yaml
    all:
    hosts:
        web01:
            ansible_host: 192.168.1.10(private ip)
            ansible_user : ec2-user
            ansible_ssh_private_key_file: clientkey.pem

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
  <sub>
  
            children:
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
                  dbservers:
  
- test : ansible webservers -m ping -i inventory
- test : ansible dc_oregon -m ping -i inventory
- test : ansible all -m ping -i inventory
- test : ansible '*' -m ping -i inventory

  - test with vars (delete those lines at host level)
  <sub>

          vars:
              ansible_ssh_user: ec2-user
              ansible_ssh_private_key_file: clientkey.pem

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
<sub>

      - name: Webserver setup
      hosts:
      become:yes
      tasks:
        -name : Instal httpd
          ansible.builtin.yum
              name: httpd
              state: present
         - name : Start service
          ansible.builtin.service
              name: httpd
              state: started
              enabled:yes
- db config
<sub>

        - name: DBserver setup
        hosts:
        become:yes
        tasks:
          - name: Install mariadb-server
            ansible.builtin.yum:
                name: mariadb-server
                state: present

- remove httpd: ansible webservers -m yum -a "name=httpd state=absent" -i inventory --become
- run : ansible-playbook -i inventory web-db.yaml (-v -vv -vvv -vvvv  for debugging information) (--syntax-check to check playbook syntax)
- ansible modules : https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html (ex: ansible.builtin.yum)
- test copy file : https://docs.ansible.com/ansible/2.9/modules/copy_module.html#copy-module
<sub>

        - name: Copy file with owner and permissions
          copy:
          src: files/index.html
          dest: /var/www/html/index.html
          backup: yes`
- tes db modules : https://docs.ansible.com/ansible/2.9/modules/mssql_db_module.html#mssql-db-module
- require python >= 2.7 or pymssql
  - check in db01 server the exact package name : yum search python | grep -i mysql
<sub>
  
        # Create a new database with name 'accounts'
        - name : create db
        mssql_db:
        name: accounts
        state: present`      
            `-name : Instal pymysql
                  ansible.builtin.yum
                  name: python3-PyMySQL
                  state: present`
- resolve access denied for user root@localhost : ansible can't connect to mysql server
<sub>

        - name : create db
          community.mysql.mssql_db:
          name: accounts
          state: present
          login_unix_socket: /var/lib/mysql/mysql.sock

- add mysql user
<sub> 

          - name : create db
            community.mysql.mssql_db:
            name: vprofile
            password: 12345
            priv :'*.*:ALL
            state: present
            login_unix_socket: /var/lib/mysql/mysql.sock

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
<sub>

            [default]
            host_key_checking =False
            inventory=inventory
            forks =5
            log_path =/var/log/ansible.log
            [privilege_escalation]
            become=true
            become_method=sudo
            become_ask_pass=False
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
<sub>

        vars : 
            dbname: electric
            dbuser: current
            dbpass: tesla

to call the variable : "{{dbname}}"
- we can create task to print the variable
<sub>

          debug:
            msg: "{{dbname}}

## variable outside playbook
- mkdir group_vars
  - vim group_vars/all
<sub>
  
        dbname: sky
        dbuser: pilot
        dbpass: aircraft
- run the playbook : comment the vars section inside playbook (playbook vars have higher priority)

### host var
- vim host_vars/web02
<sub>

        USRNM: web02user
        COMM : variables from host_vars/web02 file

### command line varibales (higher priority) : ansible-playbook -e USRNM=cliuser -e COMM=cliuser vars_precedence.yaml
  https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html
## Fact variables : runtime variables
generated when setup module executed
- we can disable these variables :
<sub>

        gather_fasts: FALSE
- ansible -m setup web01 : to see fact variables for web01
  - print variable using playbook
  <sub>
      
          tasks
            - name: print os name
              debug:
                  var: ansible_distribution
- run

## Decision-making
### provisioning chrony on centos and ntp on ubuntu
- create file provisioning.yaml
<sub>

      - name: Provisioning servers
          hosts : all
          become: yes
          tasks:
              - name : install ntp agent on centos
                yum :
                  name: chrony
                  state: present
                when: ansible_distribution =="Centos"
              - name : install ntp agent on ubuntu
                apt :
                  name: ntp
                  state: present
                  update_cache: yes
                when: ansible_distribution =="Ubuntu"
              - name: start service on centos
                service:
                    name: chroneyd
                    state: started
                    enabled: yes
                when: ansible_distribution =="Centos"
              - name: start service on ubunt
                     service:
                        name: ntp
                        state: started
                        enabled: yes
                        when: ansible_distribution =="Ubuntu"

- run
## Loops
- loops
<sub>

        tasks:
            - name : install ntp agent on centos
                yum :
                    name: "{{item}}"
                    state: present
                    when: ansible_distribution =="Centos"
                    loop:
                        - chrony
                        - wget
                        - git
                        - zip
                        - unzip

## Files
- file
<sub>

      - name: Banner file
          copy:
              content: '# This is manged by ansible. no manual changes please'
              dest: /etc/motd
- create folder
- add in playbook : vars : mydir: /opt/dir22 
<sub>

      - name: Create Folder
          copy:
              path: "{{mydir}}"
              state: directory

### Template module 
- create ntpconf_centos file in ansible server
  - copy from centos server /etc/chrony.conf content into ntpconf_centos file in ansible server
  <sub>

        - name: deploy ntp agent conf on centos
          ansible.builtin.template:
              src: templates/ntpconf_centos.j2
              dest: /etc/chrony.conf
              backup: yes 
          when: ansible_distribution =="Centos"
        - name: ReStart service
            service:
              name: chroneyd
              state: restarted
              enabled: yes
            when: ansible_distribution =="Centos"
- cons : service restart even there is no change
- same for ubuntu machine
<sub>
  
        dest : /etc/ntp.conf
- add tasks to restart ntp on ubuntu and chronyd on centos
- with template variable will be replaced by their values, template module is intelligent, no variable replacement with copy module

## Handler
- we do not want to restart the service
  - only restart when configuration file  change
  <sub>
  
        tasks:
            - name: deploy ntp agent conf on centos
              ansible.builtin.template:
                  src: templates/ntpconf_centos.j2
                  dest: /etc/chrony.conf
                  backup: yes 
                  when: ansible_distribution =="Centos"
                  notify: 
                    - ReStart service on centos
            - name: ReStart service
                service:
                  name: chroneyd
                  state: restarted
                  enabled: yes
                when: ansible_distribution =="Centos"
        handlers:
            - name: ReStart service on centos
                service:
                  name: chroneyd
                  state: restarted
                  enabled: yes
                when: ansible_distribution =="Centos"
  
- add handler for ubuntu conf too 
## Roles :
reusable in different project or different env, and structured
## create folders structures
- mkdir roles
- cd roles
- ansible-galaxy init post-install
- copy groupvars_all content form exercise15 into roles/post-install/vars/main.yaml
- copy other variables (playbook, ...) too inside roles/post-install/vars/main.yaml
- remove groupvars_all and host_vars
- cp files/*  roles/post-install/files/
- cp templates/*  roles/post-install/templates
- copy handlers from playbook to roles/post-install/handlers/main.yaml (delete extra spaces)
- copy tasks from playbook to roles/post-install/tasks/main.yaml
- delete handlers and task inside playbook
- rm -fr files templates
<sub>

      - name: Provisioning servers
        hosts : all
        become: yes
        roles:
          - post-install
     
- don't need to give template path in tasks, or files path, just the name of the file 
  - vim roles/post-install/tasks/main.yaml
  <sub>

        - name: deploy ntp agent conf on centos
          ansible.builtin.template:
            src: ntpconf_centos.j2
            dest: /etc/chrony.conf
            ...

- test the playbook 
  - we can also copy the variables inside roles/post-install/defaults/main.yaml ( but it has lower priority)
    - we can overwrite variable
    <sub>

          - name: Provisioning servers
            hosts : all
            become: yes
            roles:
              - role : post-install
                  vars :
                    ntp0: 0.europe.pool.ntp.org
                    ntp1: 1.europe.pool.ntp.org
                    ntp2: 2.europe.pool.ntp.org
                    ntp3: 3.europe.pool.ntp.org
  
- we can download roles created by others here : https://galaxy.ansible.com
- download java role setup : ansible-galaxy install geerlingguy.java
<sub>
  
      roles:
        - role: geerlingguy.java
        - role: post-install

- it is better to create role by yourself, with downloaded role sometimes it's can be hard to do Custom Features
- you can also learn from downloaded roles to create your own
- ansible is easy to write (use documentation)

## Ansible for AWS
### Authentication
- export access keys
- create iam user : ansible-admin, AdministratorAcces, generte Access keys, download csv file
  - run into ansible-server:
      export AWS_ACCESS_KEY_ID='AK123'
      export AWS_SECRET_ACCESS_KEY='abc123'
  - but when you exit this variable are gone, so add it to .bashrc file
- vim .bashrc : add at the en of the file
  export AWS_ACCESS_KEY_ID='AK123'
  export AWS_SECRET_ACCESS_KEY='abc123'
- run : source .bashrc (or logout and login)
- authentication is done

### Ansible playbook for key pair
- mkdir aws
- cd aws
  - vim test-aws.yml
  <sub>

        - hosts: localhost
          gather_facts: False
          tasks:
            - name : Create key pair
              ec2_key:
                name : sample
                region : us-east-1
              register: keyout
            #- name: print key
              # debug:
                 # var: keyout
            - name: save private key content
              copy:
                content: "{{keyout.key.private_key}}"
                dest: ./sample-key.pem
              when: keyout.changed

- ansible need python(boto3) to access libraries
  sudo apt install python3-pip -y
  pip3.10 install boto3
- run : ansible-playbook test-aws.yml
### Ansible run an EC2 instance : https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_instance_module.html
- install aws collection : ansible-galaxy collection install amazon.aws
- add to playbooks
<sub>

      - name: start an instance with a public IP address
        amazon.aws.ec2_instance:
          name: "public-compute-instance"
          key_name: "sample"
          #vpc_subnet_id: subnet-5ca1ab1e
          instance_type: t2.micro
          security_group: default
          # network_interfaces:
            # - assign_public_ip: true
          image_id: ami-016b213e65284e9c9
          exact_count: 1
          region: us-east-1
          tags:
            Environment: Testing
- test it and terminate instance,  delete key