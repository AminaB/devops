# Scenario

- variety services : php jquery wordpress joomla ubuntu ...

# problem
- not comfortable making changes in real severs
- local setup by using VM is complex, time-consuming, not repeatable
# solution 
- automate local setup
- make it repeatable
- code : iac
## tools 
- hypervisor : virtualbox
- automation : vagrant
- cli : git bash
- ide

# Stack of Vprofile project
- nginx : to create the load balancing experience, to route the req to tomcat server
- NFS : shared storage
- Rabbit MQ connected to tomcat : Message broker, queueing agent
- Meme cache : connected to mysql to cache user information
- Mysql db
- 

# Prerequisites
#
- JDK 11
- Maven 3
- MySQL 8

# Technologies
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- Rabbitmq
- ElasticSearch
