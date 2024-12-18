# Simple Devops Project 

    
    github, kuberntes, github, aws, docker jenkins, maven, ansible 
## create EC2 instance for jenkins server and active port 8080, and install
------------------------------------
link :
https://www.valaxytech.com/
https://github.com/yankils/Simple-DevOps-Project
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
-----------------------------------------------
    add dockerhost to jenkins server :
        install "publish over ssh" pluggin in jenkins
        Manage jenkins -> configure system -> got o publish over ssh section -> add -> server name = dockerhost -> hostname =@ip ->
        -> username = dockeradmin -> pwd=.. -> 
----------------------------------------
    create job to pull code from github, build with maven, and copy the artefact onto the dockker host
        job name = buildAnDdeployOntoContainer -> compy from = buildanddeploy -> ok -> delete post build actions section -> 
        -> add post build actions -> send build artefact over ssh -> ssh server = dockerhost -> source files = webapp/target/*.war->
        -> remove prefix = webapp/target  -> apply save *
------------------------------------------------
    deployment process with dockerfile
        create /opt/docker -> chown -R dockeradmin:dockeradmin docker -> mv /root/dockerfile /opt/docker ->
        ->chown -R dockeradmin:dockeradmin /opt/docker.
        
        update job -> remote directory = //opt//docker -> save -> build
        
        update Docker file :
            COPY ./*.war / usr/local/tomcat/webapps
        docker build -t tomcat:v1 .
        docker run -d --name tomcatv1 -p 8086:8080 tomcat:v1
        
        acces to @dokerhostIP:8086/webapp
------------------------------------------------------
    automate build and deployment on docker container

        to delete all containers : docker container prune 
        to delete all images : docker image prune -a

        go to job -> in post build actions -> exec command = cd /opt/docker; docker build -t regapp:v1 .; 
        docker run -d --name registerapp -p 8087:8080 regapp:v1
        run job
#### integrate ansible in ci cd pipeline ( new ec2 instance)
    setup an ec2 instance
        name Ansible-Server
    setup hostname
        ansible-server
        init 6 : to restart
    create ansadmin user
    add user to sudoers file
        visudo
    generate ssh keys
    enable pwd based login
        vi /etc/ssh/sshd_config
        service sshd reload
    install ansible
        yum install ansible
--------------------------------------------------
    integrate docker with ansible
        in docker host : create user ansadmin with admin rigth
            enable pwd based login
        in ansible host
            add hosts file 
                vi /etc/ansible/host
                    add docker ip address
            copy ssh keys
                add pub key in docker host (in authorized directory)
                ssh-copy-id @ipdockerhost
            Test connection
                ansible all -m ping
        
------------------------------------------
    integrate ansible with jenkins : 
        create /opt/docker in ansible-server, add chown ansadmin in this directory
        in jenkins -> manage jenkins -> configure system -> publish over ssh -> add ansible server -> use pwd authentication
    create job -> copy_artifacts-onto_ansible -> user copy from buildAndDeployOnContainer -> build triggers -> uncheck poll SCM
    -> post build actions -> chosse ansible-server -> clean Exec Command -> apply save -> build
------------------------------------------
    build an image and create container on ansible 
        install docker on asnible server
        add ansadmin in docker group : sudo usermod -aG docker ansadmin ( test id ansadmin to see group)
        start docker service
        build image :
            add dockerfile in docker folder
        chmod on the docker file
        docker build -t regapp:v1 .
        docker run -t --name regapp-server -p 8081:8080 regapp:v1
--------------------------------------------------
    ansible playbook to create image and container
        add ansible host ip in /etc/ansible/hosts
        copy ansible pub key in ansible host :  ssh-copy-id @ipansiblehost (or localhost)
        white ansible playbook in docker folder
            vi create_image_regapp.yml
        ansible-playbook create_image_regapp..yml
------------------------------------------------
    copy image on to dockerhub

    for security reason, we can create our own dokcer registry on our local system ( jfrog artifactory, amazon docker registry, ..)
       
     create a dockerhub account
        docker login 
        create tag : docker tag imageid username/regapp:latest
        docker push username/regapp:latest
        
  -----------------------------------------------------
    Jenkins job to build image onto ansible
        edit create_image_regapp.yml
        ansible-playbook create_image_regapp.yml --check
        nsible-playbook reate_image_regapp.yml
        
        give this comman to jenkins server : ansible-playbook create_image_regapp.yml
           update jenkins job (opy_artifacts-onto_ansible )
            -> post build section -> exec command = ansible-playbook /opt/docker/reate_image_regapp.yml -> apply save.
            enable poll scm (to execute every minute)
  ------------------------------------
    create container on dockerhost using ansible 
      ansible-playbook docker_deployment.yml --check
      ansible-playbook docker_deployment.yml
  --------------------------
    delete container before create
      ansible-playbook docker_deployment.yml --check
      ansible-playbook docker_deployment.yml
  ------------------------------------------
    jenkins CI:CD to deploy on container unsing ansible 
      in post build actions -> sleep 10; ansible-playbook /opt/docker/docker_deployment.yml;-> 

#### Kubernetes on AWS : containers management system 
      installation (go to install folder, there is the instructions)
        create ec2 instance (choose t2.micro)
        create cluster
--------------------------------------------
      create cluster
        cluster name = valaxy
        region = us-east-1 (check it in ec2 instace)
        node-type = t2-small
-------------------------------------------------
      create manifest file  (https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
        create file pod.yaml
        create pod : kubectl apply -f pod.yml
             kubectl get all
      create servive manifest file 
          create pod : kubectl apply -f service.yml
             kubectl get all
      kubectl describe service/demo-service
#### integrate kubernetes in ci cd pipeline 
      kubectl delete pod demo-pod
      kubectl delete service demo-service
______________________
      write deployment file 
      regapp-deployment.yml
      write service file for deployment
      regapp-service.yml
--------------------------------------
    create deployment and service file to create and access pod
      kubectl apply -f regapp-deployment.yml (it will create 3 pods)
      kubectl apply -f regapp-service.yml (it will create a Loadbalancer)

 --------------------------------
    integrate kubernetes bootstrap server with ansible
      connect to ansile server
        create ansadmin user, add it to sudo file, enable pawd based authentication. psswd ansadmin, service sshd reload.
        add ansadmin key onto root user: ssh-copy-id root@ipkubernetes ( add password to root user on  kubernetes)
        log with ansadmin 
        create hosts file in /docker
        ssh-copy-id @ipkubernest
---------------------
    create ansible playbook
      kube_deploy.yml
      kube_service.yml
    run playbooks files with hosts in option
-----------------------------------------------
    create jenkins deployment for kubernetes
    go to jenkins page
    new job (Regapp_CD_Job) -> freestyle project -> add post build post action-> name server= ansible-server ->
    -> Exec command =   ansible-playbook -i /opt/docker/hosts /opt/docker/kube_deploy.yml;
    
    -> apply-> build (before build delete the existing node)
-------------------------------------------
    CI Job to create image for kubernetes
    go to jenkins page
    new job (Regapp_CI_Job) -> Copy from = Copy_artifact_ontoAnsible-> ok -> Excec command  = ansible-playbook -i /opt/docker/Create_image_reapp.yml;
  
----------------------------------------
    pipeline ci cd
     go to job (Regapp_CI_Job) -> add post build action -> buil other project -> Regapp_CJ_Job-> trigger only if build is stable
      
  ------------------------------------
    terminate instances
    kubectl deployment.apps/valaxy-regapp
    kubectl service/valaxy-service

    delete cluster 
    kubectl get nodes
    eksctl delete cluster valaxy --region ap-south-1(replace with your region)
