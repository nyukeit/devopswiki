# Basic Commands

## `pull`

Pull Docker images from Docker Hub

```bash
# Pulling a docker image
docker pull nginx # By default it pulls the latest image
docker pull nginx:1.21 # Pull a specific version of the image
```

## `run`

Run a container from an image.

```bash
docker run nginx # Starts a container in foreground mode (interactive)
docker run -d nginx # Starts a container in background mode

# Map a random port to the container
docker run -d -P <container id/name>

# Map a chosen port to the container
docker run -d -p <yourport>:<mappedport> <container id/name> # Mapped port is the port of the host machine/VM/Server
```

## `ps`

List of docker containers that are currently running

```bash
docker ps # shows running containers
docker ps -a # shows all containers (running and exited)
docker ps -s # shows containers with added sizes after modifications inside container
```

## `image ls`

List docker images currently downloaded on the local system

```bash
docker image ls
```

## `start`

Start docker containers using either their id or unique name

```bash
docker start <id>
docker start <unique name>
```

## `stop`

Stop docker containers

```bash
docker stop <id> # you need to only provide the first three or four unique characters from id, not the entire id.
docker stop <unique name>
```

## `kill`

Forcefully quit a docker container (for eg. if it is unresponsive)

```bash
docker kill <id>
docker kill <unique name>
```

## `rm`

Remove docker containers. They need to be stopped to be removed.

```bash
docker rm <id>
docker rm <unique name>
docker rm --force <id> # remove running containers
```

## `logs`

Show logs for a particular container

```bash
docker logs <container id>
```

## `inspect`

Get detailed data about a container

```bash
docker inspect <container id>
```

## `stats`

Shows detailed stats about running containers

```bash
docker stats
```

## `exec`

Login to a container’s shell terminal

```bash
docker exec -it <container id/name> /bin/bash # Login to a container's bash shell in interactive mode (-i interactive, t terminal)
dockdr exec <container id/name> ls -l # Display the file contents of the container inside your current terminal
```

## `top`

Check the process that is running inside a container

```bash
docker top <container id>
```

# Build Images

## Dockerfile

Build an image by writing a dockerfile.

```docker
# Pull base image, run it as a container and login to the container
FROM docker.io/library/tomcat:8.5.45

# Change directory and download the app
RUN cd /usr/local/tomcat/webapps/ ; wget <https://github.com/lerndevops/code/raw/main/sampleapp.war>

# Update apt cache
RUN apt-get update

# Install vim editor
RUN apt-get install -y vim

# Create a file named hello.txt
RUN touch /tmp/hello.txt
```

Creating a dockerfile from local files without downloading anything while creation. It is recommended to use the ADD instruction if you need to extract files. COPY will only copy the files but ADD will copy the extracted contents.

```docker
FROM ubuntu:18.04
COPY jdk-8u331-linux-x64.tar.gz /tmp
RUN tar -xzf /tmp/jdk-8u331-linux-x64.tar.gz -C /opt
RUN mv /opt/jdk1.8.0_331 /opt/java
RUN rm /tmp/jdk-8u331-linux-x64.tar.gz
ENV JAVA_HOME /opt/java
ENV JAVA_VERSION 1.8
ADD apache-tomcat-9.0.63.tar.gz /opt
RUN mv /opt/apache-tomcat-9.0.63 /opt/tomcat
ENV TOMCAT_HOME /opt/tomcat
ENV TOMCAT_VERSION 9.0.63
COPY myapp.war /opt/tomcat/webapps/
EXPOSE 8080
CMD ["$TOMCAT_HOME/bin/catalina.sh", "run"] # You cannot have more than one CMD in a Dockerfile as only the last CMD will be executed.
```

## `commit`

Commit changes inside a container as a new image. It works like a `git commit`

```bash
docker commit -m "<msg>" <newimagename>:<tag>
```

### `ENTRYPOINT` vs `CMD`

