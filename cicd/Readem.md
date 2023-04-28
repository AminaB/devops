## CI CD
    - buy domain name : godaddy
    - hello world projetc for this course : https://github.com/rahulshettyacademy/DevopsBasics.git
#### command to build projects
    java :  mvn clean install (generate war file, to deploy in tomcat server, in App/Web Server)
    angular : dist -ng -build
    - CI : process of automation the build and test of code every time a team member commits.
    - CD : process to build test configure and deploy from build to production.
#### JAva, MAVEN & jenkins setup on aws Linux EC2 instance
     crate two linux machine, one for code and jenkins for build, jenkins send war to second machine(with tomcat server) for deploy.
    - ceate aws ec2 instance.
    - connect to EC2 instance in local machine
#### connect to instance with ssh (secure shell, control a remote machine, ) client / and config
    - AWS linux distrubution is closed to Fedora distribution
    - add chmod 0400 to key
    - to connect : ssh -i /keypair ec2-user@IPAdrese
    - to exit : exit or ctrl d
    - install yum package to do installation in ec2 instance
#### commands fo Amazon Linux 2023
    - connect as root (sudo su -), and run command in root directory
    - 2023 Packages available here : https://docs.aws.amazon.com/linux/al2023/release-notes/all-packages-al2023-20230419.html
### EC2 instance 1 ( amazon linux 2023) for continuous integration
#### install java
    wget https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.deb (chearch for working jdk, this link generate error in jenkins in a linux 2023)
    - yum install  java-11-amazon-corretto 
    - hostnamectl : to see amazonlinux version
    - edit env variable to .bash_profile 
        JAVA_HOME=/usr/lib/jvm/java-11-amazon-corretto.x86_64
        PATH=$PATH:$HOME/bin:$JAVA_HOME
#### install maven
    - use wget to download package : wget https://dlcdn.apache.org/maven/maven-3/3.9.1/binaries/apache-maven-3.9.1-bin.tar.gz
    - unzip tar xzvf
    - copy maven to /opt : cp -r src dest
#### install jenkins
    for ec2 instance use fedora distritubition
    https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos
    - start : sudo systemctl start jenkins
    - status : sudo systemctl status jenkins
    - stop : sudo systemctl stop jenkins  
    - create job
    - configure git java and maven path in jenkins manager
    - build job to create war file
### EC2 instance 2 ( amazon linux 2023) for continuous deployment
    - create new ec2 instance
    - connect to it with user ec2-uer
    - configure user(ec2-user) with password( sudo psswd ec2-user)
    - enable pwd authentification in vi /etc/sshd_config (as root). service sshd reload
