# jenkins
install jenkins and artifactory in 2 different machine
#### available env variable in jenkins
    https://www.jenkins.io/doc/book/pipeline/jenkinsfile/
    JAVA_HOME
    
        If your job is configured to use a specific JDK, this variable is set to the JAVA_HOME of the specified JDK. When this variable is set, PATH is also updated to include the bin subdirectory of JAVA_HOME
    JENKINS_URL
    
        Full URL of Jenkins, such as https://example.com:port/jenkins/ (NOTE: only available if Jenkins URL set in "System Configuration")
    JOB_NAME
    
        Name of the project of this build, such as "foo" or "foo/bar".
    NODE_NAME
    
        The name of the node the current build is running on. Set to 'master' for the Jenkins controller.
    WORKSPACE
    ....
## CI CD Pipeline with jenkins gradle and artifactory
#### Jenkins pipeline : chaining job execution 
install plugins pipeline
    - Manage ling -> Manage plugins -> available -> build pipeline plugin -> install without restart
setup job and pipeline plugin
    - click + icon -> name of pipeline -> check build pipeline view ->ok -> Title -> select initial job -> select all show options
    -> apply

#### Artifactory : repository manager
    it is a java webapp which has been deployed as a war file into apache
    - first install java
    - install Artifactory
    - service atifactory start
    - go to : server.noodle.tetra:8081/artifactory
    - sudo yum install tomcat-webapps tomcat-admin-webapps
    - Update JAVA_OPTS in /usr/share/tomcat/conf/tomcat.conf
    - edit user : vi /usr/share/tomcat/tomcat-users.xml : ucommmnet <role... and admin user
    - systemctl start tomcat
    - go to server.noodle.tetra:8080
    - go back server.noodle.tetra:8081/artifactory
    - login
#### connect artifactory to jenkins
    - install artifactory pluggins to jenkins
    - add artifactory host in jenkins OS  vi /etc/hosts  (@ip art.noodle.tetra) (@ip https://art.noodle.tetra/)
    - Manage jenkins -> configure system -> artifactory section -> add -> server id=art.noodle.tetra -> url =https://art.noodle.tetra:8081/artifactory 
        -> username -> password -> test connection
#### configure job 
    - configure job
    - check gradle-artifactory integration -> artifactory deployement server= https://art.noodle.tetra:8081/artifactory
        -> refresh -> and select one repository in the list -> enable option publish artofacts to artifactory 
        -> accept default for others options
    - configure grdalle version or use default
    - build section
        -> add build step -> invoke graddle script ->  use graddle executable -> make gradle executable -> apply
        -> build 
## CI CD Pipeline with jenkins maven
    - maven describe how our project will build and dependencies (in pom.xml)
#### war vs jar
    jar content librairies, ressources, properties files, war content web application (java files, jsp, ..) that can be deployed on a server 
#### in jenkins we create a new job 
    -> free style project
    - check discard old build ( it clean a build history)
    - git repository in jenkins job
    - build environment section
        -> check delete workspace before build starts (make sure previous buils do not affect the current build)
    - add build step
        3 options 
        -> execute shell ( we use this option)
            mvn package (do not forgot to configure mavene installation in manage jenkins section)
            
        -> invoke top level maven target
        -> Invoke Artifactory maven 3
    - add post buils action (it will use in pipeline)
        -> archive the artifacts option -> files to archive = proejectname/target/*.war
    - apply save


