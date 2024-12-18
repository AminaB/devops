# Containerize vprofile project 
In this setup I containerized a project using Dockerfile and docker compose
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
- aminatoubarry/vprofiledb
- aminatoubarry/vprofileapp

## Create dockerfiles and compose
- app
- web
- db
- docker compose file
## build and run 
- docker compose build
- docker images
- docker compose up -d

***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/docker/containerization/images.png)

***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/docker/containerization/containers.png)
## test 
- localhost:80
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/docker/containerization/result.png)
## push images to docker hub
- docker login
- docker push aminatoubarry/vprofileweb
- docker push aminatoubarry/vprofileapp
- docker push aminatoubarry/vprofiledb
***
![alt text](https://github.com/AminaB/devops/blob/master/full_devops/docker/containerization/docker-hub-images.png)
## clean up
- docker compose down
- docker system prune -a
- docker volume prune -f
- docker volume ls -q | xargs docker volume rm (delete used volumes)
