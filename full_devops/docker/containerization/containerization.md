# Containerize vprofile project 

## tools and version 
- MySQL (Database SVC)        => 8.0.33
- Memcache (DB Caching SVC)   => 1.6 
- RabbitMQ (Broker/Queue SVC) => 4.0 
- JDK                         => JDK 21 
- MAVEN                       => Maven 3.9.9 
- Tomcat (Application SVC)    => 10, jdk21  
- Nginx (Web SVC)             => 1.27 


- mysql:8.0.33
- memcache:latest 
- rabbitmq:latest 
- maven:3.9.9-eclipse-temurin-21-jammy 
- tomcat:10-jdk21 
- nginx:latest

## workflow 
- search right versions of images
- docker file for services (nginx, tomcat, mysql), pull and customize, 3 docker files
- docker build to test
- docker compose with all images
- push customize docker images to our dockerhub account

## Create Repo in dockerHub
- aminatoubarry/vprofileweb
- aminatoubarry/vprofildb
- aminatoubarry/vprofileapp

## Create dockerfiles
- app
- web
- db
