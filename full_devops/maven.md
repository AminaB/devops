# maven
## lifecycle
    source code - compile - test unit (integration test) - packaging - health check

## lunch ubuntu ec2 and install infra
- ssh into ec2
- sudo apt update
- install mvn and jdk
  - sudo apt search jdk
  - sudo apt install openjdk-11-jdk
  - sudo apt install maven
  - mvn -v
- if we want different version of maven 
  - go see in mvn archive 
  - wget link
  - tar xzvf apache-maven.gz
  - sudo mv apache-maven-3.9.3 /opt/
- clone https://github.com/hkhcoder/vprofile-project.git
- cd vprofile-project 
- mvn validate 
- mvn test : download dependencies and create target 
- mvn clean : delete target folder
- mvn install : execute all phases and create the artifact for deployment
- mvn clean install : clean before install

# delete all dependencies and reinstall ()
for example if you want to change maven version, remove the old dependencies
- rm -rf /home/ubuntu/.m2/repository/*
- mvn clean
- /opt/apache-maven-3.9.3/bin/mvn install

if you change version in pom, rebuild the app