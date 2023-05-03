### jenkis
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
#### Jenkins pipeline
