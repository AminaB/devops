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
    install maven
     set en var for java Maves and M2=../bin
    find /-name jvm