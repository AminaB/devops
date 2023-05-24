# Simple Devops Project 
    github, kuberntes, github, aws, docker jenkins, maven, ansible 
## create EC2 instance for jenkins server and active port 8080, and install
-----------------------------------------------------------------------
    connect to ec2 
        ssh -i aminatou_key_pair.pem ec2-user@13.50.246.245
-----------------------------------------------------------------------
    in aws ec2  java and jenkins are vailable in epl packages

        sudo amazon-linux-extras (display packages)
        sudo amazon-linux-extras install epel  
        sudo amazon-linux-extras install java-openjdk11
        yum install jenkins

        jenkins pwd :7b2aa7e962f74f23bcbdd73c82cc8fa9
        sudo systemctl start jenkins
        change hostname in (/etc/hostname)
-----------------------------------------------------------------------
    install git and configure in jenkins
        yum install git
          https://github.com/yankils/hello-world.git
----------------------------------------------------------------------
    install maven in /opt an add to path
     set en var for java Maves and M2=../bin
     find /-name jvm
--------------------------------------------------------------------------
#### build java project
    new job (mavenProject) -> maven project -> Git -> source code =https://github.com/yankils/hello-world.git
    -> Goals and option = clean install -> apply save
---------------------------------------------------
#### integrate tomcat server in ci cd pipeline (new ec2 instance)
    install tomcat in /opt (tar -xvzf)
    start tomcat service : run this script /opt/tomcat/startup.sh 
    if you want to access web app from outside, edit tomcat/.../META-INF/context.xml ( and restarrt tomcat)
    stop tomcat service : run this script /opt/tomcat/shutdown.sh 
    
    integrate tomcat with jenkins
    in jenkins -> admin -> configure -> setup password -> apply save
    Manage jenkins -> install "deploy to container" pluggin 
    Manage jenkins-> Manage credentials -> jenkins -> global credentials -> enter tomcat credentials
    new job (buildanddeploy) -> maven project -> git source : ...-> Goals and options = clean install ->
    -> post build actions ->Deploy war to a container -> war files = webapp/target/ webapp.war (or **/*.war)->
    -> containers = tomcat 8.x -> select credentials -> tomcat url =@ip:8080 -> apply save

    automate build when there is a push in the repo
    configure Poll SCM in Build triggers section in the job

#### integrate docker in ci cd pipeline ( new ec2 instance)
    install and start docker 
    change hostanme in /etc/hostname
    reboot with : init 6 
---------------------------------
    pull tomcat
    to fix tomcat errors : ( copy webapp.dist content  to webapps directory )
    We will use a dockerfile for saving configuration and information in the container.
-------------------------------------
    Dockerfile:
        FROM centos
        RUN yum install java -y
        RUN mkdir /opt/tomcat/
        WORKDIR /opt/tomcat
        ADD https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.54/bin/apache-tomcat-9.0.54.tar.gz
        RUN tar xvfz apache*.tar.gz
        RUN mv apache-tomcat-9.0.54/* /opt/tomcat
        EXPOSE 8080
        CMD ["/opt/tomcat/bin/catalina.sh", "run"]
    docker build -t mytomcat .
    
    Customized Dockerfile for tomcat :
        FROM tomcat:latest
        RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
    docker build -t demotomcat .
--------------------------------------------
    Integrate docker with jenkins
    create a user with admin right :
        connect to docker host -> to see groups : cat /etc/group -> 
        create user add add him to docker group : useradd dockeradmin, passwd dockeradmin , id dockeradmin ( to see the current group),
        add group by : usermod -aG docker dockeradmin -> enable pwd authentication in ec2 instance -> vi /etc/ssh/sshd_config ->
        un comment "PasswordAuthentication yes" and comment "PasswordAuthentication no" -> :wq -> service sshd reload ->
        try to log with this user "dockeradmin"
    add dockerhost to jenkins server :
        install "publish over ssh" pluggin in jenkins
        Manage jenkins -> configure system -> got o publish over ssh section -> add -> server name = dockerhost -> hostname =@ip ->
        -> username = dockeradmin -> pwd=.. -> 
    create job to pull code from github, build with maven, and copy the artefact onto the dockker host
        job name = buildAnDdeployOntoContainer -> compy from = buildanddeploy -> ok -> delete post build actions section -> 
        -> add post build actions -> send build artefact over ssh -> ssh server = dockerhost -> source files = webapp/target/*.war->
        -> remove prefix = webapp/target  -> apply save 