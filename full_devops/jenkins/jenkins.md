# intro jenkins
extensible : many plugins
## CI : build code from VCS after every commit
- fetch code - build - test - notify developer

## installation 
- create ubuntu EC2, with ports 22 from my ip and 8080 from everywhere
- https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

    sudo apt update
    sudo apt install openjdk-21-jdk -y
    sudo apt install openjdk-17-jdk -y

    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    
    sudo apt-get update
    sudo apt-get install jenkins
- systemctl status jenkins
- on browser : pubIp:8080
- temporary pwd :  cat /var/lib/jenkins/secrets/initialAdminPassword 
- install plugin
- create user 
  - username : admin
  - pwd : admin
  - fullname : Admin 
  - jenkins url : http://admin.mydevops.xyz 
# jobs 
- job is workload run by jenkins
- freestyle jobs : graphical jobs (not recommended)
- pipeline as a code (recommended)
  - pipeline created in groovy

# config
- in manage jenkins -> tools 
- add jdk : 21 & 17
  - name : JDK21
  - JAVA_HOME :  /usr/lib/jvm/java-21-openjdk-amd64/
  - name : OracleJDK8
  - JAVA_HOME :  /usr/lib/jvm/java-8-openjdk-amd64/
- add maven
  - name MAVEN3
  - install automatically : version 3.9.3
- no need for git on ubuntu

# create job
- create job add shell  command
- file will be created in jenkins workspace ***own by user jenkins***

