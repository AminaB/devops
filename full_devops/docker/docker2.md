# docker 
isolate services without running many VMs, Vms are Expensive (OS need license, take time to boot,..).
docker manage containers with docker engine
## containers
package software to ship and deploy easily.
containers offer isolation not virtualization, container virtualizes the os and VMs virtualize the hardware.
isolate process or service running in a directory
- directory : namespace, cgroup
- a directory with ip addr to connect
- all container share the same os kernel ( kernel provide the right resource for process)

## docker setup 
https://docs.docker.com/engine/install/ubuntu/
- systemctl status docker
- to avoid sudo when running docker command, add user to docker group : usermod -aG docker aminatou
## docker registry : dockerhub
- image repository
- other registry : aws ECR, nexus3 ...

## commands 
we cannot remove image if container is running from that image
- docker pull imageName
- docker images
- docker run imageName
- docker run --name myweb  -d -p 8080:80 imageName
  - --name : give name to image
  - -d : run in background
  - -p : port
  - 8080 : mapped port outside the container
  - 80 : port inside (it is like port inside private network, we cannot access it)
- docker stop containeIdOrname
- docker start containeIdOrname
- docker ps
- ps -ef : to see process
- cd /var/lib/docker/containers : to see directory where containers are running
- docker exec containeNnameOrId ls
  - run ls command inside container
- docker run -it imageName /bin/bash
- docker exec -it containeNnameOrId /bin/bash
  - run shell inside container
  - -it attach to that command : to be in the container
  - some of linux command won't be there, but we can install them if we like : apt update,...(not recommended)
  - exit (process is dead now)
- docker rmi imagename : to remove image
- docker stop containerId
- docker rm containerId

### docker logs
- docker pull nginx 
- docker inspect nginx : analyse result
  - Entrypoint : .sh file which will run when you run the image
  - cmd : and next run command in cmd section
- docker logs containernameOrid : we can run this command when we run image with -d option to see logs output, or for troubleshooting
  - ex : docker run -d -P  mysql:5.7
  - process will exit automatically
  - in the log we will see that password needed in the command.
  - docker run -d -P -e MySQL_ROOT_PASSWORD=mypass  mysql:5.7

## docker volume

## docker build
-we can build image from another with some customisation
## docker-compose
- run multi containers together 
- make services talk to each other
## multi stage docker file 
- to avoid heavy container with many KB.