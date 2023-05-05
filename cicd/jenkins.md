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
#### create a pipeline
    -> jenkins -> + -> name it -> check build pipeline view option -> ok
    -> slelect yout new job in "select initial job" - check show options -> apply ok
    - create another project that will chained after new job : to copy artifact (war file)
        -> manage jenkins -> install "copy artifact pluggin"
        -> manage jenkins ->install "deploy to container pluggins"
        -> new item -> job 2 -> add buid step -> copy artifact from another project -> project nme = new job-> 
        -> which build = lates successful build -> add post build action -> deploy war/ear to a container
        -> war/ear files = **/* -> context path = http://tomcat.noodle.tetra:8080/ (do not forget the dns http mapping adress in jenkins server for tomcat)
            -> we casn use this contex path in ci cd = tetra_noodle_app_$BUILD_NUMBER
            -> containers -> tomcat7.x -> enter credential -> tomcat url -> http://tomcat.noodle.tetra:8080
        -> apply save 
    - chaine job in pipeline
        -> in job 2 -> build triggers section -> check build after other projects are build -> select new job 
                or
        -> in new job -> configure -> add post build action -> select "Build Other project" -> job 2 ->
    - apply save
#### start pipeline
    go to pipeline -> run -> enable auto refresh
    
## Relational db with jenkins and Sqitch
    Sqitch is a db change management framework
    first allow connection from jenkins to postegresql :
        - update client authentication : vi /var/lib/pgsql/9.4/data/pg_hba.conf
        - restart db : systemctl restart postgresql
    second, enable connection from our jenkins machine
        - in this course we only desable firewall : vi /etc/sysconf/selinux
            SELINUX = desabled
            SELINUXtYPE = targeted
    Install PHPPgAdmin
#### create new job (db_source)
    source code = git -> https://..../db_test.git -> delete workspace... -> build triggers -> poll scm (when there is a push in the repo, job run) -> 
    Schedule = H/5 * * * * ( check every 5 minutes if there is a commit) -> buid step -> execute shell -> 
    command =   
        cd $workspace
        sqitch deploy db:pg:deploy:deploy@792.168.0.103:5432/tetranoodle
        (pg : postgresql, deploy : user credential , tetranoodle : db name)
    -> apply save
#### create pipeline
    -> install without restart : build pipeline pluggin
    + -> view name = db_tetranoodle -> initial source = db_source -> check show options -> apply ok
#### create another job (db_verify)
    -> source code = git -> https://..../db_test.git -> build step -> execute shell -> 
    command =   
        cd $workspace
        sqitch verify db:pg:deploy:deploy@792.168.0.103:5432/tetranoodle
        (pg : postgresql, deploy : user credential , tetranoodle : db name)
    -> save
#### configure pipeline
    got to db_source -> add post build action -> select db_verify