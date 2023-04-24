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
#### connect to instance with ssh (control a remote machine, ) client 
    - add chmod 0400 to key
    - to connect : ssh -i /keypair ec2-user@IPAdrese
    - to exit : exit or ctrl d