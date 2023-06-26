# Docker

**Documentation**
https://docs.docker.com/

## 1. Introduction
Getting Requirements
Docker Install
Container Basics
Image Basics
Docker Networking
Docker Volumes
Docker Compose
Orchestration
Docker Swarm
Kubernetes
Swarm vs. K8s
References Galore

## 2. Setup Docker and tools for your OS

All installations were made using ubuntu

### 2.1 Install Docker
https://docs.docker.com/get-docker/

Types
Docker EE (Enterpraise Edition) Paid
Docker CE (Community Edition) Free

Version
Edge (beta) released monthly
Stable released quarterly

```console
# Install using the convenience script
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh

# Evite ficar escrevendo sudo tova vez que for rodar um comando docker
pi@raspberrypi:~/Public $ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
pi@raspberrypi:~/Public $ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
# Add your user to the Docker group 
sudo usermod -aG docker ${USER}
# Restart the system and try again without sudo
pi@raspberrypi:~ $ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# Test 
docker version
docker ps
```





### 2.2 Install Docker-machine
https://docs.docker.com/machine/install-machine/
```
# Download the Docker Machine binary and extract it to your PATH.
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
 # Test
 docker-machine version
```

### 2.3 Install Docker-compose
https://docs.docker.com/compose/install/
```
# Download the current stable release of Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# Apply executable permissions to the binary:
sudo chmod +x /usr/local/bin/docker-compose
# If the command docker-compose fails after installation, check your path. You can also create a symbolic link to /usr/bin or any other directory in your path.
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# Test
docker-compose --version
```

# docker compose raspberry pi
```console
#!/bin/sh
apt-get update  # To get the latest package lists \
apt-get install libffi-dev libssl-dev -y \
apt install python3-dev \
apt-get install -y python3 python3-pip \
pip3 install docker-compose \
docker-compose --version
```

### 2.4 Install Visual Studio Code
https://code.visualstudio.com/download

Download extension for Docker in Visual Studio Code
```
# Open a folder in the Visual Studio Code
cd directory
code .
```

## 3. Creating and Using Containers

### 3.1 Check version of our docker cli and engine

```console
# Vesion of client and server, verified client can talk to engine
docker version
# Most config values of engine
docker info
# See all commands
docker
# Old Docker command format (still works)
docker <command> (options) 
# New Docker command format 
docker <command><sub-command> (options)
```

### 3.2 Create a Nginx (web server) container

Image vs. Container

An image is the application we want to run. A container is an instance of that image running as a process. You can have many containers running of the same image.

Docker's default image "registry" is called Docker Hub
hub.docker.com

```console
# Create web server

# What happens in the background when we start the command.
docker container run --publish 8080:80 nginx
# Looks for that image locally in image cache, doesn't find anything
# Then Looks in remote image repository from Docker Hub
# Downloaded the latest version (nginx:latest by default) image 'nginx' 
# Creates new container based on that image and prepares to start
# Gives it a virtual IP on a private network inside docker engine
# Opens up port 8080 on the host IP, you can use any port you want on the left, like 8888:80
# Each container can't run in the same port as other containers
# Forwards to port 80 in container, Routes that traffic to the container IP, port 80
# Starts container by using the CMD in the image Dockerfile

# Test on the browser
http://localhost:8080/
```

### 3.3 Manage containers

**Simple Container**

```console
# Run a container in background and return unique id
docker container run --publish 8080:80 --detach nginx
# Run a container with name
docker container run --publish 8080:80 --name webhost nginx
# Listinig all containers
docker container ls
# Stop a container
docker container <CONTAINER ID>
# Run a conteiner always starts a new container
docker container run
# Start a container to start an existing stopped one
docker container start
# See all containers, Exited or Online
docker container ls -a
# See logs of a container, use --help to see all the log options
docker container logs <CONTAINER ID>
docker container logs <NAMES>
# Display the running processes of a container
docker container top <NAME>
# Remove a container that It is not running.
# If you try to remove a container that is accidentally running it will not be removed for security.
docker container rm <CONTAINER ID> 
# Remove multiples containers that its not running
docker container rm <CONTAINER ID> <CONTAINER ID> <CONTAINER ID> 
# Remove a container that is running. You can pause it and use the previous command or force it to be removed.
docker container rm -f <CONTAINER ID> 
# Stop or remove all docker containers
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

**Multiple Containers**
```
# Start mysql with environment variable
docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
# Get password at logs (Run this command two times)
docker container logs db
# GENERATED ROOT PASSWORD: Eng5VeineBoo1catuphahPai6Phiena2

