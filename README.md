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

- A container is a runnable instance of an image
- You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine
- A container is defined by its image as well as any configuration options you provide to it when you create or start it
- When a container is removed, any changes to its state that are not stored in persistent storage disappear

### Example `docker run` command

```shell
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