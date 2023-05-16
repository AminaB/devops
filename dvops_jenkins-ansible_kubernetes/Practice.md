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