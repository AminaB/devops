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
#### install java
    - yum install  java-11-amazon-corretto 
    - hostnamectl : to see amazonlinux version
    - edit env variable to .bash_profile 
        JAVA_HOME=/usr/lib/jvm/java-11-amazon-corretto.x86_64
        PATH=$PATH:$HOME/bin:$JAVA_HOME
#### install maven
    - use wget to download package : wget https://dlcdn.apache.org/maven/maven-3/3.9.1/binaries/apache-maven-3.9.1-bin.tar.gz
    - unzip tar xzvf 
    - copy maven to /opt : cp -r src dest
