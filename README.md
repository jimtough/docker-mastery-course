# docker-mastery-course
Notes, code, etc, from Docker Mastery course I took on Udemy

Course URL: https://www.udemy.com/course/docker-mastery/

----

# miscellaneous notes

## Useful Docker CLI commands

NOTE: Docker has introduced a new command structure: `docker <command> <sub-command>`

* `docker version`
* `docker info`
* `docker container run ...OTHER-CONFIG-PARAMS`
  * Runs a **new** container instance
* `docker container run -rm ...OTHER-CONFIG-PARAMS`
  * Runs a **new** container instance, and automatically removes the container after it has been stopped
* `docker container start ...OTHER-CONFIG-PARAMS`
  * Restart an existing container instance that is not currently running
* `docker container ls -a`
  * List all containers (running OR stopped)
* `docker container stop <container-name>`
  * Stops a running container but does NOT remove it (use `docker container ls` to see your running containers list)
* `docker container rm <container-names...>`
  * Remove a previously stopped container
  * Use the `-f` option to force running containers to stop, then remove
  * You can also use the first few digits of the container hex-based id instead of container names, as long as the values are unique
* `docker container top <container-name>`
  * Similar to UNIX 'top' command
  * Displays info on processes running inside the container
* `docker container port <container-name>`
  * Display port mappings for this container
* `docker container inspect <container-name>`
* `docker container stats`
* `docker image ls -a`
  * List all images on my local machine
  * The `-a` flag means all - otherwise intermediate images are not displayed
* `docker image pull <image-name>:<image-tag>`
  * Download an image from Docker Hub to my local machine
* `docker network ls`
* `docker network inspect`
* `docker network create --driver <driver>`
* `docker network disconnect`


## other useful tips

* CTRL-C
  * On Windows - puts a foreground container into the background (you need to then stop the container with another command)
* docs.docker.com
  * Good resource for CLI command details, etc.
* Periodically run the 'prune' command to remove things from your local machine that are no longer being used
  * `docker system prune`
  * `docker image prune`
  * `docker image prune -a`
* Check on Docker disk usage on your local workstation:
  * `docker system df`


----


# General Course Notes

## image versus container

* image - the application we want to run
* container - one instance of an image that is running as a process
  * You can have many containers running the same image
  * Docker's default image "registry" (repository of images) is Docker Hub (hub.docker.com)

## What's in an Image? (and what isn't?)

* App binaries and dependencies
* Metadata about the image data and how to run the image
* Official definition:
  * "An Image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime."
* An image is **NOT** a complete operating system!
  * Does not provide a kernel
  * Does not provide kernel modules (example: drivers)
  * The **host** provides the kernel! This is a big difference between Docker images and traditional "virtual machine" images.

## Container Lifetime and Persistent Data

* Containers are (usually) immutable and ephemeral
* "immutable infrastructure" - only re-deploy containers, never change them
* "unique data" is a separate concern from the container
* Docker provides two solutions for persistent data:
  * Volumes - make a special location outside of the container's UFS (union file system) that is not part of the container's lifecycle
  * Bind Mounts - link container path to a path on the container's host

## Docker Compose

* Used to configure and run one or more Docker containers, including configuration of dependencies between the containers, Docker virtual networks, etc
* Configuration is defined in a YAML file
  * Default file name is 'docker-compose.yml'
  * File format reference: https://docs.docker.com/compose/compose-file/
  * There are multiple version of the YAML file (course covers up to version spec 3.1)


----


# Course Labs and Exercises

## lab - start an nginx web server in a container

In this lab, the image will be for an Nginx web server

Command to start the container in the foreground: `docker container run --publish 80:80 nginx`

Wait for Docker to download the 'nginx' image and start a new container. I should see feedback in PowerShell as this is happening. After the startup is complete, go here: http://localhost/

I should see a "Welcome to nginx!" in my browser.

What did this do?

* Downloaded 'nginx' image from Docker Hub (the default image repo)
* Started a new container using that image
* Opened port 80 on the host
* Routes traffic on host port 80 to the Docker container, port 80

Command to start the container in the background:  
`docker container run --publish 80:80 --detach nginx`

If you want to see the console output of a container that is running in the background, use this command:  
`docker container logs <container-name>`

### overriding the default behaviour with more command line options

The `docker container run` command applies some default behaviours if we don't explicitly provide values. Here is an example related to the nginx lab above.

