### jenkins
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
