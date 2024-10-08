# micro services
## monolithic
- everything in one application
- so everything must be in same programming language
- difficult to maintain
- difficult to move or slow while moving

## micro service
- separate application for different functionalities
- we can use different programming language for each service, they work together but are different
- interact through api gateway (proxy). we can use NGINX for the gateway service
- we need multiple servers to run each microservice (isolation)
- for resolving isolation problem, we can containerize them


# testing emart app
- run vagrant vm
- vagrant up 
- vagrant ssh 
- sudo -i
- cd emartapp
- docker compose up -d
- 