#### manage jenkins to deploy war file
    - in manage jenkins install new pluggins (publish over ssh)
    - configure system in jenkins : in publish ssh section fill information about server.
        - advanced (use password and give it
    - in jenkins job,  add post-build action -> send build artifacts over ssh
    - save file in home direcory of ec2-user ("." in remote directory input)
    - run job and we will see war file in home.
#### install docker (to run tomcat) : 
    we cannot run many tomcat in physic computer. if later we have differents app using deffereent java version, we need different tomcat versions server
    so, with docker we can run in pararell 2 or more containers with tomcat running on them.
    - cat /etc/group : to see all groups available for root user
    - giving permission access to group
        usermod -aG docker ec2-user
    - login to ec2-user
    - sudo service docker start
    - create container with tomcat image runing on it: 
        docker pull tomcat:latest
        docker run -d -p 8080:8080 --name tomcat-container ImageID 
    - docker exec -it tomcat-container /bin/bash
        cd webapps.dist
        cp -r ./* ../webapps
    - copy webapp.war file from my amazon linux to my tomcat container
        sudo docker cp webapp.war tomcat-container:/usr/local/tomcat/webapp.war
    - verified tomcat container is still up : docker ps
    - go to @ip:8080/webapp to see your site web
#### create docker file to automate this config
    - sudo vi Dockerfile
    FROM tomcat:latest
    COPY ./webapp.war /usr/local/tomcat/webapps
    Run cp -r /usr/local/tomcat/webapps;dist/* /usr/local/tomcat/webapps
    - create image from docker file
      sudo docker build -t custom-image .
    - run image 
     docker run -d -p 8080:8080 --name custom-container custom-image
    - go to @ip:8080/webapp to see your site web

#### we can create image from jenkins to automate
    - add these commands to post-build actions -> transfers-> exec command :
      sudo docker build -t custom-image .
      docker run -d -p 8080:8080 --name custom-container custom-image
    - inconvenient of create container in jenkins.
          jenkins always create an image
          if we run another build, it will fail because we have already a running container with same name
          first we have to stop and remove manually the container before running new build
#### ssh connection to another linux server
    - ssh connection from server x to server y :
        - server x: ssh-keygen
        - server x : cd .ssh ( we have to keys private and public)
        _ in server 2 allow password authentication
            sudo su -
            vi /etc/ssh/sshd_config : passwordauthent to yes
            service sshd reload
            add password to ec2-user : passwd ec2-user
        - deploy public key to server y
            server x : ssh-copy-id ec2-user@@ipy : the public key wil be in .ssh/authorized_keys in server y 
        - now you can connect to server y
            ssh ec2-user@@ipy
    - exit 
#### Ansible 
    is a tools that used for configuration management (Installing software, ... )
    ansible cans connect to nodes(our linux machines through ssh) and push small program, and delete them when we finished
    ansible use yaml language 
    - install ansible : ansyble is a python package, first we have to install python
        yum install python
        pip install ansible
        ansible --version
    - add sudo access to ec2-user
        echo "ec2-user All=(ALL) NOPASSWD:ALL">> /etc/sudoers
    stop and delete docker containers
    delete docker images
    - create yaml file for config docker commands in it
       see file : firsplaybook.yml
    - create a host file in root of ec2-user, to add all ip@ where you want to run this ansible config
        vi hosts
    - run ansible config file
        NB : ansible connect to localhost with ssh, so we had better add publicKey into Authorize ( see "ssh connection to another linux server" section)
        ansible-playbook -i hosts firstplaybook.yml
#### use ansible to jenkins to deploy
    - remove all docker stuff in jenkins and replace it by this command : ansible-playbook -i hosts firstplaybook.yml
#### we can deploy with ansible in many servers by adding their @ip in hosts (ec-user@@ip).
    - first we can copy dockerfile and webapp from one server to another if their is not present (update .yml file)
    - check if docker is installed to all servers ( we cans also install docker in ansible config file)
    - we can run ti in only one machine even if their are many in hosts
          ansible-playbook -i hosts firstplaybook.yml --limit ec2-user@@ip
        upadet command on jenkisn
#### Summary :
    Using Ansible we can write playbooks which can run automation script with set of commands on any machine.
    You can deploy your code into n number of machines by having IP Address list in hosts file and running Ansible 
    deployment playbook yaml file
    Greatest advantage of deploying Apps in Docker Containers is that we need not worry about 
    existing software installations and their version compatibility
    We can create Deployment environment in docker file so that it creates image out of it and deploy in 
    container in any machine instantly with out worrying about downloading the softwares manually
    Deploying Apps using Ansible + Docker on Linux and automating the Continuous Delivery process 
    with Jenkins is the powerful Process followed in Devops World.

#### automate integration (without running mually the job)
    achieve continuous integration with github hook in jenkisn
    - in jenkis server -> manage jenkins -> configure system-> gitHub -> advanced -> activr "Specify another hook for GitHub.."
    -> copy url
    - in github repo go to settings -> webhooks -> add webhook -> past url copied from jenkins in "paylod URL"
    - go to job -> configure -> build triggers -> select "Github hook trigger for gitscm polling" -> aplly
    - do some changes commit and push, we will see buil starting automatically in jenkins

#### parameterize jenkins to deploy in multiple environment
    ansible-playbook -i hosts firstplaybook.yml --limit QA (see hosts file, we rename server to QA)
    - parametirized the build, create another job
        select "This project is parameterized" -> choice parameter -> name "env" ->choices ( QA /n QA1 /n  prod...)
    - in exec command 
        ansible-playbook -i hosts firstplaybook.yml --limit ${Environment}
    - to buil click on "build with patameters" asn select env