* change listening port to 8080
* specify the container name as 'webhost'
* use image version 1.11 instead of the default 'latest'
* change startup command instead of using command specified in image's Dockerfile

`docker container run --publish 8080:80 --name webhost -d nginx:1.11 nginx -T`

----

## lab - getting a shell inside containers (without SSH)

This lab explores alternatives to adding SSH support to your container/image.

### Run container with `-it` option

This is actually two separate CLI options:
* -i : interactive mode (keeps STDIN open even when nobody is connected)
* -t : enable pseudo-TTY

Try this as an example:  
`docker container run -it --name proxy nginx bash`

In the command above, 'bash' is executed as a command inside the container, opening a new bash shell. This overrides any default commands that were defined in the Dockerfile for this image!

Because we started the container in `-it` mode, our command line will be left connected to the bash shell. When you're done issing commands inside the container, type `exit` to exit the bash shell and the interactive mode.

NOTE: In the example above, we did not run in `--detach` mode. When you exit interactive mode (exit the shell) the container will stop, because there is nothing running after you exit bash.

### Start an existing container with `-ai` option

This is actually two separate CLI options:
* -a : attach STDOUT/STDERR
* -i : attach container's STDIN

Try this example on the stopped container from above:  
`docker container start -ai proxy`

The container should be in the exact state that you left it when you stopped it earlier.

### Run 'bash' in an existing container with `-it` option

You can execute a command inside a running container with `docker container exec`. When combined with the `-it` option, you can execute a shell command and then just use the shell as you normally would on a server.

Example that executes a bash shell inside a running container:  
`docker container exec -it nginx bash`

----

## lab - Docker Networks

### network basics

Example command to obtain a running container's internal IP address:
`docker container inspect --format "{{.NetworkSettings.IPAddress}}" nginx`

* By default, your local Docker install will create its own virtual network with a name like 'bridge' or 'docker0'. All your locally running containers will use that virtual network unless you explicitly specify otherwise.
* When you map ports to containers with `-p` such as `-p 80:8080`, Docker will expose the first port number (80) on your local machine, and direct all received traffic to the second port number (8080) on the virtual network.
* You can create more virtual networks locally, but you must do so explicitly.
* You can expose ports inside your virtual network such that 2 or more containers can communicate with each other, but never be exposed to the outside world.
  * example: 'httpd' container (exposing only its own port 80) connecting to fully private 'mysql' container on a port that is never exposed to the outside world

### CLI Management of networks

Command to list available networks:  
`docker network ls`

Command to display details of the 'bridge' network:  
`docker network inspect bridge`

Create a new virtual network and name it 'my_app_net':  
`docker network create my_app_net`

Start a new container and connect it to virtual network 'my_app_net':  
`docker container run  -d --name another_nginx --network my_app_net nginx:alpine`

Connect a running container to an existing virtual network:  
`docker network connect bridge another_nginx`
* bridge - the network I'm connecting to
* another_nginx - the running container

Disconnect a running container from a virtual network:  
`docker network disconnect bridge another_nginx`

### DNS and Docker Networks

* Static IPs and using IPs for talking to containers is an anti-pattern. Do your best to avoid it.
* Docker daemon has a built-in DNS server that containers use by default.
* Docker defaults the hostname to the container's name, but you can also set up aliases.
  * This only works if you use a custom network, rather than the default 'bridge' network!

Have two nginx containers ping each other using their own Docker-assigned DNS names (works in both directions):  
* `docker container run -p 4321:80 -d --name my_nginx --network my_app_net nginx:alpine`
* `docker container run -p 1234:80 -d --name new_nginx --network my_app_net nginx:alpine`
* `docker container exec -it new_nginx ping my_nginx`
* `docker container exec -it my_nginx ping new_nginx`

### DNS Round Robin

It is possible to configure a Docker network such that:  
* Two containers have the same (explicitly assigned) DNS name alias
* Docker will do a DNS Round Robin when requests for that DNS name alias are received

There is a short exercise on this in the course, but I've skipped it for now. I'll come back to it if I really need this feature.

----

## lab - Docker Hub Registry Images

Docker Hub site: https://hub.docker.com

You'll need to sign up for a free user account.

* During development, it is common to allow Docker to default to using the 'latest' version of an image. Before you go to production, it is highly recommended that you specify an explicit image version tag so your builds will be repeatable.
* Many images have 'alpine' in the name. Alpine is a very small Linux distribution that is popular due to its minimal footprint. Images with 'alpine' in the name are built on top of a base Alpine Linux.

