# Getting Started with Docker

## The Docker platform

- Docker provides the ability to package and run an application in a loosely isolated environment called a **container**
- Containers are lightweight and contain everything needed to run the application

## Docker architecture

- Docker uses a client-server architecture
- The Docker **client** talks to the Docker **daemon**
- The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon
- Another Docker client is Docker Compose, that lets you work with applications consisting of a set of containers

### The Docker daemon

- The Docker daemon (`dockerd`) listens for Docker API requests
  - manages Docker objects such as images, containers, networks, and volumes
- A daemon can also communicate with other daemons to manage Docker services

### The Docker client

- The Docker client (`docker`) is the primary way that many Docker users interact with Docker
- When you use commands such as `docker run`, the client sends these commands to `dockerd`, which carries them out
- The `docker` command uses the Docker API
- The Docker client can communicate with more than one daemon

### Docker registries

- A Docker registry stores Docker images
- Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default
  - you can run your own private registry
- `docker pull` or `docker run` commands: the required images are pulled from your configured registry
- `docker push` command: your image is pushed to your configured registry

### Docker objects

#### Images

- An image is a read-only template with instructions for creating a Docker container
- Often, an image is based on another image, with some additional customization
  - e.g., an image which is based on the ubuntu image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run
- To build your own image, you create a **Dockerfile**
  - defines the steps needed to create the image and run it
  - each instruction in a Dockerfile creates a layer in the image
  - when you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt

#### Containers

- A container is
  - a runnable instance of an image
  - a sandboxed process on your machine that is isolated from all other processes on the host machine
    - leverages kernel namespaces and cgroups, features that have been in Linux for a long time
- You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine
- A container is defined by its image as well as any configuration options you provide to it when you create or start it
- When a container is removed, any changes to its state that are not stored in persistent storage disappear

### Example `docker run` command

```console
$ docker run -i -t ubuntu /bin/bash
```

- Runs an `ubuntu` container, attaches interactively to your local command-line session, and runs `/bin/bash`
- If you do not have the `ubuntu` image locally, Docker pulls it from your configured registry, as though you had run `docker pull ubuntu` manually
- Docker creates a new container, as though you had run a `docker container create` command manually
- Docker allocates a read-write filesystem to the container, as its final layer
  - allows a running container to create or modify files and directories in its local filesystem
- Docker creates a network interface to connect the container to the default network, since you did not specify any networking options
  - includes assigning an IP address to the container
  - by default, containers can connect to external networks using the host machine’s network connection
- Docker starts the container and executes `/bin/bash`
  - the container is running interactively and attached to your terminal due to the `-i` and `-t` flags
- When you type `exit` to terminate the `/bin/bash` command, the container stops but is not removed
  - you can start it again or remove it

## The underlying technology

- Docker takes advantage of several features of the Linux kernel to deliver its functionality
- Docker uses a technology called `namespaces` to provide the isolated workspace called the container
  - when you run a container, Docker creates a set of namespaces for that container 
  - these namespaces provide a layer of isolation
  - each aspect of a container runs in a separate namespace and its access is limited to that namespace

## Install Docker Engine on Ubuntu

### Set up the repository

- Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:

```console
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl gnupg lsb-release
```

- Add Docker's official GPG key

```console
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

- Set up the repository

```console
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine

- Update the `apt` package index, and install the latest version of Docker Engine, containerd, and Docker Compose

```console
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- Verify that Docker Engine is installed correctly by running the `hello-world` image

```console
$ sudo docker run hello-world
```

### Manage Docker as a non-root user

- The Docker daemon binds to a Unix socket instead of a TCP port
  - by default that Unix socket is owned by the user `root`
  - the Docker daemon always runs as the `root` user
- Create a Unix group called `docker` and add users to it
  - when the Docker daemon starts, it creates a Unix socket accessible by members of the `docker` group

```console
$ sudo groupadd docker
```

- Add your user to the `docker` group

```console
$ sudo usermod -aG docker $USER
```

- Log out and log back in so that your group membership is re-evaluated, or

```console
$ newgrp docker
```

- Verify that you can run `docker` commands without `sudo`

```console
docker run hello-world
```

### Configure Docker to start on boot

- On Debian and Ubuntu, the Docker service is configured to start on boot by default
- To disable starting the Docker service on boot:

```console
$ sudo systemctl disable docker.service
$ sudo systemctl disable containerd.service
```

- To enable starting the Docker service on boot:

```console
$ sudo systemctl enable docker.service
$ sudo systemctl enable containerd.service
```

- To start the Docker service:

```console
$ sudo systemctl start docker.service
$ sudo systemctl start containerd.service
```

- To stop the Docker service:

```console
$ sudo systemctl stop docker.service
$ sudo systemctl stop containerd.service
```

- Note: `docker.service` can still be activated by `docker.socket` after `docker.service` has been stopped
  - to disable this:

```console
$ sudo systemctl stop docker.socket
```

- To list Docker-related services:

```console
$ systemctl list-units --no-pager | egrep -i "container|docker"
  sys-devices-virtual-net-docker0.device                                                   loaded active     plugged   /sys/devices/virtual/net/docker0
  sys-subsystem-net-devices-docker0.device                                                 loaded active     plugged   /sys/subsystem/net/devices/docker0
  containerd.service                                                                       loaded active     running   containerd container runtime
  docker.service                                                                           loaded active     running   Docker Application Container Engine
  docker.socket                                                                            loaded active     running   Docker Socket for the API