- build artifact of vprofile on jenkins 
  - create job
  - select : jdk11
  - source code : add project url (git) https://github.com/hkhcoder/vprofile-project.git
  - branch : */main
  - build steps : invoke top-level maven targets (we can also use command: mvn install, but always use plugins if it exists)
    - Maven3
    - Goal : clean install
  - save 
  - build job (if it doesn't work increase t2.micro to t2.small)
  - artifact will be in workspace
  - archive artifact : post-build actions -> files to archive : **/*.war

- we can create a job from another (create new item..)

# versioning

- we can save war file by adding shell (to add version of war)
    - mkdir -p versions
    -  cp target/vprofile-v2.war versions/vprofile-$BUILD_ID
- BUILD_ID : is jenkins env variable
- we can add param to build : passing by using $VAR
- we can add plugins : ex. build timestamp plugin (for versioning)

# Flow CI
***git -> jenkins -> SonarQube (code analysis) -> uploaded artifact to NexusOSS  repo***

## ci pipeline
on single  VM we have :
- sonarqube
- nginx
- PostgreSQL instead of mysql
create key pair form each vm
## VMs
nexus 
- nexus -server centos(centos stream9)
- t2 medium
- create SG : 22 from my ip, 8081 from myip and jenkinsSG
- copy nexus-setup.sh in user data
- lunch

sonarqube
- sonar-server
- iam : ubuntu 
- t2 medium
- SG : 22 from my ip, 80 from my ip and jenkinsSG
- copy sonar-setup.sh in user data

## test VMS
- ssh ...
- systemctl ...
- for nexus : ip:8081 (see welcome page), signin (create new pwd : nexuspwd)
- for sonar : ip:80, login =pwd=admin (updatepwd = sonarpwd)

## isntall pluggins on jenkins EC2
- manage jenkins - add plugging 
- nexus artifact
- sonarqube scanner
- build timestamp
- pipeline maven integration
- pipeline utility steps
- install without restart

## pipeline as code
jenkinsfile defines stages in CICD pipeline
`Pipeline{
  stages{
    // clone from vcs
    steps{}
    post{} // post-build
  }
  stages{
  // maven build
  }
}`

# code analysis ( checkstyle, cobertura, mstest, Sonarqube Scanner, etc ..)
- best practices
- vulnerability
- functional errors

## add sonar to jenkins
allow http for jenkins-sg in sonar-sg
- tools -  add sonarqube scanner 
- name : sonar6.2
- version : 6.2***
- install automatically

- go to configure system - SonarQube server - check env variable - 
- url = http://publicIp (remember to update when stop instance or use private ip)
- name : sonarserver
- token : got to sonar cube server, myacount - security - generate token (name=jenkins, type= user token) - copy token 
  - in jenkins choose secret text - past token
  - id : mysonarToken
  - add

## sonar demo
- add in jenkinsfile and create another job:
  // stage for sonarqube
  `stage('Checkstyle Analysis'){
  steps{
  sh 'mvn checkstyle:checkstyle'
  }
  }`
- scan code with sonar and apply result in sonar server through pipeline as code
- add to pipeline as code (create another job) : 

- there is a link in jenkins pipeline to sonar server (check the analysis result)

## sonar quality gate
in jenkins-sg allow connexion from sonar-sg on 8080
- go to sonar server 
  - Quality gate -> create our own quality gate
  - name : vprofile-QG
  - Unlock editing
  - add condition : on overall code
  - choose "Bugs" in list
  - greater than 10
- select the created quality gate
  - vprofile project setting -> select "vprofile-QG" -> save
- send info to jenkins using webhooks
  - project -settings -> webhooks -> create
  - name : jenkins-ci-webhook
  - url : http://jenkinsprivateip:8080/sonarqube-webhook 
- got jenkins ec2 SG : 8080 allow from Sonar SG
- add to jenkins :
  `stage("Quality Gate") {
  steps {
  timeout(time: 1, unit: 'HOURS') {
  // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
  // true = set pipeline to UNSTABLE, false = don't
  waitForQualityGate abortPipeline: true
  }
  }
  }`
- result failed (10 is smaller)
## upload artifact to nexus
- nexus : is a software repository, runs on java
- support variety of repo : maven, apt, docker, ruby gems etc...
- jenkins with nexus 
  - code commit -> github ->fetch build on jenkins  -> upload to nexus repository
- connect to : nexus-serverIP:8081
- setting -> repositories -> create maven2(hosted)
  - name : vprofile-repo
  - create
- go to jenkins ans setups nexus credential
  - manage jenkins -> manage credentials -> System -> Global credentials  -> add credentials
  - username : admin
  - pwd : nexuspwd
  - ID : nexuslogin
- timestamp value
  - manage jenkins -> configure system -> enable build timestamp 
  - pattern : yy-MM-dd_HH-mm
  - save
- write pipeline code
  ` stage("UploadArtifact"){
  steps{
  nexusArtifactUploader(
  nexusVersion: 'nexus3',
  protocol: 'http',
  nexusUrl: 'nexusPrivateip:8081',
  groupId: 'QA',
  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
  repository: 'vprofile-repo',
  credentialsId: 'nexuslogin',
  artifacts: [
  [artifactId: 'vproapp',
  classifier: '',
  file: 'target/vprofile-v2.war',
  type: 'war']
  ]
  )
  }
  }`
- if no more space : in jenkins server, cd /var/lib/jenkins/workspace and  rm -rf * 
  - and restart jenkins service
- test pipeline and check the artifact in nexus

# automate pipeline notification
- create slack account https://slack.com/get-started#landing
  - create workspace : vprofilecicd
  - create channel : jenkinscicd
- create token in slack
  - go to https://cprofilecicd.slack.com/marketplace
  - search jenkins CI -> add to slack -> choose jenkinscicd channel -> add ..
  - copy token in step 3 : O74LMlz9yyDysiNqy5enNrlB
  - save

  - plugins section -> search "Slack notification" 
  - add token : manage jenkins -> configuration -> slack 
    - workspace : vprofilecicd
    - add -> jenkins -> secret text -> 
    - secret : token
    - id : slacktoken
    - add
    - select token
    - channel : #jenkinscicd
    - test connection
    - save
  - add post build steps in pipeline
  ` post {
    always {
    echo 'Slack Notifications.'
    slackSend channel: '#jenkinscicd',
    color: COLOR_MAP[currentBuild.currentResult],
    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
    }
    }
  `
  - add color var:
  `def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    ]`
  - test success
  - test fail


# Build docker img and add to AMAZON ECR instead of nexus
- install docker engine in jenkins
  - ssh in jenkins ec2
  - sudo -i
  - sudo apt update, sudo apt install awscli -y
  - install docker engine : https://docs.docker.com/engine/install/ubuntu/#installation-methods
  - systemctl status docker
  - give right to jenkins user , add to docker group : usermod -a -G docker jenkins
  - checks : id jenkins 
  - reboot

- Iam user with ecr permission
  - user name : jenkins 
  - no access to aws console
  - permission policy : amazonEC2ContainerRegistryFllAccess,amazonECS_FullAcces
  - create
  - create access key
  
- ecr repo on aws
  - ECR
  - Repositories -> create
  - name : vprofileappimg
  - create
- plugin docker pipeline
  - manage jenkins -> plugin -> searc 
  - "docker pipeline" plugin
  - install "Amazon ECR" plugin
  - install "Aws sdk" plugin
  - install "CloudBees Docker Build and publish" plugin
- store aws cred in jenkins
  - manage jenkins -> manage credentials -> global -> add credentials -> 
  - id : awscreds
  - access key : ***
  - secret key :  **
- build pipeline: PAAC_CI_Docker_ECR.txt

# ci cd : deploy our container image
- setups ecs cluster on aws
  - clusters 
  - create cluster -> name : vprofile 
  - select Amazon fargate 
  - Monitoring -> active use container in sights
  - add tags
  - create
  
- create task definition : config
  - name : vprofileapptask
  - memory : 2gb
  - container -1 : name : vproapp, image uri : copy uri from ecr
  - port :8080
  - add tags 
  - create
-add cloudWatchLogsFullAcces permission to task definition role

- add service to cluster
  - got to cluster -> service -> create
  - Family : choose vprofileapptask
  - version : latest
  - name : vprofileappsvc
  - uncheck in deployment failure:  "use the amazon ecs deployment circuit breaker"
  - create SG : name = vproappecselb-sg, http 80 anywhere, custom tcp 8080 anywhere
  - LB : app lb, name : vproappelbecs
  - TG : name : vproappecstg, health check path: /login 
  - create
- test : copy (cprofileappsvc ->networking -> dns name) to browser (without backend service yet)


- install plugins 
  - manage jenkins -> manage pluggins - > install "pipeline:aws steps"
- create job with with PAAC_CICD_Docker_ECR_ECS+(1).txt script
  the script will create a new container and fetch the latest image and run it and delete the old container

# job triggers 
- git web hook : whenever there is a commit
  - configured on GitHub
- poll SCM : check for commit (intervall : every 5 minutes for exemple)
  - configured in jenkins
- scheduled jobs : precise time
  - configured in jenkins
- remote triggers : 
  - triggered from anywhere
- build after other projects are build

# job master slave 
- til now, we run job on master
- when you run job from another machine (those machine are slave)
## use case
- load distribution
- cross-platform 
- software testing
- 