## lab - Images and their Layers

Command to see the 'history' of a specified image:  
`docker image history nginx`

* The history shows the layers of that image.
* Every image starts with a blank layer known as 'scratch'.
* Every change that happens to that image creates another layer.
* Some changes add to the layer size (adding files), while some do not (executing a command such as 'bash' to open a shell - this is known as adding metadata).
* If two images build on top of the same base layer, you only need to store one copy of that base layer in your image cache and on the container host server.

Command to view the details of a specified image:  
`docker image inspect nginx`

This 'image inspect' command returns a JSON string. This is where image metadata can be viewed.

## lab - Image Tagging and Pushing to Docker Hub

Non-official images are identified using a string in this format:  
`<user>/<repo>:<tag>`  

Official (Docker Hub) images omit the user, and look like this:  
`<repo>:<tag>`

The tag 'latest' is a pseudo-tag that is similar to HEAD in a Git repo. Image owners **should** associate it with the most recent stable release of their image, but nothing forces them to do so! If you don't specify any tag in your image-related commands, then 'latest' is used as the default.

The 'user' is the name of the Docker Hub account that uploaded that image.

Example of adding your own tag to an existing image:  
`docker image tag nginx bretfisher/nginx`  

(Bret Fisher is the course instructor)

Command to upload an image to a repo (Docker Hub is the default repo):  
`docker image push bretfisher/nginx`

NOTE: The 'image push' command will only succeed if you're current logged in to the target repo (in this case Docker Hub) on your workstation. 

* Your local install of Docker can be configured to automatically 'login' to Docker Hub at startup.
* The Docker CLI has both 'login' and 'logout' commands that can be used.
* These methods can also be used to authenticate/login/logout to remote repos other than Docker Hub.

**BEST PRACTICE** - If you use the Docker CLI to `login` on a machine that you don't trust, then remember to `logout` when you're finished. This will remove your locally cached credentials from the user profile on that machine.

## lab - 'Dockerfile' Basics

* 'Dockerfile' (with a capital D) is a file that contains the recipe for building an image.
* The file named 'Dockerfile' is the default that Docker expects. If you want to use a file with a different name, then you need to explicitly specify its name.
* Each command in the Dockerfile is executed in order from top to bottom of the file. The order of execution is important!

### FROM command

This command is required in every Dockerfile.

* It specifies the base image that you are building on top of.
* When you 'FROM' an image, you inherit all the commands from the Dockerfile of that base image.

**TIP:** Package managers like 'apt' and 'yum' are one of the reasons to build containers from Debian, Ubuntu, Fedora or CentOS

### ENV section

Set environment variables in the container when this image is run.

### RUN commands

* There can be zero or more of these in your Dockerfile.
* These are commonly used to call Package Managers that download and install packages on the container when the image is run.

**BEST PRACTICE** - Chain Package Manager commands together in your RUN command with &&'s. Each RUN command in the Dockerfile creates a new image layer, so it is best to create fewer layers when possible.

**BEST PRACTICE** - Add a RUN command that redirects your application logs to `/dev/stdout` and `/dev/stderr` rather than physical files. This allows Docker to capture the log output and send it elsewhere.

### EXPOSE command

* Expose ports on the Docker virtual network to the container.
* This is NOT the same as exposing those ports to the host machine and the outside world - the `-p` option is used for that when you start the container.


### CMD command

This command is required in every Dockerfile.

* This is the command that will be executed whenever a new container is launched using this image, or when a stopped container is restarted.

## lab - Building Images - Running Docker Builds

**TIP** - If you are copying an application build into the image as a step in your Dockerfile, put that step as close to the end as possible. This will allow Docker to re-use existing (cached) intermediate layers of your image on subsequent builds. Only layers after the last change need to be rebuilt.

* Things that change the least should go at the top of your Dockerfile. Things that change the most should go at the bottom.

### Dockerfile WORKDIR command

* Change the current working directory before executing the next Dockerfile step.
* Using `RUN cd /some/path` is considered bad practice.

### Dockerfile COPY command

* Copies files from your workstation (or build server) into the container.
* This is how you add your application build to the container, for example.

### Building your image (locally)

`docker image build -t my-new-image-tag`

* Run this command from the same directory where your Dockerfile is located.
* Run your image with `docker container run my-new-image-tag`

----

