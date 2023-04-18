# Devops notes
    Unify dev & ops, it is a culture, movement, philosophy
    Shorter deployment cycles 
    By the Practiotionners, For the Practitionners 
### Back in time :
    Waterfall : Long planning phases
    Agility : Small, frequent cycles. We focus on doing work in samll pieces & releasing features very frequently
    Devops and agile often go hand and hand 
    It can be difficult to do agile without Devops & it can be difficult to do devops without agile 
#### Traditional Silos
    Dev, QA and Ops work in silos.

### Devops Goals and culture
    for the dev team the KPi = speed, and for ops team the KPI = stability
    in devops dev and ops are one team, they work together
    Goals:
        - Delivery features quickly
        - Maintain stability (less fealures and immediately recovery them)
        - Goeal is spead
        - Shared Goals
        - Time to market
#### In devops : when dev commit a code, there is :
    Automated builds
    Automated integration
    Automated testing
    Automated deployement
    Parallel Processing
    Highly Efficent Teams
    Robust Automation process (bug always fixed before users notice them)

### Devops Concepts & practices
#### Build automation
    Build automation is independent of an IDE
    So build code is not dependent on specific individuals, it has to be portable
    for that we use Continuous integration (CI).
- CI 
    Reduce Merge conflicts, frequently merging the code changes, automatic notifications to the code owners, etc...
    its execute an automate gradle or maven build
- Ci tools : Jenkins, Travis CI is good for GitHub code, TeamCity, CircleCi, Gitlab CI, etc...
#### Tools for build automation
    - Ant, maven gradle, for java
    - npm grand, culp for js
    - Packer Hashicorp for images and containers,... 
#### Continuous deployment
    it is a practice where every change that is passing through all satges of production pipeline is released to customers
    it accelerate the feedback loop with customers
#### Continuous delivery
        https://agilemanifesto.org/principles.html
        it is the build process
        the code is in deployable state



#### Iac
    It is manging and provisionning infrastrucyures using code
    use code to deploy insfrastructure
    time to market
    we can build complex infrastructures repeatedly
    avoid human errors
    consistent deployment
    reusability of code
    sacbility of code
    self doccumentation (save every change)
 #### configuration management
    doing a change in maintainable way
    Minimize the configuration drift (accumulation of all small changes over in a period)
    Drift happens when changes are rolled out.
    Identify the state of infrastructure
#### Tools for configuration management
    - Ansible : use declarative configuration (YAML), it can act like a controle server. Doesn't need a agent
    - Puppet : use declarative configuration (YAML), manage state through UI. It use DSL.
    - CHEF : procedural configuration, instead of declaring state of that machine, it use script. It use agent server Model. It use DSL.
    - SALT : use declarative configuration (YAML).
#### IAAC
    - iac is provisioning infrastructure and configuration management is controlling the configuration/deployment (they goes hand in hand)
#### Tools Iaac :
    Terraform
#### Monitoring devops
    we collect data and put them in presentable format (chart, graph,..).
    it's help to quickly repond to problem
    Realtime monitoring and notifications
    Assists in troubleshooting
#### Tools Monitoring
    SENSU, New Relic, SAAS, APPDYNAMICS
    - Infrastructure Monitoribg : memory usage, CPU usage, ...
    - Application Performance Monitoring : stability of application,..
    ELK STACK 
    - Aggreagation and analytics : What we do with data after collecting it. it is going to be the graph, chart, notification,...
    
#### Micro services
    its break an application in small collection
    they are monolithic  architecture
    each micro service has his own indivudualy executable portion
    micro services interaact with each others
#### Virtualization 
    it make use of hypervisor. hypervisor is a program that runs on the bare physical metal and allows to manage virtual machines
#### Tools Virtualization
    VMWARE, etc...
### Containerization
    instead of Vm that contains an entire  and virtual version of the hardware, the containerization contains the bare minimum nedded by software
    The container only simulate the OS. they are easier to manage.
#### Tools containerization
    - Docker 
#### Orchestration 
    - Docker swarm : explicitly for orchestration of docker container
    - Kubernetes : open source, it is able to manage containerize across multiple host.
    - ZooKeeper : 
    - terraform : terraform combines orchestration with IAAS

## devops and the cloud
    they are very close but not same, devops is a culture, collab between dev and ops team. 
    DevOps can hapen inside or outside the cloud.
    Cloud is Remote servers on the internet that offer service.
    Practices of Devops are very useful in the world of cloud.
### Cloud architectures
    - IAAS : the cloud provider provides us with bare operating system, 
    and we are reponsable for being able to loggin into those machines, add do addintional config.
    Ex : AWS EC2 instances, Microst azure Virtual machines, ...
    - PAAS :  it provide us with standardized os , and we deploy our data layer, apllications on tap for that
    Ex : AWS Elastic Beanstalk, Heroku.. 
    - IAAS : we do not code applications, we consume it that the service provider have given
    Ex : GMAIL, ...
    - FAAS : everythings is abstracted, we only deploy samll single-purpose functions of code. 
    It is serverless, the server is running in cloud provider's date center.
    Ex : AWS Lambda, google cloud functions,... 