# Start httpd
docker container run -d -p 8080:80 --name webserver httpd

# Start nginx
docker container run -d -p 80:80 --name proxy nginx
docker container ls

# Test
curl localhost
curl localhost:8080
```

## 3.4 Container vs. Virtual Machine

Containers aren't mini-virtual machine
Container, They are just processes, limited to what resources they can acces, exit when process stops.

It is proved
```console
# Start simple database mogo
docker run --name mongo -d mongo
docker ps
# List running processes in specific container
docker top mongo
# Show all running processes in my machine
ps aux
# Find specific process name
ps aux | grep mongo
```

### 3.5 CLI Process Monitoring

```console
# Process list in one container
docker container top <NAME>
# Show metadata about container (startup, config, volumes, networking, etc) in JSON
docker container inspect <NAME>
# Show live performance data for all containers (CPU, MEM, NET)
docker container stats <NAME>
```

### 3.6 Getting a Shell inside containers

No SSH needed, docker cli is great substitute for adding SSH to containers

```console
# Start new container interactively
# -i or --interactive keep session open to receive terminal input
# -t or --tty simulates a real terminal, like what SSH does
docker container run -it --name proxy nginx bash
# Then see all folders inside a container
ls -al
# Run additional command in existing container
# The command exec start a container that already exist
docker container exec -it mysql bash
```

### 3.7 Images

```console
# Get an image
docker pull alpine
# See images
docker image ls
```

### 3.8 Networks

Each container connected to a private virtual network "bridge"
Each virtual network routes through NAT firewal on host IP
All containers on a virtual network can talk to each other without -p
Best practice is to create a new virtual network for each app:
  - network "my_web_app" for mysql and php/apache containers
  - network "my_api" for mongo and nodejs containers
Batteries included, but removable
Make new virtual networks
Attach containers to more then one virtual network (or none)
Skip virtual networks and use host IP (--net=host)
Use different Docker network drivers to gain new abilities
  

IP and Port

```console
# Creating a container and exposing its port to the physical network.
# -p (--publsih) port is always in CHOST:CONTAINER format
docker container run -p 80:80 --name webhost -d nginx
# Quick port check, what ports are open for that container on your network
docker container port webhost
docker container port <container>

# The containers are not on the same network as the computer.
# The IP of container 
docker container inspect --format "{{ .NetworkSettings.IPAddress }}" webhost
172.17.0.2
# The IP of computer 
ifconfig
inet 192.168.76.35
```

Traffic Flow and Firewalls
How Docker networks move packets in and out
If you do not expose the port but connect the container on the same network as another container, they can talk to each other.

Add image bridge/docker0
Docker just have one container listining on physical port 80 for example, this is a limitation.

**CLI Management of Virtual Networks**
```console
# Show networks
# --network bridge Default docker virtual network, which is NAT'ed behid the Host IP
# --network host it gains performance by skipping virtual networks but sacrifices security of container model
# --network none removes eth0 and only leaves you with localhost interface in container
docker network ls

# Create a network (bridge default)
docker network create my_app_net
docker network create --driver

# Create container and conect to a network
docker container run -d --name new_nginx --network my_app_net neginx
docker network inspect my_app_net

# Inspect a network
# You can see the containers attach to this network 
docker network inspect bridge
docker network inspect my_app_net
# "Containers": { ... "Name": "webhost",

# Attach a network to container
docker network connect

# Detach a network from container
docker network disconnect
```

***Docker Networks: Default Security***
Create your apps so frontend/backend sit on same Docker network
Their inter-communication never leaves host
All externally exposed ports closed by default
You must manually expose via -p wich is better default security
This gets even better later with swarm and overlay networks


***DNS an How Container Find Each Other***
Forget IP's Static IP's and using IP's for talking to containers is an anti-pattern. Do your best to avoid it.
Solution: Docker daemon has a built-in DNS server that containers use by default. Use hostname to talk each others.
DNS Default names. Docker defaults the hostname to the container's name, but you can also set aliases.

***DNS Round Robin Test***
```console
docker network create dude
docker container run -d --net dude --net-alias search elasticsearch:2
docker container run -d --net dude --net-alias search elasticsearch:2
docker container ls
docker network ls
docker network inspect dude
docker container run --rm --net dude alpine nslookup search


docker container run --rm --net dude centos curl -s search:9200
{
  "name" : "Emplate",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "P8cdQ6vgQCehPD4nESt6Jg",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}

