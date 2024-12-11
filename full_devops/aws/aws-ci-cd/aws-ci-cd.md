# Tools
- Elastic Beanstalk
- RDS
- BitBucket Repository
- Migrate from github to Bitbucket
- Code build
- code pipeline (bitbucket, code build, elastic beanstalk)

## Beanstalk
- setup in aws-paas-saas.md 
## RDS
- setup in aws-paas-saas.md 
## authorize beanstalk to access to db
- copy sg id of beanstalk( not the elb sg)
- and add the copied sg in db sg inbound rules (port 3306)

## test connection between beanstalk and rds
- ssh to any of instances created by beanstalk
- sudo -i
- dnf search mysql
- dnf install mariadb105 -y
- mysql -h rds-endpoints.com -u admin -pPWD accounts
- show databases
- exit
- wget  https://github.com/hkhcoder/vprofile-project/blob/aws-ci/src/main/resources/db_backup.sql
- log again to add data : mysql -h rds-endpoints.com -u admin -pPWD accounts < db_backup.sql
- test : mysql -h rds-endpoints.com -u admin -pPWD accounts
- show tables : role, user, user_role

## bitbucket repo
- generate ssh-key: ssh-keygen
- create repo in bitbucket 
- migrate GitHub vprofile project to bitbucket
- git clone git@github.com:hkhcoder/vprofile-project.git
- git checkout aws-ci
- cat .git/config
- git remote rm origin
- cat .git/config
- git remote add origin git@bitbucket.org:aminabarry/vproapp.git
- git push origin --all

## aws code build
- first, create a s3 bucket to store artifact
- got to aws code build (like cloud version of jenkins)
  - name : vproappbuild
  - select : bitbucket
  - connect using Oauth
  - click : connect to bitbucket
  - select repo
  - branch : aws-ci
  - select ubuntu
  - Build Spec :
    - insert build commands 
    - edit and add content of buildspec.yml (change username and pwd)
  - artifacts :
    - select amazon s3
    - select bucket name
  - in cloudwatch log
    - group name : vprofileproject
    - stream name : vprobuild
  - create build project
  - start build to test
## create code pipeline
- name : vprocicdpipeline
- source : bitbucket
- connect to bitbucket
- install new app
- grant access for workspace
- select repository
- connect
- select branch
- build provider
- select the codebuild
- Deploy option -> select aws beanstalk project
- create pipeline

- test in browser
- modify code commit push and test again