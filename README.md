# Docker-compose-deployment-using-docker-machine
Seamless micro-service deployment using docker-compose and docker machine

# Prerequisite

From [docker compose page](https://docs.docker.com/compose/install/)

``````````
Docker Compose relies on Docker Engine for any meaningful work, so make sure you have Docker Engine installed either locally or remote, depending on your setup.

Docker for Mac and Docker Toolbox already include Compose along with other Docker apps, so Mac users do not need to install Compose separately.

``````````


# 1. Create Your Awesome Service

Let say you have a project with the following directory
````````
  root
  |- auth
  | |- build
  |   |- lib
  |     |-xx.jar
  |- api
  | |- build
  |   |- lib
  |     |-xx.jar
  
````````
# 1.2 Create Docker directory

First, you have to create a docker directory 

````````
  root
  |- auth
  | |- build
  |   |- lib
  |     |-xx.jar
  |- api
  | |- build
  |   |- lib
  |     |-xx.jar
  |- docker
  
````````

Move the xx.jar built from gradle 

````````
  root
  |- auth
  | |- build
  |   |- lib
  |     |- xx-auth.jar
  |- api
  | |- build
  |   |- lib
  |     |- xx-api.jar
  |- docker
  | |-xx-auth.jar
  | |-xx-api.jar
  
````````
# 1.3 Create Dockerfile

Then create docker file for every build that you would like to encapsulate as a container.

E.g. Create a docker file for api 

Dockerfile.auth

```````
FROM openjdk:8-jdk-alpine
EXPOSE 8080
VOLUME /tmp
ADD xx-auth.jar auth.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/auth.jar"]
```````

Dockerfile.api
```````
FROM openjdk:8-jdk-alpine
EXPOSE 8081
VOLUME /tmp
ADD xx-api.jar api.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/api.jar"]
```````


>  -D stand for the parameter for start script

Now the directory should look something like this.

````````
  root
  |- auth
  | |- build
  |   |- lib
  |     |- xx-auth.jar
  |- api
  | |- build
  |   |- lib
  |     |- xx-api.jar
  |- docker
  | |- xx-auth.jar
  | |- xx-api.jar
  | |- Dockerfile.auth
  | |- Dockerfile.api
  
````````

# 1.4 Create docker-compose.yml

docker-compose.yml

```````````
version: '3' #using version 3 of docker-compose
services:
  auth:
    container_name: auth
    build:
      context: .
      dockerfile: ./Dockerfile.auth
    image: auth:latest
    expose:
      - 8080 #EXPOSE_HOST_PORT
    ports:
      - 8080:8080 #HOST_PORT:CONTAINER_PORT
    networks:
      - spring-cloud-network
    volumes:
      - /tmp # @Ref_1 
    restart: on-failure #restart if container failure
  api:
    container_name: api
    build:
      context: .
      dockerfile: ./Dockerfile.core
    image: api:latest
    expose:
      - 8081
    ports:
      - 8081:8081
    networks:
      - spring-cloud-network
    volumes:
      - /tmp
    restart: on-failure
networks:  #custom network see docker-compose doc for more detail @Ref_2
  spring-cloud-network:
    driver: bridge
```````````

@Ref_1 from [spring.io tutorial](https://spring.io/guides/gs/spring-boot-docker/)
>We added a VOLUME pointing to "/tmp" because that is where a Spring Boot application creates working directories for Tomcat >by default. The effect is to create a temporary file on your host under "/var/lib/docker" and link it to the container under >"/tmp". This step is optional for the simple app that we wrote here, but can be necessary for other Spring Boot applications >if they need to actually write in the filesystem.

@Ref_2 from [docker compose doc](https://docs.docker.com/compose/compose-file/)

At this point , when you run 
`````````
docker-compose up --build 
`````````
you should be able to start auth and api service on your local computer.

# 2. Remote Deploy using docker-machine

TBC