## lab - Persistent Data: Volumes

Command to view local volumes:  
`docker volume ls`

Useful command to clean up leftover volumes from previous labs:  
`docker volume prune`

Lab commands:  
* `docker pull mysql`
* `docker image inspect mysql`

You can see in the JSON output that this image has a volume defined for a particular path:
```
            "Volumes": {
                "/var/lib/mysql": {}
            },
```

* `docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql`
* `docker container ls`
* `docker container inspect mysql`

You can see in the JSON output that this container has mounted the volume defined in the image to the default local volume:
```
        "Mounts": [
            {
                "Type": "volume",
                "Name": "9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1",
                "Source": "/var/lib/docker/volumes/9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

* `docker volume ls`
* `docker volume inspect <volume-name>`

```
[
    {
        "CreatedAt": "2020-06-25T13:54:43Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1/_data",
        "Name": "9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1",
        "Options": null,
        "Scope": "local"
    }
]
```

* `docker container run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql`
* `docker volume ls`

Each container instance of 'mysql' image is assigned a new local volume with a unique, but not-user-friendly, volume name.
```
DRIVER              VOLUME NAME
local               9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1
local               f43b713e8ee336fb0f278d3d46565fe6547496c08d31a0ed2f4bdd21074bafe7
```

* `docker container rm -f mysql mysql2`
* `docker volume ls`
  * The containers are gone, but the volumes still exist

Let's make these volumes easier to manage by using 'named volumes'.

* `docker container run -d --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql1-db:/var/lib/mysql mysql`
* `docker container run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql2-db:/var/lib/mysql mysql`
* `docker volume ls`

```
DRIVER              VOLUME NAME
local               9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1
local               f43b713e8ee336fb0f278d3d46565fe6547496c08d31a0ed2f4bdd21074bafe7
local               mysql1-db
local               mysql2-db
```

* `docker container rm -f mysql1 mysql2`
* `docker volume ls`
* `docker container run -d --name mysql1B -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql1-db:/var/lib/mysql mysql`
* `docker container run -d --name mysql2B -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql2-db:/var/lib/mysql mysql`
* `docker volume ls`

The newly created containers will reuse the same named volumes
```
DRIVER              VOLUME NAME
local               9ad6e7b5dd6286945e6e78925d4eb0b89fdd6bd5bcce70ad81d2bc9407ea15c1
local               f43b713e8ee336fb0f278d3d46565fe6547496c08d31a0ed2f4bdd21074bafe7
local               mysql1-db
local               mysql2-db
```

NOTE: It is possible to create a volume ahead of time using `docker volume create <options>`. This is usually done on production environments and/or when a special volume driver is required. This is rarely done on a local development workstation.

### Dockerfile VOLUME command

* `VOLUME /path/in/container`

----

## lab - Persistent Data: Bind Mounting

* A 'bind mount' maps a host file or directory to a container file or directory
* This bypasses UFS and the host files overwrite any in the container
  * host files have precedence while bind mount is active
* Bind mounts CANNOT be defined in a Dockerfile! They must be specified at `container run` time.
  * `... run -v //c/Users/bret/stuff:/path/in/container` (Windows host)
  * `... run -v /Users/bret/stuff:/path/in/container` (Linux host)

Use this command (in Windows PowerShell) to bind mount my current working directory to the nginx container path `/usr/share/nginx/html`:  
`docker container run -d --name nginx -p 80:80 -v ${pwd}:/usr/share/nginx/html nginx`

----

## lab - Trying out basic Docker Compose commands

### docker-compose CLI
* Comes packaged with Docker for Windows/Mac
* Not a production-grade tool, but ideal for local development and testing
* Two most common commands:
  * `docker-compose up` (setup volumes/networks and start all containers)
  * `docker-compose down` (stop all containers and remove all containers/networks)
* More commands:
  * `docker-compose up -d` (run in detached mode)
  * `docker-compose ps`
  * `docker-compose top`
  * `docker-compose down -v` (stop all containers and remove all containers/networks AND volumes!)

## lab - Adding Image Building to docker-compose files

* Docker Compose supports building a custom image using commands defined in your docker-compose.yml file
  *  Can be triggered automatically when `docker-compose up` is executed if the image does not exist yet
  * Must be triggered explicitly with `docker-compose up --build` or `docker-compose build` if you want to replace the existing image
* See instructor example in 'examples/bretfisher/compose-sample-3'




























