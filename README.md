# Getting Started with Docker

<!-- TOC -->
* [Getting Started with Docker](#getting-started-with-docker)
  * [The Docker platform](#the-docker-platform)
  * [Docker architecture](#docker-architecture)
    * [The Docker daemon](#the-docker-daemon)
    * [The Docker client](#the-docker-client)
    * [Docker registries](#docker-registries)
    * [Docker objects](#docker-objects)
      * [Images](#images)
      * [Containers](#containers)
    * [Example `docker run` command](#example-docker-run-command)
  * [The underlying technology](#the-underlying-technology)
  * [Install Docker Engine on Ubuntu](#install-docker-engine-on-ubuntu)
    * [Set up the repository](#set-up-the-repository)
    * [Install Docker Engine](#install-docker-engine)
    * [Manage Docker as a non-root user](#manage-docker-as-a-non-root-user)
    * [Configure Docker to start on boot](#configure-docker-to-start-on-boot)
    * [Uninstall Docker Engine](#uninstall-docker-engine)
  * [Sample application](#sample-application)
    * [Build the app's container image](#build-the-apps-container-image)
    * [Start an app container](#start-an-app-container)
    * [Update the application](#update-the-application)
    * [Share the application](#share-the-application)
    * [Persist the DB](#persist-the-db)
      * [The container's filesystem](#the-containers-filesystem)
      * [Container volumes](#container-volumes)
      * [Persist the todo data](#persist-the-todo-data)
    * [Use bind mounts](#use-bind-mounts)
    * [Multi container apps](#multi-container-apps)
      * [Start MySQL](#start-mysql)
      * [Connect to MySQL](#connect-to-mysql)
      * [Run your app with MySQL](#run-your-app-with-mysql)
    * [Use Docker Compose](#use-docker-compose)
      * [Define the app service](#define-the-app-service)
      * [Define the MySQL service](#define-the-mysql-service)
      * [Run the application stack](#run-the-application-stack)
      * [Tear it all down](#tear-it-all-down)
    * [Image-building best practices](#image-building-best-practices)
      * [Security scanning](#security-scanning)
      * [Image layering](#image-layering)
      * [Layer caching](#layer-caching)
      * [Multi-stage builds](#multi-stage-builds)
  * [Getting started with Java](#getting-started-with-java)
    * [Build your Java image](#build-your-java-image)
      * [Enable BuildKit](#enable-buildkit)
      * [Test the application without Docker](#test-the-application-without-docker)
      * [Create a Dockerfile for Java](#create-a-dockerfile-for-java)
      * [Create a `.dockerignore` file](#create-a-dockerignore-file)
      * [Build an image](#build-an-image)
      * [Build image failure with `mvnw dependency:go-offline`](#build-image-failure-with-mvnw-dependencygo-offline)
    * [Run your image as a container](#run-your-image-as-a-container)
  * [Sources](#sources)
<!-- TOC -->

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

### Update the application

- Update the source code in `src/static/js/app.js`:

```html
-                <p className="text-center">No items yet! Add one above!</p>
+                <p className="text-center">You have no todo items yet! Add one above!</p>
```

- Stop the container:
  - `docker stop c8df059c1dd5`
    - `c8df059c1dd5` is the container ID obtained via `docker ps`

```console
$ docker stop c8df059c1dd5
c8df059c1dd5

$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS                     PORTS     NAMES
c8df059c1dd5   getting-started   "docker-entrypoint.s…"   4 minutes ago   Exited (0) 3 minutes ago             sad_chandrasekhar
```

- Remove the container:
  - `docker rm c8df059c1dd5`
- Build the updated version of the image:
  - `docker build -t getting-started .`

```console
$ docker build -t getting-started .
Sending build context to Docker daemon  4.655MB
Step 1/7 : FROM node:12-alpine
 ---> bb6d28039b8c
Step 2/7 : RUN apk add --no-cache python2 g++ make
 ---> Using cache
 ---> 1391e56b4519
Step 3/7 : WORKDIR /app
 ---> Using cache
 ---> 805e1ef0150d
Step 4/7 : COPY . .
 ---> d463bb69b63d
Step 5/7 : RUN yarn install --production
 ---> Running in 9db2e1143361
yarn install v1.22.18
[1/4] Resolving packages...
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^2.0.0"
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^3.0.0"
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 7.15s.
Removing intermediate container 9db2e1143361
 ---> 5d27ac92cb88
Step 6/7 : CMD ["node", "src/index.js"]
 ---> Running in ee54c36bb9bb
Removing intermediate container ee54c36bb9bb
 ---> 8c97124a9f9e
Step 7/7 : EXPOSE 3000
 ---> Running in 591fea5861dd
Removing intermediate container 591fea5861dd
 ---> 2c0bb796933b
Successfully built 2c0bb796933b
Successfully tagged getting-started:latest

$ docker images
REPOSITORY        TAG         IMAGE ID       CREATED          SIZE
getting-started   latest      2c0bb796933b   12 seconds ago   404MB
<none>            <none>      5e46b6aa4a3f   7 hours ago      404MB
node              12-alpine   bb6d28039b8c   3 months ago     91MB
hello-world       latest      feb5d9fea6a5   10 months ago    13.3kB
```

- Note the new image ID, `2c0bb796933b`
- Start the updated app container:
  - `docker run -dp 3000:3000 getting-started`
- Check the update at <http://localhost:3000/>

### Share the application

- To share Docker images, you have to use a Docker registry
- The default registry is Docker Hub and is where all the images we’ve used have come from
- Create a Docker Hub repository
  - sign in (or sign up to) [Docker Hub](https://hub.docker.com/)
  - create a repository named `getting-started` with `Public` visibility
- Push the image
  - tag the `getting-started` image with your Docker (Hub) ID
    - `docker tag getting-started DOCKER-ID/getting-started`
  - log in to the Docker Hub
    - `docker login -u DOCKER-ID`
  - push the image to Docker Hub
    - `docker push DOCKER-ID/getting-started`
    - as we didn't specify a tag (as in the `TAG` in `docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`), Docker will use a tag called `latest`

```console
$ docker images
REPOSITORY                     TAG         IMAGE ID       CREATED         SIZE
jashburn8020/getting-started   latest      2c0bb796933b   12 hours ago    404MB
getting-started                latest      2c0bb796933b   12 hours ago    404MB
<none>                         <none>      5e46b6aa4a3f   19 hours ago    404MB
node                           12-alpine   bb6d28039b8c   3 months ago    91MB
hello-world                    latest      feb5d9fea6a5   10 months ago   13.3kB

$ docker push jashburn8020/getting-started
Using default tag: latest
The push refers to repository [docker.io/jashburn8020/getting-started]
dd6ac768356c: Pushed 
a2ba117ef57f: Pushed 
23606073410e: Pushed 
072dfb56dc9e: Pushed 
7f30cde3f699: Pushed 
fe810f5902cc: Pushed 
dfd8c046c602: Pushed 
4fc242d58285: Pushed 
latest: digest: sha256:d7011bc85f8269cb27bcd48c1b588fbd47a1488076e8af9c509462afb0c4720a size: 1999
```

- Note: if we hadn't changed `getting-started` to `DOCKER-ID/getting-started`, the push would fail

```console
$ docker push getting-started
Using default tag: latest
The push refers to repository [docker.io/library/getting-started]
dd6ac768356c: Preparing 
a2ba117ef57f: Preparing 
23606073410e: Preparing 
072dfb56dc9e: Preparing 
7f30cde3f699: Preparing 
fe810f5902cc: Waiting 
dfd8c046c602: Waiting 
4fc242d58285: Waiting 
denied: requested access to the resource is denied
```

- Log out of Docker Hub
  - `docker logout`

### Persist the DB

#### The container's filesystem

- When a container runs, it uses the various layers from an image for its filesystem
- Each container also gets its own “scratch space” to create/update/remove files
- Any changes won’t be seen in another container, even if they are using the same image
- Start an `ubuntu` container that will create a file named `/data.txt` with a random number between 1 and 10000
  - `docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"`
    - note: `tail -f` to simply watch a file to keep the container running
- See the content of the `/data.txt` file
  - `docker exec <container-id> cat /data.txt`

```console
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
9bffc8c7d57b   ubuntu    "bash -c 'shuf -i 1-…"   16 minutes ago   Up 16 minutes             dazzling_hodgkin

$ docker exec 9bffc8c7d57b cat /data.txt
8605

$ docker exec -it 9bffc8c7d57b bash
root@9bffc8c7d57b:/# ls
bin   data.txt  etc   lib    lib64   media  opt   root  sbin  sys  usr
boot  dev       home  lib32  libx32  mnt    proc  run   srv   tmp  var
root@9bffc8c7d57b:/# cat data.txt 
8605
root@9bffc8c7d57b:/# exit
```

- Start another `ubuntu` container (the same image) - `data.txt` is not there
  - `docker run -it ubuntu ls /`

```console
$ docker run -it ubuntu ls /
bin   dev  home  lib32	libx32	mnt  proc  run	 srv  tmp  var
boot  etc  lib	 lib64	media	opt  root  sbin  sys  usr
```

- Remove the first container
  - `docker rm -f <container-id>`

#### Container volumes

- [Volumes](https://docs.docker.com/storage/volumes/) provide the ability to connect specific filesystem paths of the container back to the host machine
  - if a directory in the container is mounted, changes in that directory are also seen on the host machine
  - if we mount that same directory across container restarts, we’d see the same files
- There are two main types of volumes:
  - named volumes
  - bind mounts

#### Persist the todo data

- The todo app stores its data in a SQLite Database at `/etc/todos/todo.db` in the container’s filesystem
  - see [`app/src/persistence/sqlite.js`](app/src/persistence/sqlite.js)
- We can persist the data by creating a volume and attaching (often called “mounting”) it to the directory the data is stored in
  - as our container writes to the `todo.db` file, it will be persisted to the host in the volume
- We are going to use a named volume
  - Docker maintains the physical location on the disk and you only need to remember the name of the volume
- Stop and remove the todo app container (with `docker rm -f <container-id>`) if it is still running
- Create a volume
  - `docker volume create todo-db`
- Check the location of the volume

```console
$ docker volume inspect todo-db
[
    {
        "CreatedAt": "2022-07-26T02:11:45+01:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

- Start the todo app container, but add the `-v` flag to specify a volume mount and mount it to `/etc/todos`
  - this will capture all files created at the path
  - `docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started`
- Open the app and add a few items to the todo list
- Check that `todo.db` is in the container's filesystem

```console
$ docker exec ba3a05573a3d ls /etc/todos
todo.db
```

- Check that `todo.db` is stored in the volume

```console
$ sudo ls /var/lib/docker/volumes/todo-db/_data
todo.db
```

- Stop and remove the container for the todo app (with `docker rm -f <container-id>`)
- Start a new container using the same command from above
- Open the app and see that the items are still in the list
- Remove the container when you’re done checking out the list
- Remove the volume
  - `docker volume rm todo-db`

### Use bind mounts

- With bind mounts, we control the exact mountpoint on the host
- We can use this to
  - persist data
  - provide additional data into containers
  - mount our source code into the container to let the application see code changes, respond, and let us see the changes right away
- Using bind mounts is very common for local development setups
  - the dev machine doesn't need to have all the build tools and environments installed
- To run our container to support a development workflow
  - mount our source code into the container
  - install all dependencies, including the “dev” dependencies
  - start nodemon to watch for filesystem changes
    - [nodemon](https://npmjs.com/package/nodemon): a tool to watch for file changes and then restart the application

```console
$ docker run -dp 3000:3000 -w /app -v "$(pwd):/app" node:12-alpine sh -c "yarn install && yarn run dev"
```

- Run the above command from the `app` directory
  - `-dp 3000:3000`: run in detached (background) mode and create a port mapping
  - `-w /app`: sets the “working directory” or the current directory that the command will run from
  - `-v "$(pwd):/app"`: bind mount the current directory from the host in the container into the `/app` directory
    - note: files created in the container's `/app` directory will also "synced" to the current directory
  - `node:12-alpine`: the image to use - this is the base image for our app from the Dockerfile
  - `sh -c "yarn install && yarn run dev"`: the command
    - run `yarn install` to install all dependencies and then run `yarn run dev`
    - the `dev` script in [package.json](app/package.json) starts `nodemon`

```console
$ docker exec -it 32152b48329c sh
/app # ls
Dockerfile    package.json  src
node_modules  spec          yarn.lock
/app # exit
```

- Watch the logs
  - `docker logs`

```console
$ docker logs 32152b48329c
yarn install v1.22.18
[1/4] Resolving packages...
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^2.0.0"
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^3.0.0"
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 6.84s.
yarn run v1.22.18
$ nodemon src/index.js
[nodemon] 2.0.13
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node src/index.js`
Using sqlite database at /etc/todos/todo.db
Listening on port 3000
```

- Edit the app in the `src/static/js/app.js` file to change the “Add Item” button to simply say “Add”:

```text
-                         {submitting ? 'Adding...' : 'Add Item'}
+                         {submitting ? 'Adding...' : 'Add'}
```

```console
$ docker logs 32152b48329c
yarn install v1.22.18
[1/4] Resolving packages...
...
Using sqlite database at /etc/todos/todo.db
Listening on port 3000
[nodemon] restarting due to changes...
[nodemon] starting `node src/index.js`
Using sqlite database at /etc/todos/todo.db
Listening on port 3000
```

- Open your web browser to <http://localhost:3000>
  - you should see the change reflected in the browser
- When you’re done, stop the container and build your new image using:
  - `docker build -t getting-started .`

### Multi container apps

- Adding MySQL to the application stack - where will MySQL run, the same container or run it separately
  - each container should do one thing and do it well
    - scale APIs and front-ends differently than databases 
    - version and update versions in isolation
    - use a managed service for the database in production
    - running multiple processes will require a process manager (the container only starts one process), which adds complexity to container startup/shutdown
- Container networking: if two containers are on the same network, they can talk to each other, otherwise they can't

#### Start MySQL

- 2 ways to put a container on a network:
  1. assign it at start
  2. connect an existing container 
- The following will create the network first and attach the MySQL container at startup
- Create the network 
  - `docker network create todo-app`

```console
$ docker network create todo-app
d45705b749291bc9b15dbb78b1b0be28ac9115d332b188eaa3e8914ca6bdd009

$ docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
0a249546e0b4   bridge     bridge    local
bb85c7780343   host       host      local
1ff9908fd506   none       null      local
d45705b74929   todo-app   bridge    local
```

- Start a MySQL container and attach it to the network
  - `docker run -d --network todo-app --network-alias mysql -v todo-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:5.7`
    - `--network`: connect a container to a network
    - `--network-alias`: add network-scoped alias for the container
    - see <https://hub.docker.com/_/mysql/> for the environment variables to initialize the database
    - we’re using a volume named `todo-mysql-data` and mounting it at `/var/lib/mysql`, which is where MySQL stores its data
      - we never ran a `docker volume create` command, but Docker recognizes we want to use a named volume and creates one automatically for us
- Connect to the database and verify it connects
  - `docker exec -it <mysql-container-id> mysql -u root -p`
  - type `secret` at the password prompt
- In the MySQL shell, list the databases and verify you see the `todos` database 
  - `SHOW DATABASES;`
- Exit the MySQL shell
  - `exit`

```console
$ docker exec -it 3f1258495fd8 mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.39 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| todos              |
+--------------------+
5 rows in set (0.00 sec)

mysql> exit
Bye
```

#### Connect to MySQL

- Use of the [nicolaka/netshoot](https://github.com/nicolaka/netshoot) container to find the MySQL container
  - a Docker + Kubernetes network troubleshooting swiss-army container
  - `docker run -it --network todo-app nicolaka/netshoot`
- Inside the container, use the `dig` command to look up the IP address for the hostname `mysql`
  - `dig mysql`

```console
$ docker run -it --network todo-app nicolaka/netshoot
Unable to find image 'nicolaka/netshoot:latest' locally
latest: Pulling from nicolaka/netshoot
...
4aafb6d72b2b: Pull complete 
Digest: sha256:aeafd567d7f7f1edb5127ec311599bb2b8a9c0fb31d7a53e9cff26af6d29fd4e
Status: Downloaded newer image for nicolaka/netshoot:latest
                    dP            dP                           dP   
                    88            88                           88   
88d888b. .d8888b. d8888P .d8888b. 88d888b. .d8888b. .d8888b. d8888P 
88'  `88 88ooood8   88   Y8ooooo. 88'  `88 88'  `88 88'  `88   88   
88    88 88.  ...   88         88 88    88 88.  .88 88.  .88   88   
dP    dP `88888P'   dP   `88888P' dP    dP `88888P' `88888P'   dP   
                                                                    
Welcome to Netshoot! (github.com/nicolaka/netshoot)
                                                                


 93798b3f43c7  ~  dig mysql

; <<>> DiG 9.18.3 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10433
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.				IN	A

;; ANSWER SECTION:
mysql.			600	IN	A	172.18.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Mon Aug 01 18:11:58 UTC 2022
;; MSG SIZE  rcvd: 44
```

- The “ANSWER SECTION” shows an A record for `mysql` that resolves to `172.18.0.2`
  - A (Address) record:
    - maps a domain name to the IP (v4) address of the computer hosting the domain
    - simplest type of DNS record, and one of the primary records used in DNS servers
  - Docker was able to resolve `mysql` to the IP address of the container that had that network alias
  - our app only simply needs to connect to a host named `mysql` and it’ll talk to the database

#### Run your app with MySQL

- The todo app supports the setting of a few environment variables to specify MySQL connection settings
  - see [`app/src/persistence/index.js`](app/src/persistence/index.js), [`app/src/persistence/mysql.js`](app/src/persistence/mysql.js)
- Note: while using env vars to set connection settings is generally ok for development, it is **HIGHLY DISCOURAGED** when running applications in production
  - see <https://diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/>
  - a more secure mechanism is to use the secret support provided by your container orchestration framework
  - in most cases, these secrets are mounted as files in the running container
  - many apps (including the MySQL image and the todo app) also support env vars with a `_FILE` suffix to point to a file containing the variable
  - e.g., setting the `MYSQL_PASSWORD_FILE` var will cause the app to use the contents of the referenced file as the connection password
- Note: for MySQL versions 8.0 and higher, include the following commands in `mysql`:
  - see also <https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password>

```text
mysql> ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret';
mysql> flush privileges;
```

- Connect the container to our app network
  - `docker run -dp 3000:3000 -w /app -v "$(pwd):/app" --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine sh -c "yarn install && yarn run dev"`

```console
$ docker run -dp 3000:3000 -w /app -v "$(pwd):/app" --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine sh -c "yarn install && yarn run dev"
b79c8faaae6d39009b40926ff3cb4d4b988c9ab911b3cdc0e247831954b5aa86

$ docker logs b79c8faaae6d39009b40926ff3cb4d4b988c9ab911b3cdc0e247831954b5aa86
yarn install v1.22.18
[1/4] Resolving packages...
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^2.0.0"
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^3.0.0"
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 7.36s.
yarn run v1.22.18
$ nodemon src/index.js
[nodemon] 2.0.13
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node src/index.js`
Waiting for mysql:3306.
Connected!
Connected to mysql db at host mysql
Listening on port 3000
```

- Open the app in your browser and add a few items to your todo list
- Connect to the mysql database and prove that the items are being written to the database 
  - `docker exec -it <mysql-container-id> mysql -p todos`

```console
$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                       NAMES
b79c8faaae6d   node:12-alpine   "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   jovial_moore
11cf51316e2b   mysql:5.7        "docker-entrypoint.s…"   8 minutes ago   Up 8 minutes   3306/tcp, 33060/tcp                         beautiful_wu

$ docker exec -it 11cf51316e2b mysql -p todos
Enter password: 
Reading table information for completion of table and column names
...
mysql> select * from todo_items;
+--------------------------------------+------+-----------+
| id                                   | name | completed |
+--------------------------------------+------+-----------+
| 6ac6cbdb-48fe-4405-9c37-619217ea797e | qwer |         0 |
| cdc4125a-dd50-43e8-af8c-bb0c92ef5c61 | asdf |         0 |
| cbde99fb-fea6-4b66-bc0a-8c981e8ef832 | zxcv |         0 |
+--------------------------------------+------+-----------+
3 rows in set (0.00 sec)

mysql> exit
Bye
```

### Use Docker Compose

- A tool to help define and share multi-container applications
- A YAML file to define the services and with a single command, can spin everything up or tear it all down
- Advantages:
  - define your application stack in a file
  - version-control it in your project repo
  - contributors to your project would only need to clone your repo and start the compose app
- At the root of the app project, create a file named [`compose.yaml`](app/compose.yaml)

#### Define the app service

- The command we were using to define our app container:

```text
docker run -dp 3000:3000 \
-w /app -v "$(pwd):/app" \
--network todo-app \
-e MYSQL_HOST=mysql \
-e MYSQL_USER=root \
-e MYSQL_PASSWORD=secret \
-e MYSQL_DB=todos \
node:12-alpine \
sh -c "yarn install && yarn run dev"
```

- The `app` service in `compose.yaml`:

```yaml
services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - target: 3000
        published: 3000
        mode: host
    working_dir: /app
    volumes:
      - type: bind
        source: ./
        target: /app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

- Notes:
  - `app`: the service name can be anything and it will automatically become a network alias
    - see <https://docs.docker.com/compose/compose-file/#services-top-level-element>
  - `ports`: long syntax is used here
    - see <https://docs.docker.com/compose/compose-file/#ports>
  - `volumes`: long syntax is used here
    - see <https://docs.docker.com/compose/compose-file/#volumes>
  - `environment`: can use either an array or a map
    - see <https://docs.docker.com/compose/compose-file/#environment>

#### Define the MySQL service

- The command that we used to define the MySQL container:

```text
docker run -d \
--network todo-app --network-alias mysql \
-v todo-mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=secret \
-e MYSQL_DATABASE=todos \
mysql:5.7
```

- The `mysql` service in `compose.yaml`:

```yaml
services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
    volumes:
      - type: volume
        source: todo-mysql-data
        target: /var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

- Notes:
  - top-level `volumes`: the `todo-mysql-data` named volume is not create automatically when running with Compose, unlike with `docker run`
    - need to define the volume in the top-level `volumes:` section and then specify the mountpoint in the service config
    - by simply providing only the volume name, the default options are used
    - see <https://docs.docker.com/compose/compose-file/#volumes-top-level-element>

#### Run the application stack

- Make sure no other copies of the app/db are running
- Start up the application stack
  - `docker compose up -d`
    - the `-d` flag to run everything in the background

```console
$ docker compose up -d
[+] Running 4/4
 ⠿ Network app_default           Created                                   0.0s
 ⠿ Volume "app_todo-mysql-data"  Created                                   0.0s
 ⠿ Container app-app-1           Started                                   0.5s
 ⠿ Container app-mysql-1         Started                                   0.5s

$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                       NAMES
eee8d0219dd2   node:12-alpine   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   app-app-1
6545ba1749eb   mysql:5.7        "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp, 33060/tcp                         app-mysql-1
```

- Look at the logs using the `docker compose logs -f` command
  - the logs from each of the services interleaved into a single stream
    - useful when you want to watch for timing-related issues
    - the `-f` flag “follows” the log, so will give you live output as it’s generated
  - the service name is displayed at the beginning of the line (often colored) to help distinguish messages
  - if you want to view the logs for a specific service, you can add the service name to the end of the logs command (for example, `docker-compose logs -f app`)
- Waiting for the DB before starting the app 
  - when the app is starting up, it actually sits and waits for MySQL to be up and ready before trying to connect to it
  - Docker doesn't have any built-in support to wait for another container to be fully up, running, and ready before starting another container
  - for Node-based projects, you can use the [wait-port dependency](https://github.com/dwmkerr/wait-port)
    - similar projects exist for other languages/frameworks

#### Tear it all down

- When you’re ready to tear it all down, run `docker compose down`
  - the containers will stop and the network will be removed
  - named volumes in your compose file are NOT removed
    - add the `--volumes` flag if you want to remove the volumes

```console
$ docker compose down --volumes
[+] Running 4/4
 ⠿ Container app-mysql-1       Removed                                     3.9s
 ⠿ Container app-app-1         Removed                                     0.3s
 ⠿ Network app_default         Removed                                     0.1s
 ⠿ Volume app_todo-mysql-data  Removed                                     0.0s
```

- Once torn down, you can switch to another project, run `docker compose up` and be ready to contribute to that project

### Image-building best practices

#### Security scanning

- When you have built an image, scan it for security vulnerabilities using the `docker scan` command
  - Docker has partnered with [Snyk](https://snyk.io/) to provide the vulnerability scanning service
- Log in to Docker Hub to scan your images
  - `docker scan --login`
- Scan your images
  - `docker scan <image-name>`
- See <https://docs.docker.com/engine/scan/>

```console
$ docker scan --login

To authenticate your account, open the below URL in your browser.
After your authentication is complete, return to this prompt to start using Snyk.

https://snyk.io/login?token=<token>&utm_medium=Partner&utm_source=Docker&utm_campaign=Docker-Desktop-2020&os=linux&docker=true


Your account has been authenticated. Snyk is now ready to be used.

$ docker scan getting-started

Testing getting-started...

✗ High severity vulnerability found in openssl/libcrypto1.1
  Description: Inadequate Encryption Strength
  Info: https://snyk.io/vuln/SNYK-ALPINE315-OPENSSL-2941810
  Introduced through: openssl/libcrypto1.1@1.1.1n-r0, openssl/libssl1.1@1.1.1n-r0, apk-tools/apk-tools@2.12.7-r3, libretls/libretls@3.3.4-r3, python2/python2@2.7.18-r4
  From: openssl/libcrypto1.1@1.1.1n-r0
  From: openssl/libssl1.1@1.1.1n-r0 > openssl/libcrypto1.1@1.1.1n-r0
  From: apk-tools/apk-tools@2.12.7-r3 > openssl/libcrypto1.1@1.1.1n-r0
  and 6 more...
  Fixed in: 1.1.1q-r0

------------ Detected 6 vulnerabilities for node@12.22.12 ------------ 


✗ Medium severity vulnerability found in node
  Description: DNS Rebinding
  Info: https://snyk.io/vuln/SNYK-UPSTREAM-NODE-2946423
  Introduced through: node@12.22.12
  From: node@12.22.12
  Fixed in: 14.20.0, 16.16.0, 18.5.0

...

Organization:      jashburn8020
Package manager:   apk
Project name:      docker-image|getting-started
Docker image:      getting-started
Platform:          linux/amd64
Base image:        node:12.22.12-alpine3.15
Licenses:          enabled

Tested 39 dependencies for known issues, found 7 issues.

Base Image                Vulnerabilities  Severity
node:12.22.12-alpine3.15  7                0 critical, 1 high, 6 medium, 0 low

Recommendations for base image upgrade:

Major upgrades
Base Image               Vulnerabilities  Severity
node:fermium-alpine3.15  0                0 critical, 0 high, 0 medium, 0 low

Alternative image types
Base Image                  Vulnerabilities  Severity
node:18.6.0-slim            45               0 critical, 1 high, 0 medium, 44 low
node:16.16.0-bullseye-slim  45               0 critical, 1 high, 0 medium, 44 low
node:18.5.0-buster-slim     71               0 critical, 1 high, 0 medium, 70 low


-------------------------------------------------------

Testing getting-started...

Tested 190 dependencies for known issues, found 7 issues.


Issues to fix by upgrading:

  Upgrade sqlite3@5.0.2 to sqlite3@5.0.3 to fix
  ✗ Regular Expression Denial of Service (ReDoS) [Low Severity][https://snyk.io/vuln/SNYK-JS-TAR-1536758] in tar@2.2.2
    introduced by sqlite3@5.0.2 > node-gyp@3.8.0 > tar@2.2.2
...



Organization:      jashburn8020
Package manager:   yarn
Target file:       /app/package.json
Project name:      101-app
Docker image:      getting-started
Licenses:          enabled


Tested 2 projects, 2 contained vulnerable paths.
```

- You can also [configure Docker Hub](https://docs.docker.com/docker-hub/vulnerability-scanning/) (paid subscription) to scan all newly pushed images automatically

#### Image layering

- Use the `docker image history` command to see the command that was used to create each layer within an image
  - see the size of each layer to help diagnose large images

```console
$ docker image history getting-started
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
2c0bb796933b   10 days ago    /bin/sh -c #(nop)  EXPOSE 3000                  0B        
8c97124a9f9e   10 days ago    /bin/sh -c #(nop)  CMD ["node" "src/index.js…   0B        
5d27ac92cb88   10 days ago    /bin/sh -c yarn install --production            86MB      
d463bb69b63d   10 days ago    /bin/sh -c #(nop) COPY dir:c65361744edcce5e1…   4.62MB    
805e1ef0150d   10 days ago    /bin/sh -c #(nop) WORKDIR /app                  0B        
1391e56b4519   10 days ago    /bin/sh -c apk add --no-cache python2 g++ ma…   223MB     
bb6d28039b8c   3 months ago   /bin/sh -c #(nop)  CMD ["node"]                 0B        
<missing>      3 months ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      3 months ago   /bin/sh -c #(nop) COPY file:4d192565a7220e13…   388B      
<missing>      3 months ago   /bin/sh -c apk add --no-cache --virtual .bui…   7.84MB    
<missing>      3 months ago   /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.18     0B        
<missing>      3 months ago   /bin/sh -c addgroup -g 1000 node     && addu…   77.6MB    
<missing>      3 months ago   /bin/sh -c #(nop)  ENV NODE_VERSION=12.22.12    0B        
<missing>      4 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B        
<missing>      4 months ago   /bin/sh -c #(nop) ADD file:5d673d25da3a14ce1…   5.57MB
```

- Each of the lines represents a layer in the image
  - the base at the bottom and the newest layer at the top
  - add the `--no-trunc` flag to get the full output of truncated lines
- Image layers
  - each layer is an image itself 
  - each layer stores the changes compared to the image it's based on
  - an image can consist of a single layer
  - each instruction in a Dockerfile results in a layer
  - except for multi-stage builds, where usually only the layers in the final image are pushed, or when an image is squashed to a single layer
  - layers are used to avoid transferring redundant information and skip build steps which have not changed (according to the Docker cache)

#### Layer caching

- Original [Dockerfile](app/Dockerfile):

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

- Once a layer changes, all downstream layers have to be recreated as well
- When we made a change to the image, the yarn dependencies had to be reinstalled
  - to avoid this, restructure the Dockerfile to help support the caching of the dependencies
  - for Node-based applications, those dependencies are defined in the `package.json` file
  - if we copied only that file in first, install the dependencies, and then copy in everything else, then we only recreate the yarn dependencies if there was a change to the `package.json`

- Updated Dockerfile, [Dockerfile_cachedep:](app/Dockerfile_cachedep)

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --production
COPY . .
CMD ["node", "src/index.js"]
EXPOSE 3000
```

- Create an updated Dockerfile as above to copy in the `package.json` first, install dependencies, and then copy everything else in
- Create a file named `.dockerignore` in the same folder as the Dockerfile with the following contents
  - to selectively copy only image relevant files
  - in this case, the `node_modules` folder should be omitted in the second `COPY` step because otherwise, it would possibly overwrite files which were created by the command in the `RUN` step
  - see also
    - <https://docs.docker.com/engine/reference/builder/#dockerignore-file>
    - <https://nodejs.org/en/docs/guides/nodejs-docker-webapp/>

```dockerignore
node_modules
```

- Build a new image using `docker build`

```console
$ docker build -f Dockerfile_cachedep -t getting-started .
Sending build context to Docker daemon  4.658MB
Step 1/8 : FROM node:12-alpine
 ---> bb6d28039b8c
Step 2/8 : RUN apk add --no-cache python2 g++ make
 ---> Using cache
 ---> 1391e56b4519
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> 805e1ef0150d
Step 4/8 : COPY package.json yarn.lock ./
 ---> e8ea77e6ddea
Step 5/8 : RUN yarn install --production
 ---> Running in 151fddd00d24
yarn install v1.22.18
[1/4] Resolving packages...
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^2.0.0"
warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^3.0.0"
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 7.05s.
Removing intermediate container 151fddd00d24
 ---> 7a3dc7870fbf
Step 6/8 : COPY . .
 ---> af089a383d0c
Step 7/8 : CMD ["node", "src/index.js"]
 ---> Running in 6774af7a5bef
Removing intermediate container 6774af7a5bef
 ---> a75576978ce4
Step 8/8 : EXPOSE 3000
 ---> Running in 9065e03211d5
Removing intermediate container 9065e03211d5
 ---> a47397c367e3
Successfully built a47397c367e3
Successfully tagged getting-started:latest
```

- Make a change to the `src/static/index.html` file (like change the `<title>` to say “The Awesome Todo App”).
- Build the Docker image again
  - the build should be much faster with more steps having `Using cache`

```console
$ docker build -f Dockerfile_cachedep -t getting-started .
Sending build context to Docker daemon  4.658MB
Step 1/8 : FROM node:12-alpine
 ---> bb6d28039b8c
Step 2/8 : RUN apk add --no-cache python2 g++ make
 ---> Using cache
 ---> 1391e56b4519
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> 805e1ef0150d
Step 4/8 : COPY package.json yarn.lock ./
 ---> Using cache
 ---> e8ea77e6ddea
Step 5/8 : RUN yarn install --production
 ---> Using cache
 ---> 7a3dc7870fbf
Step 6/8 : COPY . .
 ---> c7b8ddb50225
Step 7/8 : CMD ["node", "src/index.js"]
 ---> Running in 901983261ac2
Removing intermediate container 901983261ac2
 ---> 7a88eaf07152
Step 8/8 : EXPOSE 3000
 ---> Running in bad107bc00df
Removing intermediate container bad107bc00df
 ---> 0ed65bc88451
Successfully built 0ed65bc88451
Successfully tagged getting-started:latest
```

#### Multi-stage builds

- Multi-stage builds are a tool to help use multiple stages to create an image
  - separate build-time dependencies from runtime dependencies
  - reduce overall image size by shipping only what your app needs to run
- Maven/Tomcat example
  - when building Java-based applications, a JDK is needed to compile the source code to Java bytecode - not needed in production
  - you might be using tools like Maven or Gradle to help build the app - not needed in the final image
  - multi-stage build:
    - use one stage (called `build`) to perform the actual Java build using Maven
    - in the second stage (starting at `FROM tomcat`), copy in files from the `build` stage
    - the final image is only the last stage being created (which can be overridden using the `--target` flag)

```dockerfile
# syntax=docker/dockerfile:1
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps
```

- React example
  - when building React applications, we need a Node environment to compile the JS code (typically JSX), SASS stylesheets, and more into static HTML, JS, and CSS
  - if we aren’t doing server-side rendering, we don't need a Node environment for our production build
  - ship the static resources in a static nginx container

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

## Getting started with Java

- The Java getting started guide teaches you how to create a containerized Spring Boot application using Docker
  - clone and run a Spring Boot application with Maven
  - create a new Dockerfile which contains instructions required to build a Java image
  - run the newly built image as a container
  - set up a local development environment to connect a database to the container
  - use Docker Compose to run the Spring Boot application
  - configure a CI/CD pipeline for your application using GitHub Actions
  - deploy your application to the cloud

### Build your Java image

#### Enable BuildKit

- Ensure you have enabled BuildKit on your machine
  - BuildKit allows you to build Docker images efficiently
  - see <https://docs.docker.com/develop/develop-images/build_enhancements/>
- BuildKit is enabled by default for all users on Docker Desktop
  - you don’t have to manually enable BuildKit
- If you are running Docker on Linux, you can enable BuildKit either by using an environment variable or by making BuildKit the default setting
- To set the BuildKit environment variable:

```console
$ DOCKER_BUILDKIT=1
```

- To enable docker BuildKit by default:
  - set daemon configuration in `/etc/docker/daemon.json` feature to `true` and restart the daemon
  - if the `daemon.json` file doesn't exist, create it and then add the following to the file and restart the Docker daemon

```json
{
  "features": {
    "buildkit" : true
  }
}
```

#### Test the application without Docker

- The sample application is located in [`spring-petclinic`](spring-petclinic)
  - requires Java OpenJDK version 15 or later
  - contains an embedded version of Maven
- Run the following command in the application directory:

```console
$ ./mvnw spring-boot:run
```

- Open a new browser and navigate to <http://localhost:8080>

#### Create a Dockerfile for Java

- The [Dockerfile](spring-petclinic/Dockerfile)

#### Create a `.dockerignore` file

- Create a [`.dockerignore` file](spring-petclinic/.dockerignore) to exclude the `target` directory, which contains output from Maven, from the Docker build context
  - increase the performance of the build, and as a general best practice

#### Build an image

- Run `docker build --tag java-docker .`

```console
$ docker build --tag java-docker .
[+] Building 375.5s (15/15) FINISHED                                                              
 => [internal] load build definition from Dockerfile                                         0.0s
 => => transferring dockerfile: 603B                                                         0.0s
 => [internal] load .dockerignore                                                            0.0s
 => => transferring context: 34B                                                             0.0s
 => resolve image config for docker.io/docker/dockerfile:1                                   0.9s
 => CACHED docker-image://docker.io/docker/dockerfile:1@sha256:443aab4ca21183e069e7d8b2dc68  0.0s
 => [internal] load .dockerignore                                                            0.0s
 => [internal] load build definition from Dockerfile                                         0.0s
 => [internal] load metadata for docker.io/library/openjdk:16-alpine3.13                     0.0s
 => [1/6] FROM docker.io/library/openjdk:16-alpine3.13                                       0.0s
 => [internal] load build context                                                            0.0s
 => => transferring context: 80.45kB                                                         0.0s
 => CACHED [2/6] WORKDIR /app                                                                0.0s
 => [3/6] COPY .mvn/ .mvn                                                                    0.0s
 => [4/6] COPY mvnw pom.xml ./                                                               0.1s
 => [5/6] RUN ./mvnw dependency:go-offline                                                 373.1s
 => [6/6] COPY src ./src                                                                     0.1s
 => exporting to image                                                                       1.1s 
 => => exporting layers                                                                      1.1s 
 => => writing image sha256:4ed2da81e4b3e5cae6dfebf0a3f1d1db283450b355f3a7532332926a3d94238  0.0s 
 => => naming to docker.io/library/java-docker                                               0.0s 
```

#### Build image failure with `mvnw dependency:go-offline`

- When building the image (Spring PetClinic 2.7.0-SNAPSHOT with Spring Boot 2.7.1) using the original implementation from <https://github.com/spring-projects/spring-petclinic> with `RUN ./mvnw dependency:go-offline` in the Dockerfile, the build fails
  - the following is the output (with BuildKit disabled)

```text
Downloaded from central: https://repo.maven.apache.org/maven2/org/ehcache/ehcache/3.10.0/ehcache-3.10.0.jar (1.8 MB at 4.9 MB/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  05:32 min
[INFO] Finished at: 2022-08-04T00:45:08Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-dependency-plugin:3.3.0:go-offline (default-cli) on project spring-petclinic: org.eclipse.aether.resolution.DependencyResolutionException: Failed to collect dependencies at org.ehcache:ehcache:jar:3.10.0
-> org.glassfish.jaxb:jaxb-runtime:jar:2.3.0-b170127.1453
-> org.glassfish.jaxb:jaxb-core:jar:2.3.0-b170127.1453
-> javax.xml.bind:jaxb-api:jar:2.3.0-b161121.1438: Failed to read artifact descriptor for javax.xml.bind:jaxb-api:jar:2.3.0-b161121.1438: Could not transfer artifact javax.xml.bind:jaxb-api:pom:2.3.0-b161121.1438 from/to maven-default-http-blocker (http://0.0.0.0/): Blocked mirror for repositories: [releases.java.net (http://maven.java.net/content/repositories/releases/, default, releases+snapshots), shapshots.java.net (http://maven.java.net/content/repositories/snapshots/, default, releases+snapshots), jvnet-nexus-staging (http://maven.java.net/content/repositories/staging/, default, releases+snapshots), netbeans (http://bits.netbeans.org/nexus/content/groups/netbeans, default, releases)] -> [Help 1]
```

- This is due to Maven 3.8.1 onwards [blocking downloads](https://maven.apache.org/docs/3.8.1/release-notes.html#cve-2021-26291) from repositories that refer to a URL over HTTP to prevent Man-In-The-Middle-Attack
  - Maven 3.8.2 was used above
- A [workaround](https://github.com/spring-projects/spring-petclinic/issues/996) for the purpose of the tutorial involves creating [repository mirrors](https://maven.apache.org/guides/mini/guide-mirror-settings.html) that use HTTP URLs:
  - [`spring-petclinic/.mvn/maven.config`](spring-petclinic/.mvn/maven.config)
  - [`spring-petclinic/.mvn/local-settings.xml`](spring-petclinic/.mvn/local-settings.xml)
- However, applying the workaround results in a different build failure

```text
Downloaded from central: https://repo.maven.apache.org/maven2/org/ehcache/ehcache/3.10.0/ehcache-3.10.0.jar (1.8 MB at 4.9 MB/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  06:23 min
[INFO] Finished at: 2022-08-04T01:21:25Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-dependency-plugin:3.3.0:go-offline (default-cli) on project spring-petclinic: org.eclipse.aether.resolution.DependencyResolutionException: Failed to collect dependencies at org.ehcache:ehcache:jar:3.10.0 -> org.glassfish.jaxb:jaxb-runtime:jar:2.3.0-b170127.1453 -> org.glassfish.jaxb:jaxb-core:jar:2.3.0-b170127.1453 -> javax.xml.bind:jaxb-api:jar:2.3.0-b161121.1438: Failed to read artifact descriptor for javax.xml.bind:jaxb-api:jar:2.3.0-b161121.1438: Could not transfer artifact javax.xml.bind:jaxb-api:pom:2.3.0-b161121.1438 from/to netbeans-ssl (https://netbeans.apidesign.org/maven2/): transfer failed for https://netbeans.apidesign.org/maven2/javax/xml/bind/jaxb-api/2.3.0-b161121.1438/jaxb-api-2.3.0-b161121.1438.pom: Unknown host netbeans.apidesign.org: Try again -> [Help 1]
```

- This new problem occurs because none of the repositories, including the above mirrors, contains `javax/xml/bind/jaxb-api/2.3.0-b161121.1438/jaxb-api-2.3.0-b161121.1438.pom`
- A further [workaround](https://stackoverflow.com/a/72150668) involves adding `<dependencyManagement>` to [`pom.xml`](spring-petclinic/pom.xml) so that `jaxb-api 2.3.1` is used instead of the above problematic version

### Run your image as a container

- Run the image
  - `docker run -d -p 8080:8080 --name springboot-server java-docker`
    - `--name` gives the container a specific, otherwise a random name will be assigned
- Restart the container
  - `docker restart springboot-server`
- Stop the container
  - `docker stop springboot-server`
- Start the container
  - `docker start springboot-server`

## Sources

- "Docker Overview." _Docker Documentation_, 18 July 2022, [docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/). Accessed 18 July 2022.
- "Install Docker Engine on Ubuntu." _Docker Documentation_, 20 July 2022, [docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/). Accessed 20 July 2022.
- "Post-Installation Steps for Linux." _Docker Documentation_, 20 July 2022, [docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/). Accessed 20 July 2022.
- "Sample Application." _Docker Documentation_, 22 July 2022, [docs.docker.com/get-started/02_our_app/](https://docs.docker.com/get-started/02_our_app/). Accessed 23 July 2022.
- "Update the Application." _Docker Documentation_, 22 July 2022, [docs.docker.com/get-started/03_updating_app/](https://docs.docker.com/get-started/03_updating_app/). Accessed 24 July 2022.
- "Share the Application." _Docker Documentation_, 22 July 2022, [docs.docker.com/get-started/04_sharing_app/](https://docs.docker.com/get-started/04_sharing_app/). Accessed 24 July 2022.
- "Persist the DB." _Docker Documentation_, 25 July 2022, [docs.docker.com/get-started/05_persisting_data/](https://docs.docker.com/get-started/05_persisting_data/). Accessed 26 July 2022.
- "Use Bind Mounts." _Docker Documentation_, 27 July 2022, [docs.docker.com/get-started/06_bind_mounts/](https://docs.docker.com/get-started/06_bind_mounts/). Accessed 28 July 2022.
- "Multi Container Apps." _Docker Documentation_, Aug. 2022, [docs.docker.com/get-started/07_multi_container/](https://docs.docker.com/get-started/07_multi_container/). Accessed 1 August 2022.
- "Use Docker Compose." _Docker Documentation_, Aug. 2022, [docs.docker.com/get-started/08_using_compose/](https://docs.docker.com/get-started/08_using_compose/). Accessed 2 August 2022.
- "Image-Building Best Practices." _Docker Documentation_, 3 Aug. 2022, [docs.docker.com/get-started/09_image_best/](https://docs.docker.com/get-started/09_image_best/). Accessed 3 August 2022.
- "What Are Docker Image Layers?" _Vsupalov.com_, 15 Feb. 2021, [vsupalov.com/docker-image-layers/](https://vsupalov.com/docker-image-layers/). Accessed 3 August 2022.
- "Build Your Java Image." _Docker Documentation_, 4 Aug. 2022, [docs.docker.com/language/java/build-images/](https://docs.docker.com/language/java/build-images/). Accessed 5 August 2022.
- "Dockerfile Reference." _Docker Documentation_, 22 July 2022, [docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/). Accessed 23 July 2022.
- "Compose Specification." _Docker Documentation_, Aug. 2022, [docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/). Accessed 2 August 2022.
