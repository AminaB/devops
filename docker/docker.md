container : is a runtime instance of image
image : is a executable package of software tha content everything we needed to run an application code, 
runtime, system tools, libraries and settings.
docker.md

## create image 
### create a workflow : 
		Ex : for python app
			- install python
			- install flask
			- run web server using flask (you need source code app.py)
				FLASK_APP=app.py flask run --host=0.0.0.0
### dockerise the app
	- create a Dockerfile
		FROM ubuntu

		RUN apt-get update
		RUN apt-get install -y python python-pip
		RUN pip install flask

		COPY app.py /opt/app.py

		ENTRYPOINT FLASK_APP=app.py flask run --host=0.0.0.0

	- docker build . or docker build . -t mys-simple-web-app
    - docker build -t webapp-color:lite .

### run images
	- docker run my-simple-web-app
    - docker run -p 8282:8080 webapp-color
### we can push it to dockerhub
	- docker login
	- docker build . -t yourrepositoryNAME/mys-simple-web-app
	- docker push Username/mys-simple-web-app

## Env variables
	
	- export APP_COLOR=blue; python app.py
	- in python file color=os.environ.get('APP_COLOR')
	- docker run -e APP_COLOR=blue my-simple-web-app
	- docker inspect blissful_hopper

## commands
	CMD sleep 5
	- docker run ubuntu-sleeper sleep 10

## entrypoints
	ENTRYPOINT['sleep']
	- docker run ubuntu-sleeper 10

## Docker Compose
	version 2
	services:
		redis :
			image : redis
			networks: 
				- backend

		db :
			image : postgres:9.4
			networks: 
				- backend
		vote :
			image : vote-app
			networks: 
				- backend
				- frontend
	networks:
		-backend
		-frontend