```

### Uninstall Docker Engine

- Uninstall the Docker Engine, CLI, containerd, and Docker Compose packages:

```console
$ sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- Images, containers, volumes, or customized configuration files on your host are not automatically removed
- To delete all images, containers, and volumes:

```console
$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /var/lib/containerd
```

- You must delete any edited configuration files manually

## Sample application

- See [`app`](app)

### Build the app's container image

- A Dockerfile is a text-based script of instructions that is used to create a container image
- See [`Dockerfile`](app/Dockerfile)
- Build the container image from the `app` directory:
  - `docker build -t getting-started .`
    - `-t`: tag the image - a human-readable name for the final image, `getting-started`
    - `.`: look for `Dockerfile` in the current directory

```console
$ docker build -t getting-started .
Sending build context to Docker daemon  4.654MB
Step 1/7 : FROM node:12-alpine
12-alpine: Pulling from library/node
df9b9388f04a: Pull complete 
3bf6d7380205: Pull complete 
7939e601ee5e: Pull complete 
31f0fb9de071: Pull complete 
Digest: sha256:d4b15b3d48f42059a15bd659be60afe21762aae9d6cbea6f124440895c27db68
Status: Downloaded newer image for node:12-alpine
 ---> bb6d28039b8c
Step 2/7 : RUN apk add --no-cache python2 g++ make
 ---> Running in 657067da95f4
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/22) Installing binutils (2.37-r3)
(2/22) Installing libgomp (10.3.1_git20211027-r0)
...
(22/22) Installing python2 (2.7.18-r4)
Executing busybox-1.34.1-r5.trigger
OK: 229 MiB in 38 packages
Removing intermediate container 657067da95f4
 ---> 1391e56b4519
Step 3/7 : WORKDIR /app
 ---> Running in d65e32b9e401
Removing intermediate container d65e32b9e401
 ---> 805e1ef0150d
Step 4/7 : COPY . .
 ---> 15b9cecfb1d1
Step 5/7 : RUN yarn install --production
 ---> Running in fc92b90e2f79
yarn install v1.22.18
[1/4] Resolving packages...
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^2.0.0"
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^3.0.0"
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 8.15s.
Removing intermediate container fc92b90e2f79
 ---> 7cc9ab23cd2a
Step 6/7 : CMD ["node", "src/index.js"]
 ---> Running in 7318e94174f9
Removing intermediate container 7318e94174f9
 ---> 556a895cfe58
Step 7/7 : EXPOSE 3000
 ---> Running in e70851750781
Removing intermediate container e70851750781
 ---> 5e46b6aa4a3f
Successfully built 5e46b6aa4a3f
Successfully tagged getting-started:latest
```

### Start an app container

- Start the container:
  - `docker run -dp 3000:3000 getting-started`
    - `-d`: run the container in the background (in "detached" mode) and print the container ID
    - `-p`: publish the container's port(s) to the host, mapping between the host's port 3000 to the container's port 3000

```console
$ docker run -dp 3000:3000 getting-started
c8df059c1dd5e861a5dfbbcc22a9fa55ce88050c72c39f306184c039e8ab2c66
```

- Open your web browser to <http://localhost:3000>
- List docker images:
  - `docker images`

```console
$ docker images
REPOSITORY        TAG         IMAGE ID       CREATED         SIZE
getting-started   latest      5e46b6aa4a3f   3 hours ago     404MB
node              12-alpine   bb6d28039b8c   3 months ago    91MB
hello-world       latest      feb5d9fea6a5   10 months ago   13.3kB
```

- List containers:
  - `docker ps`

```console
$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                       NAMES
c8df059c1dd5   getting-started   "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   sad_chandrasekhar
```

## Sources

- "Docker Overview." _Docker Documentation_, 18 July 2022, [docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/). Accessed 18 July 2022.
- "Install Docker Engine on Ubuntu." _Docker Documentation_, 20 July 2022, [docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/). Accessed 20 July 2022.
- "Post-Installation Steps for Linux." _Docker Documentation_, 20 July 2022, [docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/). Accessed 20 July 2022.
- "Sample Application." _Docker Documentation_, 22 July 2022, [docs.docker.com/get-started/02_our_app/](https://docs.docker.com/get-started/02_our_app/). Accessed 23 July 2022.
- "Dockerfile Reference." _Docker Documentation_, 22 July 2022, [docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/). Accessed 23 July 2022.