Entrypoint is the default command executed when a docker container is run. This process cannot be replaced or overwritten.

CMD commands can be overwritten while executing a `docker run` process.

```docker
# Dockerfile with ENTRYPOINT
FROM ubuntu:18.04
ENTRYPOINT ["sleep", "6000"]
docker run -d mybuntu:entry . # Runs docker container with entrypoint
docker run -d mybuntu:entry sleep 900 . # Does not affect as ENTRYPOINT cannot be changed

docker build --file Dockerfile --tag myubuntu:entrypoint .
# Dockerfile with CMD
FROM ubuntu:18.04
CMD ["sleep", "6000"]
docker run -d myubuntu:cmd # Runs docker container with the CMD specified in the dockerfile.
docker run -d mybuntu:cmd sleep 900 . # Runs docker container with the new specifications replacing the original CMD
# Dockerfile with ENTRYPOINT and CMD
FROM ubuntu:18.04
ENTRYPOINT ["sleep"]
CMD ["6000"]
docker run -d myubuntu:entry-cmd . # Runs the docker container with the settings mentioned in dockerfile.
docker run -d myubuntu:entry-cmd 900 . # Replaces the CMD argument with 900 but still runs the sleep command as entrypoint is immutable.
```

## `USER`

The `USER` instruction will make the container use the specified user for any commands specified after the instruction.

```docker
# Dockerfile with USER
FROM ubuntu:18.04
RUN useradd -ms /bin/bash nyukeit # -m sets default directory for user, -s sets the default shell
USER nyukeit
RUN touch /tmp/hello.txt 
CMD ["sleep", "500"]

docker build --file Dockerfile --tag myubuntu:user . # Creates the image
docker run -d <containerid> # Creates the container
docker exec -it <containerid> /bin/bash # Will login as the USER instead of ROOT
```

## `WORKDIR`

Set the default working directory inside the container.

```docker
# Dockerfile with WORKDIR
FROM ubuntu:18.04
RUN mkdir /home/nyukeit
WORKDIR /home/nyukeit
RUN touch hello.txt # This file will now be created in /home/nyukeit instead of the root directory
```

## Multi-Stage Build

Multiple FROM statements in a single Dockerfile.

```docker
# Stage 1 of the build
FROM maven:3.6.3-jdk-8 AS stage1
RUN git clone <https://github.com/lerndevops/samplejavaapp>
WORKDIR /samplejavaapp
RUN mvn package

# Stage 2 of the build
FROM tomcat:8.5
WORKDIR /usr/local/tomcat/webapps/
COPY --from=stage1 /samplejavaapp/target/samplejavaapp.war .
```

Use Case:

In stage 1, docker will compile the entire webapp from source code and create a deployable .war file and a container which is separate from Stage 2. For further usage, we do not need to install Maven again, java again, compile and build the maven package again. If we use a single FROM instruction and use everything together, docker has to perform those steps everytime there is an update, which is unnecessary.

Once Docker is done with the Stage 1 steps, it discards that container and only works with the Stage 2 container.  This keeps the front-facing webapp container lean without needing the additional package installation, etc.

Usually, there are only two stages in a Dockerfile.

## `tag`

Retag an image

```docker
docker tag sampleapp:v4 docker.io/nyukeit/dockertrial:sampleapp-v4
#                                 account repository  imagename tag
```

# Transfering an image from one server to another - Offline

When there is no access to online repositories or locally hosted repositories.

## `save`

```docker
# Generates a tarball with all the docker images included
docker save --output myimages.tar <image1> <image2> <image3> 
```

## `load`

```docker
# Reloads images in another server
docker load --input myimages.tar
```

# Docker Cleanup

## `prune`

```docker
# Remove all stopped containers
docker container prune

# Remove all images without running containers
docker image prune

# Prune entire docker system including images and containers, cache, etc
docker system prune
```

# Docker Local Registry

A docker container as a Docker Registry which is totally private.

