# docker-mastery-course
Notes, code, etc, from Docker Mastery course I took on Udemy

Course URL: https://www.udemy.com/course/docker-mastery/

## Useful Docker CLI commands

NOTE: Docker has introduced a new command structure: `docker <command> <sub-command>`

* `docker version`
* `docker info`
* `docker container run ...OTHER-CONFIG-PARAMS`
  * Runs a **new** container instance
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
* `docker container top <container-name>`
  * Similar to UNIX 'top' command
  * Displays info on processes running inside the container
* `docker container inspect <container-name>`
* `docker container stats`
* `docker image ls -a`
  * List all images on my local machine
  * The `-a` flag means all - otherwise intermediate images are not displayed


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
