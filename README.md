# docker-mastery-course
Notes, code, etc, from Docker Mastery course I took on Udemy

Course URL: https://www.udemy.com/course/docker-mastery/

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
* `docker network ls`
* `docker network inspect`
* `docker network create --driver <driver>`
* `docker network disconnect`


### other useful tips

* CTRL-C
  * On Windows - puts a foreground container into the background (you need to then stop the container with another command)
* docs.docker.com
  * Good resource for CLI command details, etc.



----

# Course Notes

## image versus container

* image - the application we want to run
* container - one instance of an image that is running as a process
  * You can have many containers running the same image
  * Docker's default image "registry" (repository of images) is Docker Hub (hub.docker.com)


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

