docker container run --rm --net dude centos curl -s search:9200
{
  "name" : "Luke Cage",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "v8LV1F0PSPW-qPR2TA9AhA",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}


docker container ls
docker container rm -f <CONTAINER ID>
```

## 4. Container Images

## 4.1 What's in an Image

Definition: App binaries and dependencies, metadata about the image data and how to run the image, not a complet Operation System, no Kernel modules (e.g. drivers), smal as one file (your app binary) like a golang static binary, big as Ubuntu distro with apt, and Apache, PHP, and more installed.

Official: "Docker images are the basis of containers. An Image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime. An image typically contains a union of layered filesystems stacked on top of each other. An image does not have state and it never changes." 

https://docs.docker.com/glossary/

### 4.2 The Mighty Hub: Using Docker Hub Registry Images

Basics of Docker hub 

https://hub.docker.com

Search: nginx
We have a lot of results. Whats the official?

Official
https://github.com/docker-library/official-images/tree/master/library
https://hub.docker.com/search?q=&type=image

just nginx
https://hub.docker.com/_/nginx
Docker Official Images
Official build of Nginx.

Not official
user/nginx
inc/nginx

You can filter images to show just official in the site.

```console
# Download the latest
docker pull nginx
# Download specific version with tags: 1.19.3, mainline, 1, 1.19, latest, It's has a lot of tags but, it's just one image ID.
docker pull nginx:1
docker pull nginx:1.11.9
# See the images and size
docker image ls
```

Public images
https://hub.docker.com/r/jwilder/nginx-proxy

Always check if there is a open repository, and check the files.
https://github.com/nginx-proxy/nginx-proxy


### 4.3 Images and Their Layers: Discover the image cache

Images are made up of file system changes and metadata, Each layer is uniquely identified and only stored once on a host, this saves storage space on host and transfer time on push/pull, a container is just a single read/write layer on top of image.

```console
┌───────────────────┐
│    Image Layers   │
│   ┌───────────┐   │
│   │ ENV       │   │
│   └───────────┘   │  
│   ┌───────────┐   │
│   │ apt       │   │
│   └───────────┘   │  
│   ┌───────────┐   │
│   │ Ubuntu    │   │
│   └───────────┘   │  
└───────────────────┘
# Docker image history, show layers of changes made in image
docker history nginx:latest
# Return JSON metadata about the image
docker image inspect nginx
```

### 4.4 Images Tagging and Pushing to docker hub

All about image tags: assign one or more tags to an image
```console
# Help about tag
docker image tag --help
# Default tag is latest if not specified <user>/<repoitory>:<tag>, the official images have no users
docker pull mysql/mysql-server
docker image ls
# Use new tag from exist image
docker image tag nginx pasquati/nginx
# Login on docker hub
docker login
cat .docker/config.json
# Logout, always logout from shared machines or servers when done, to protect your acount
docker logout
# Push an image to docker hub
docker image push pasquati/nginx
# Add new tag
docker image tag pasquati/nginx pasquati/nginx:1.0.0
docker image push pasquati/nginx:1.0.0
# The ID always the same, the tag can be different to point to same ID image.
```

### 4.5 Build Images: The dockerfile basics
 	

A Dockerfile is a text document that contains all the commands you would normally execute manually in order to build a Docker image. Docker can build images automatically by reading the instructions from a Dockerfile.

All docker images we've been using were created from Dockerfiles where you can go look

```console
# Dockerfile
# NOTE: this example is taken from the default Dockerfile for the official nginx Docker Hub Repo
# https://hub.docker.com/_/nginx/
# NOTE: This file is slightly different than the video, because nginx versions have been updated 
#       to match the latest standards from docker hub... but it's doing the same thing as the video
#       describes
FROM debian:stretch-slim
# all images must have a FROM
# usually from a minimal Linux distribution like debian or (even better) alpine
# if you truly want to start with an empty container, use FROM scratch

ENV NGINX_VERSION 1.13.6-1~stretch
ENV NJS_VERSION   1.13.6.0.1.14-1~stretch
# optional environment variable that's used in later lines and set as envvar when container is running
# one reason they were chosen as preferred way to inject key/value is they work everywhre, on every OS and config

