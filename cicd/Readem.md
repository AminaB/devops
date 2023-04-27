## CI CD
    - buy domain name : godaddy
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
    - in jenkins job,  add post-build action -> sen build artifacts over ssh
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
#### create docker file to automate those config
        