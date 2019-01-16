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
  | |- docker-compose.yml
  
````````

@Ref_1 from [spring.io tutorial](https://spring.io/guides/gs/spring-boot-docker/)
>We added a VOLUME pointing to "/tmp" because that is where a Spring Boot application creates working directories for Tomcat >by default. The effect is to create a temporary file on your host under "/var/lib/docker" and link it to the container under >"/tmp". This step is optional for the simple app that we wrote here, but can be necessary for other Spring Boot applications >if they need to actually write in the filesystem.

@Ref_2 from [docker compose doc](https://docs.docker.com/compose/compose-file/)

At this point , when you run 
`````````
docker-compose up --build 
`````````
> --build indicate the image will be rebuild on compose

you should be able to start auth and api service on your local computer.

# 2. Remote Deploy using docker-machine

Assume you would like to deploy to a remote linux instance.

> For instance hosted on Digital Ocean or AWS , please follow the following guides instead.
> Digital Ocean : https://docs.docker.com/machine/examples/ocean/
> AWS : https://docs.docker.com/machine/examples/aws/

You have to setting up a ssh-key between your development env and intergration env.

In Your Development Env

Run:
```````
ssh-keygen -t rsa  
```````

then

`````
ssh-copy-id {username}@{host}   //host is your integration env
`````

This action copy your ssh-key to your integration env . 
Which allow docker on your dev env can run ssh command without entering credentials

Also, You need to allow docker to run sudo command while configurating the docker machine.
(You should only enable this while we configure Docker Machine.)

In Your Integration env

Run: 

`````
sudo vim /etc/sudoers  // I prefer using vim , any other editor is ok 
`````

then edit the sudoers file : 

`````
{username}  ALL=(ALL) NOPASSWD:ALL . //{username} is your ssh username
`````

After login and logout , You should be able to connect to ssh without password and sudo without password.

# Make sure the Docker port is open

>Docker Machine will SSH to the remote machine to configure the Docker engine. The Docker client will then connect on TCP port 2376. You'll need to make sure this port is open on your firewall. 

# 2.1 Create Docker Machine 

In Your Dev env , create docker machine with this command.

For Mac env : 

>docker-machine create --driver generic --generic-ip-address={ip-address} --generic-ssh-key "~/.ssh/id_rsa" --generic-ssh-user={remote-ssh-username} {remote-docker-host} 

For Window env : 

>docker-machine create --driver generic --generic-ip-address={ip-address} --generic-ssh-key "%localappdata%/lxss/home/{bash-username}/.ssh/id_rsa" --generic-ssh-user={remote-ssh-username} {remote-docker-host}  

{ip-address} stand for the ip of the integration host.
{remote-ssh-username} stand for the username of ssh.
{remote-docker-host} is just the name of the docker machine , you can name it whatever you want.
`````
// Make sure the docker daemon is running on the integration env.
// if not , run this command on the integration env.

//Create the docker group.
sudo groupadd docker
//Add the user to the docker group.
sudo usermod -aG docker $(whoami)
//Log out and log back in to ensure docker runs with correct permissions.
//Start docker.
sudo service docker start

`````
@Ref [Stackoverflow ans](https://stackoverflow.com/questions/21871479/docker-cant-connect-to-docker-daemon)

At this point , You should created a docker machine which connected to integration env.

To activate this machine setting on your dev env

Run:
````````
eval $(docker-machine env {your-docker-machine-name})
````````
Now the docker running on your machine should be the remote host.

Run:
````````
docker info
````````
To check whether the docker info is the same on the remote computer

Now , as we connected to the remote docker machine.

We can deploy to it using docker-compose ! :tada: :tada:

Go to the docker directory , we create before 

recap : 
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
  | |- docker-compose.yml
  
````````

run :

`````
docker-compose config 
`````
to check whether our docker-compose.yml is typed correctly

now run: 

`````
docker-compose up --build -d
`````

> -d stand for detach , which allow docker-container to run in background


To deactive the detach from the remote docker machine:

Run:
````````
eval $(docker-machine env -u)
````````

# Congratulation!:confetti_ball:

Now you deploy painlessly using docker-compose and docker-machine !

P.S spent days to research this way to deploy , please let me know if this method is not a good pratice.