```docker
# Pull the official Docker Registry image
docker pull registry:latest

# Create the registry container with a dedicated port
docker run -d -p 5000:5000 --restart always --name registry

# Tag local images to the docker Registry container
docker tag myimage:1 localhost:5000/nyukeit/myapp:1

# Push images
docker push localhost:5000/nyukeit/myapp:1

# Pull images
docker pull localhost:5000/nyukeit/myapp:1
```

## Docker Network

Understand about docker’s networking

```docker
docker network ls

# Ouput
NETWORK ID     NAME      DRIVER    SCOPE
9dd7f68dd225   bridge    bridge    local
550b260adc35   host      host      local
2440bbbbab37   none      null      local
```

### Inspect docker network

```docker
docker network inspect <network name/id>
```

### Create a Network

- Custom bridge networks also enable container name resolution which means you could ping one container from another using the container name instead of ip address

```docker
# Docker used options
docker network create <network name>

# User defined settings
docker network create <name> --driver bridge --subnet <subnet ip> gateway <gateway ip>
```

### Join a container to a custom network

```docker
docker run -d --name <cont> --network <networkname>
docker run -d --name <cont2> --network <networkname>

# These containers are now in a totally different network

docker exec cont1 ping cont2
# This should receive a response because name resolution is enabled in a custom network
```

### Join a container to a second network

```docker
# This makes cont1 a part of 2 different networks, the default bridge and the custom bridge mynet.
docker network connect mynet cont1
```

### `disconnect`

```docker
# This disconnects a container from a network
docker network disconnect mynet cont1
```

## None network

- Cannot be customized
- Cannot create a custom none network
- It means switching off all networking for the container, it becomes unreachable

```docker
docker run -d --name <cont> --network none

# When you do ifconfig on this container, it will only have the local loopback network 
```

## Host network

- Join a container to the host/vm network, outside of other container network.

```docker
docker run -d --name <cont> --network host

# Thsi runs the container as a process on the vm network and is directly accessible using the VM ip.
```

## Overlay Network

- Connect containers across VMs

## Docker Volumes

- Attach external storage to a container
- Makes data persistent
- Docker managed volumes are in /var/lib/docker/volumes - this is accessible only by root.

```docker
# List volumes
docker volume ls

# Create an empty volume
docker volume create <volumename>

# Attach a volume to a container
docker run -dP --mount type=volume,src=<volumename>,target=<pathinsidecontainer> <containername>:<tag>
# src = volumename/path on host
# target = path inside container
# type=volume = docker managed volume i.e. present in /var/lib/docker/volumes/<volumename>/_data 

Example
docker run -dP --mount type=volume,src=applogs,target=/usr/local/tomcat/logs tomcat:latest

# Another syntax
docker run -dP --volume <src>:<target> <containername>:<tag>
#Example
docker run -dP --volume applogs:/usr/local/tomcat/logs tomcat:latest
docker run -dP -v applogs:/usr/local/tomcat/logs tomcat:latest

# Map another directory as a src volume to a container, also called a Bind Mount, user managed volume.
mkdir /opt/nginxlogs
docker run -dP --mount type=bind,src=<volumepath>,target=<pathinsidecontainer> <containername>:<tag>
#Example
docker run -DP --moutn type=bind,src=/opt/nginxlogs,target=/var/log/nginx nginx:latest

# Mount multiple volumes
Use more than one mount agruments to attach multiple volumes.
docker run -dP --moutn type=bind,src=/opt/nginxlogs,target=/var/log/nginx --mount type=bind,src=/opt/nginxlogs,target=/var/log/nginx

# Bind volume with another syntax
docker run -dP -v <pathinsidevm>:<pathinsidecontainer> <containername>:<tag>
#Example
docker run -dP -v /opt/nginxlogs:/var/log/nginx nginx:latest
```

### `tmpfs`

Storage managed by Docker in memory. This is not persistent storage. It is deleted when the container is removed.