RUN apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y gnupg1 \
	&& \
	NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
	found=''; \
	for server in \
		ha.pool.sks-keyservers.net \
		hkp://keyserver.ubuntu.com:80 \
		hkp://p80.pool.sks-keyservers.net:80 \
		pgp.mit.edu \
	; do \
		echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
		apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
	done; \
	test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
	apt-get remove --purge -y gnupg1 && apt-get -y --purge autoremove && rm -rf /var/lib/apt/lists/* \
	&& echo "deb http://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y \
						nginx=${NGINX_VERSION} \
						nginx-module-xslt=${NGINX_VERSION} \
						nginx-module-geoip=${NGINX_VERSION} \
						nginx-module-image-filter=${NGINX_VERSION} \
						nginx-module-njs=${NJS_VERSION} \
						gettext-base \
	&& rm -rf /var/lib/apt/lists/*
# optional commands to run at shell inside container at build time
# this one adds package repo for nginx from nginx.org and installs it

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log
# forward request and error logs to docker log collector

EXPOSE 80 443
# expose these ports on the docker virtual network
# you still need to use -p or -P to open/forward these ports on host

CMD ["nginx", "-g", "daemon off;"]
# required: run this command when container is launched
# only one CMD allowed, so if there are multiple, last one wins
```

### 4.6 Building images: Running docker builds
```console
# Go to the folder that contain Dockerfile
cd dockerfile-saple-1
ls
Dockerfile
# Build an image, this command execute line by line from Dockerfile
docker image build -t customnginx .
docker image ls
# Add port 8080 Change the EXPOSE 80 433 8080
# If we do the build again, the docker will use the cache from the previous steps and it will be much faster, because only the step where the door is exposed will be executed, the others used the cache.
docker image build -t customnginx .
```

### 4.7 Building Images: Extending Official Images

When we take an official image equal to nginx and we want to change the default index.html page.

```console
# Dockerfile
# this shows how we can extend/change an existing official image from Docker Hub

FROM nginx:latest
# highly recommend you always pin versions for anything beyond dev/learn

WORKDIR /usr/share/nginx/html
# change working directory to root of nginx webhost
# using WORKDIR is preferred to using 'RUN cd /some/path'

COPY index.html index.html

# I don't have to specify EXPOSE or CMD because they're in my FROM
```

```console
# Run official page html of nginx
docker container run -p 80:80 --rm nginx
# Test
http://localhost/
# Go to the folder with Dockerfile build 
docker image build -t nginx-with-html .
# Run container from image
docker container run -p 80:80 --rm nginx-with-html
# Test again
http://localhost/
```

### Using Prune to Keep Your Docker System Clean (YouTube)

You can use "prune" commands to clean up images, volumes, build cache, and containers. Examples include:

- docker image prune to clean up just "dangling" images

- docker system prune will clean up everything

- The big one is usually docker image prune -a which will remove all images you're not using. Use docker system df to see space usage.

Remember each one of those commands has options you can learn with --help.

Here's a YouTube video I made about it: https://youtu.be/_4QzP7uwtvI

## 5. Container Lifetime & Persistent Data: Vlumes, Volumes, Volumes

### 5.1 Container Lifetime & Persistent Data
Containers are usually immutable and ephemeral
Immutable infrastructure: only re-deploy containers, never change
This is the ideal scenario, but what about databases, or unique data?
Docker gives us features to ensure these "separation of concerns"
This is known as "persistent data"
Two ways: Volumes and Bind Mounts
Volumes: make spectial location outside of containers union file system
Bind Mounts: link container path to host path

### 5.2 Persistent Data: Data Volumes

You can't clean them up just by removing a container. We manually delete the volume.

https://docs.docker.com/storage/volumes/

```console
# Start mysql database
# For default Dockerfile VOLUME /var/lib/mysql
docker pull mysql
docker image inspect mysql
"Volumes": {
	"/var/lib/mysql": {}
},

# Run a container mysql
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql

# Inspect the container, see Volume and Mounts
# We can also see it up here under mounts. What this is, is this is actually the running container getting
# its own unique location on the host, to store that data, and then it's in the background, mapped or mounted,
# to that location in the container, so that the location in the container actually just thinks it's writing
# to "Destination": "/var/lib/mysql",
# In this case, we can see that the data is actually living in that location on the host "Source": "/var/lib.../_data",
docker container inspect mysql

# There will actually be some databases there.
docker volume ls
docker volume inspect <volume>
"Mountpoint": "/var/lib.../_data",

# If I delete the container, my data will still be there. My data is still safe.
docker container rm mysql
docker volume ls

# Named volumes, friendly way to assign volumes to container
# By default 
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v /var/lib/mysql mysql
# Naming volume
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v <name>:/var/lib/mysql mysql
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
docker volume ls
docker volume inspect mysql-db

# Connect a new container in existing volume
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql

# Create a volume, required to do this before "docker run" to use custom drivers and labels, see production section
docker volume create --help
```

### Tips
Console 
You can just type the first couple of characters and hit Tab to complete it.
Looking through the README.md or Dockerfile of the mysql official image, you could find the database path documented or the VOLUME stanza.

### 5.3 Shell Differences for Path Expansion
In the next lecture, you'll learn how to share files and directories between a host and a Docker container. One of the parts of the command line you'll need to type is the host file path you want to share.
With Docker CLI, you can always use a full file path on any OS, but often you'll see me and others use a "parameter expansion" like $(pwd) which means "print working directory".
Here's the important part. Each shell may do this differently, so here's a cheat sheet for which OS and Shell your using. I'll be using $(pwd) on a Mac, but yours may be different!
This isn't a Docker thing, it's a Shell thing.
For PowerShell use: ${pwd} 
For cmd.exe "Command Prompt use: %cd%
Linux/macOS bash, sh, zsh, and Windows Docker Toolbox Quickstart Terminal use: $(pwd) 
Note, if you have spaces in your path, you'll usually need to quote the whole path in the docker command.

### 5.4 Persistent Data: Bind Mounting
Maps a host file or directory to a container file or directory
Basically just two locations pointing to the same file(s)
Again, skips UFS, and host files overwrite any in container
Which type of persistent data allows you to attach an existing directory on your host to a directory inside of a container?
Bind mount


```console
# Can't use in docker file, must be at container run, on mac/linux
run -v /Users/<user>/stuff:?path/container
# Go to dockerfile-sample-2 its a custom html page to test vs default page of nginx
cd dockerfile-sample-2
# Custom html page test localhost
docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nfinx/html nginx
# Default html page
docker container run -d --name nginx2 -p 8080:80 nginx

# For the files that I edit locally to appear in a container that is already running
cd dockerfile-sample-2
ll
# Create new file
touch testme.txt
# Acces terminal of a container and see the file
docker container exec -it nginx bash
cd /usr/share/ngins/html
ls -al
```

### 5.4 Assignment: Database Upgrades with Named Volumes

Real-world scenario
Database upgrade with containers
Create a postgres container with named volume psql-data using version 9.6.1
Use Docker Hube to learn VOLUME path and version needed to run it
Check logs, stop container
Create a new postgres container with same named volume using 9.6.2
Check logs to validate

### 5.5 Assignment Answers: Database Upgrades with Named Volumes


```console
# Create a postgres container with named volume psql-data using version 9.6.1
docker container run -d --name psql -v psql:/var/lib/postgresql/data postgres:9.6.1

# Check logs, stop container
docker container logs -f psql
docker container stop <id>

# Create a new postgres container with same named volume using 9.6.2
docker container run -d --name psql2 -v psql:/var/lib/postgresql/data postgres:9.6.2

# Check logs to validate
docker volume ls
docker container logs <id>
```

### 5.6  Assignment: Edit Code Running In Containers With Bind Mounts
Use a Jekyll "Static Site Generator" to start a local web server
Don't have to be web developer: this is example of bridging the gap between local file access and apps running in containers
Source code is in the course repo under bindmount-sample-1
we edit files with editor on our host using native tools
container detects changes with host files and updates web server
start container with docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
Refresh our brower to see changes

### 5.6  Assignment Answers: Edit Code Running In Containers With Bind Mounts

```console
cd bindmount-sample-1
run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
http://localhost/jekyll/update/2020/07/21/welcome-to-jekyll.html

# Change file
2020-07-21-welcome-to-jekyll.markdown
title:  "Welcome to Jekyll!"
title:  "Welcome to Jekyll! Change" 
http://localhost/jekyll/update/2020/07/21/welcome-to-jekyll.html

# As soon as I change the file and automatically save the server goes up again.
```

### 5.7  Database Passwords in Containers
We all know databases usually need passwords, but since the dawn of Docker, the postgres image (and a few others like redis) has allowed you to do a simple docker run on it and it starts without a password. Sure you could set a password but it didn't require one.

In Feburary 2020 that changed, and will affect using postgres in this course (and my others). When running postgres now, you'll need to either set a password, or tell it to allow any connection (which was the default before this change).

For docker run, and the forthcoming Docker Compose sections, you need to either set a password with the environment variable:

POSTGRES_PASSWORD=mypasswd

Or tell it to ignore passwords with the environment variable:

POSTGRES_HOST_AUTH_METHOD=trust


## 6.0 Docker Compose

### 6.1 Docker Compose and The docker-compose.yml File

Why: configure relationships betwenn containers
Why: save out docker container run sttings in easy-to-read file
Why: create one-liner developer environment startups
Comprised of 2 separate but related things
1.Part: YAML-formatted file that describes our solution options for:
	containers
	networks
	volumes
2.Part: A CLI tool docker-compose used for local dev/test automation with those YAML files

**docker-compose.yml**
Documentation: https://docs.docker.com/compose/compose-file/
Compose YAML format has it's own versions:1, 2, 2.1, 3, 3.1
YAML file can be used with docker-compose command for local docker automation or with docker directly in production with Swarm (as of v1.13)
docker-compose --help
docker-compose.yml is default filename, but any can be used with docker-compose -f

template.yml
```
version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```

Example 1
docker-compose.yml
```
version: '2'

# same as 
# docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - '80:4000'

```

Example 2
```
version: '2'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: example
      WORDPRESS_DB_PASSWORD: examplePW
    volumes:
      - ./wordpress-data:/var/www/html

  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: examplerootPW
      MYSQL_DATABASE: wordpress
      MYSQL_USER: example
      MYSQL_PASSWORD: examplePW
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

### 6.2 Tryng Out Basic Compose Commands

docker-compose CLI

CLI tool comes with Docker for Windows/Mac, but separate download for Linux
Not a production-grade tool but ideal for local development and test 
Two most common commands are
```
# setup volumes/networks and start all containers
docker-compose up
# stop all containers and remove cont/vol/net
docker-compose down
```

If all your project had a Dockerfile and docker-compose.yml then "new developer onboarding" would be:
```
git clone github.com/some/software
docker-compose up
```

``` 
# Go to the folder
cd compose-sample-2
# See the file
pcat docker-compose.yml
# Start locally to see the log
docker-compose up
# Sart in background
docker-compose up -d
docker-compose logs
docker-compose --help
# Finish
docker-compose ps
docker-compose top
docker-compose down
```

### 57. Assignment: Build a Compose File For a Multi-Container Service

Assignment: Writing a Compose File

> Goal: Create a compose config for a local Drupal CMS website

- This empty directory is where you should create a docker-compose.yml 
- Use the `drupal:8.8.2` image along with the `postgres:12.1` image
- Set the version to 2
- Use `ports` to expose Drupal on 8080
- Be sure to setup POSTGRES_PASSWORD on postgres image
- Walk though Drupal config in browser at http://localhost:8080
- Tip: Drupal assumes DB is localhost, but it will actually be on the compose service name you give it
- Use Docker Hub documentation to figure out the right environment and volume settings
- Extra Credit: Use volumes to store Drupal unique data

### 58. Assignment Answers: Build a Compose File For a Multi-Container Service
See video

### 59. Adding Image Building to Compose Files
Compose can also build your custom images
will buid them with docker-compose up if not found in cache
also rebuild with docker-compose build
great for complex builds that have lots of vars or build args

cd compose-sample-3
build custom nginx

docker-compose.yml
```
Version: '2'
# based off compose-sample-2, only we build nginx.conf into image
# uses sample HTML static site from https://startbootstrap.com/themes/agency/

services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    ports:
      - '80:80'
  web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs/
```

nginx.Dockerfile
```
FROM nginx:1.13

COPY nginx.conf /etc/nginx/conf.d/default.conf
```

nginx.conf
```
server {

	listen 80;

	location / {

		proxy_pass         http://web;
		proxy_redirect     off;
		proxy_set_header   Host $host;
		proxy_set_header   X-Real-IP $remote_addr;
		proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header   X-Forwarded-Host $server_name;

	}
}
```

### 60. Assignment: Compose For Run-Time Image Building and Multi-Container Development

Assignment: Compose For On-The-Fly Image Building and Multi-Container Testing

Goal: This time imagine you're just wanting to learn Drupal's admin and GUI, or maybe you're a software tester and you need to test a new theme for Drupal. When configured properly, this will let you build a custom image and start everything with `docker-compose up` including storing important db and config data in volumes so the site will remember your changes across Compose restarts.

- Use the compose file you created in the last assignment (drupal and postgres) as a starting point.
- Let's pin image version from Docker Hub this time. It's always a good idea to do that so a new major version doesn't surprise you.

Dockerfile
- First you need to build a custom Dockerfile in this directory, `FROM drupal:8.8.2` NOTE: if it fails to build, try the lastest 8 branch version with `FROM drupal:8`
- Then RUN apt package manager command to install git: `apt-get update && apt-get install -y git`
- Remember to cleanup after your apt install with `rm -rf /var/lib/apt/lists/*` and use `\` and `&&` properly. You can find examples of them in drupal official image. More on this below under Compose file.
- Then change `WORKDIR /var/www/html/themes`
- Then use git to clone the theme with: `RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git`
- Combine that line with this line, as we need to change permissions on files and don't want to use another image layer to do that (it creates size bloat). This drupal container runs as www-data user but the build actually runs as root, so often we have to do things like `chown` to change file owners to the proper user: `chown -R www-data:www-data bootstrap`. Remember the first line needs a `\` at end to signify the next line is included in the command, and at start of next line you should have `&&` to signify "if first command succeeds then also run this command"
- Then, just to be safe, change the working directory back to its default (from drupal image) at `/var/www/html`

Compose File
- We're going to build a custom image in this compose file for drupal service. Use Compose file from previous assignment for Drupal to start with, and we'll add to it, as well as change image name.
- Rename image to `custom-drupal` as we want to make a new image based on the official `drupal:8.8.2`.
- We want to build the default Dockerfile in this directory by adding `build: .` to the `drupal` service. When we add a build + image value to a compose service, it knows to use the image name to write to in our image cache, rather then pull from Docker Hub.
- For the `postgres:12.1` service, you need the same password as in previous assignment, but also add a volume for `drupal-data:/var/lib/postgresql/data` so the database will persist across Compose restarts.

Start Containers, Configure Drupal
- Start containers like before, configure Drupal web install like before.
- After website comes up, click on `Appearance` in top bar, and notice a new theme called `Bootstrap` is there. That's the one we added with our custom Dockerfile.
- Click `Install and set as default`. Then click `Back to site` (in top left) and the website interface should look different. You've successfully installed and activated a new theme in your own custom image without installing anything on your host other then Docker!
- If you exit (ctrl-c) and then `docker-compose down` it will delete containers, but not the volumes, so on next `docker-compose up` everything will be as it was.
- To totally clean up volumes, add `-v` to `down` command.

## Section 7: Swarm Intro and Creating a 3-Node Swarm Cluster

### 62. Swarm Mode: Built-In Orchestration

- How do we automate container lifecycle?
- How can we easily scale out/in/up/down?
- How can we ensure out containers are re-created if they fail?
- How can we replace containers without downtime (blue/green deploy)?
- How can we control/track where containers get started?
- How can we create cross-node virtual networks?
- How can we ensure only trusted servers run out containers?

**Swarm Mode: Buit-in orchertration**

- Swarm mode is a clustering solution built inside docker
- Not related to Swarm "classic" for pre-1.12 versions
- Added in 1.12 (Summer 2016) via SwarmKit tollkit
- Enhanced 1.13 (January 2017) via Stack and Secrets
- Not enabled by default, new commands once enabled
	- docker swarm
	- docker node
	- docker service
	- docker stack
	- docker secret

### 63. Create Your First Service and Scale It Locally

```
# Swarm: inactive
dokcer info
# Initialize
# What just happened?
# Lots of PKI and security automation
# 	- root signing certificate created for our Swarm
#	- Certificate is issued for first manager node
#	- Join tokens are created
# Raft database created to store root CA, configs and secrets
#	- Encrypted by default on disk (1.13+)
#	- No need for another key/value system to hold orchestration/secrets
#	- Replicates logs amongst managers via mutual TLS in "control plane"
docker swarm init
# Swarm: active
dokcer info
# Create a service
docker service create alpine ping 8.8.8.8
# See all services
docker service ls
# See what image use
docker service ps <NAME>
# See containers
docker container ls
# Replicated
docker service update <ID> --replicas 3
docker service ls
docker service ps <NAME>
# Remove
docker container rm -f <name>.1.<ID>
docker service ls
docker service ps <NAME>
```

### 64. UI Change For Service Create/Update

UI Change For Service Create/Update

You may notice that when using docker service create  and update , that the CLI acts differently for some Lectures then your own Docker. This is due to a change in the way Docker CLI shows the service commands in 2017.

    --detach is a new option that changes the CLI response after you run a command.

This is a good thing, and doesn't affect the functionality of Swarm. It's just a UI difference. I use various 2017 versions of Docker in this course, so you may see different output for your own service create/update commands vs. mine, which is fine.

History of changes to CLI output for service create/update:

    Before 17.05, the service commands would immediately return to your shell prompt and the containers would be scheduled in the background (asynchronously). To check if they deployed properly you would need to use docker service ls and docker service ps.
    Starting in 17.05 the service commands were given a --detach  option, which defaulted to true , but warned you each time about a future change to false . The create/update commands were still asynchronous.
    Starting in 17.10 the --detach  default changes to false , so you'll always see the UI wait synchronously while service tasks are deployed/updated, unless you set --detach true  in each command.

For all stable versions after 17.12, just remember:

    Use the defaults if you're interactive at the CLI, typing commands yourself.
    Use --detach true  if you're using automation or shell scripts to get things done.

### 65. Docker Machine Bug With Swarm

Docker Machine Bug With Swarm

If you're planning to use docker-machine to create local VM's for Swarm, note there is a bug in 18.09.0 that prevents Swarm network ports from listening. This is fixed in 18.09.1 so make sure you have the latest Docker Desktop or Docker Toolbox and your VM's are up to date.

Current versions (October 2019) include:

docker version 19.03.2

docker-machine version 0.16.2

To ensure you have the latest Docker Toolbox, download it from the GitHub release page.

You'll know you're up to date when you do a docker-machine ls and the VM's have 18.09.1 or later.

To update existing docker-machine VM's, you can use docker-machine upgrade <name of vm>.

### 66. Creating a 3-Node Swarm Cluster



### Quiz 7: Quiz on Swarm Mode Basics

## Section 8: Swarm Basic Features and How to Use Them In Your Workflow


67. Scaling Out with Overlay Networking

68. Scaling Out with Routing Mesh


69. Assignment: Create A Multi-Service Multi-Node Web App

70. Assignment Answers: Create A Multi-Service Multi-Node Web App

71. Swarm Stacks and Production Grade Compose

72. Secrets Storage for Swarm: Protecting Your Environment Variables

73. Using Secrets in Swarm Services

74. Using Secrets with Swarm Stacks

75. Assignment: Create A Stack with Secrets and Deploy

76. Assignment Answers: Create A Stack with Secrets and Deploy

Section 9: Swarm App Lifecycle
0 / 6|37min

77. Using Secrets With Local Docker Compose
3min
78. Full App Lifecycle: Dev, Build and Deploy With a Single
Compose Design
10min
79. Service Updates: Changing Things In Flight
9min
80. Healthchecks in Dockerfiles
13min
Quiz 9: Quiz on Swarm App Lifecycle
81. Info on Swarm Mastery
1min

Section 10: Container Registries: Image Storage and
Distribution
0 / 7|29min

82. Docker Hub: Digging Deeper
8min
83. Understanding Docker Registry
4min
84. Run a Private Docker Registry
7min
85. Assignment: Secure Docker Registry With TLS and
Authentication
1min
86. Using Docker Registry With Swarm
9min
87. Third Party Image Registries
1min
Quiz 10: Quiz on Container Registries
https://www.youtube.com/watch?v=7Esk0QKURIk
https://www.youtube.com/watch?v=zBA2AQuUUsA
Docker Tutorial - Docker overview - Enable Docker Engine API to remote management

https://www.ivankrizsan.se/2016/05/18/enabling-docker-remote-api-on-ubuntu-16-04/

cd /etc/systemd
sudo mkdir docker.service.d 
tentei dar chmod 777 pra ver se consigo algo igual no ubuntu
cd docker.service.d/
sudo nano docker.conf

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376

sudo systemctl daemon-reload
sudo systemctl restart docker.service
cd ..
cd docker.service.d/
sudo systemctl daemon-reload
sudo systemctl restart docker.service
curl -X GET http://localhost:2376/images/json



# References
https://blog.alexellis.io/getting-started-with-docker-on-raspberry-pi/
https://blog.alexellis.io/5-things-docker-rpi/
https://raspberrypi.stackexchange.com/questions/91475/crosscompiling-exact-archictecture-for-all-models
https://www.raspbian.org/RaspbianFAQ#What_compilation_options_should_be_set_Raspbian_code.3F
https://chowdera.com/2021/07/20210726202011695l.html
https://withblue.ink/2020/06/24/docker-and-docker-compose-on-raspberry-pi-os.html
