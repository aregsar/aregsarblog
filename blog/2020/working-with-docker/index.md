# Working With Docker

December 31, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail various docker commands and concepts that I use frequently to manage docker containers in my local development environment.

## Common commands when working with containers

```bash
#list all stopped and running containers (ommiting the flag will list only running containers)
docker ps -a

#list all images on host machine
docker images

#stop one or more running containers by name or id
#when running a container with the --rm flag, stoping the container will auto remove it
docker stop container_name_or_id

#remove one or more stopped containers by name or id
docker rm container_name_or_id

#stop all running containers
docker stop $(docker ps -q)

#remove all stoped containers
docker container prune -f

#remove all images (either command will work)
docker image prune -a -f
docker rmi $(docker images -q)

#show all docker stats about images and containers on the host
docker system info

#removes unused docker resources. see help for usage
docker system prune --help
```

## Listing available commands

```bash
#list available container commands
docker container

#list available image commands
docker image

#list available volume commands
docker volume

#list available network commands
docker network
```

Example of running the available ls and prune network commands

```bash
#list all the networks
docker network ls
#prune unused networks
docker network prune
```

## The inspect command

The inspect command dumps the configuration of images and containers

```bash
#inspect the build time configuration an image
docker inspect image_name_or_id

#inspect the runtime configuration of a container
docker inspect container_name_or_id
```

Examples of inspecting images:

```bash
docker inspect nginx
docker inspect nginx:latest
docker inspect library/nginx:latest
docker inspect library/nginx
docker inspect ubuntu:20.04
docker inspect library/ubuntu:20.04
```

## Running a container

I will illustrate running a container using the docker ubuntu image.

Simplest form of running a container:

```bash
docker run ubuntu
```

This is equivalent as running with the default tag:

```bash
docker run ubuntu:latest
```

which is equivalent to running with default namespace:

```bash
docker run libarary\ubuntu
```

which is equivalent to running with the default tag and namespace:

```bash
docker run libarary\ubuntu:latest
```

Generally we can omit the namespace unless we have a specific namespace for our organization.

However it is a good practice to use a specific tag. This is because specifying `latest` or omitting the tag which is the equivalent to specifying `latest` may cause a different version of the image to be used depending on when we run the command.

Using a specific tag ensures that we are always using the same image.

Running the ubuntu container with the ubuntu version tag:

```bash
docker run ubuntu:20.10
```

This is equivalent to running with the default namespace

```bash
docker run library\ubuntu:20.10
```

Finally note that we did not type a command after the image name, in any of the examples. This will cause the default Entrypoint\Command that the image is configured with to run.

In the case of ubuntu, the command that the container runs is the bash command. This will run the bash shell as the main docker process PID 1 and will exit the process automatically because there is no bash command to run. This in turn will will automatically stop the running container.

## Docker run command options

Generally we run containers with a few standard options.

I will demonstrate by way of examples:

The --rm tag to auto remove the container after it has been stoped.

```bash
docker run --rm ubuntu:20.10
```

When we run a container it gets assigned a container id, even after it is stoped.

The name tag gives the running or stopped container a name we can reference it with instead of the container id.
We can only have one container with that specific name running at one time, so docker run will show an error if we try using the same name twice.

```bash
docker run --rm --name zubuntu ubuntu:20.10
```

The -it option runs the container in interactive mode (the i option) and communicates with container using the tty teletype command line interface (the t option). Both options are combined together to allow you to interact with the running container.

```bash
docker run --rm --name zubuntu -it ubuntu:20.10
```

Running this command will put you in the ubuntu container. To exit the container, type `exit`.

The order or the options does not matter:

```bash
docker run --rm -it --name zubuntu ubuntu:20.10
```

## Running containers in the background

The -d option can be used instead of the -it option to run the container in detached mode.

By default running the container without the -d option, runs the container in the foreground, so control will not return to the terminal if the container continues to run.

> Note this does not mean that the command/process within the container runs in the background. It only means the container itself running on the host machine will run in the background. In practice the main process that runs within the container needs to run in the foreground so that the container does not exit automatically. This is why using this option does not make sense in the context of the ubuntu container, as will be shown in the next section.

This is the right option when using a web server image that needs to continue running after we run the container. This way the control will return to the terminal on the host, while the container continues running in the background.

Here is an example of running the nginx server using the -d option:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
```

Omitting the -d option would make the container run in the forground.

Since the -d option does not let us interact with the container, we can use the docker exec command to connect to the running container using the container name or id.

Here is an example of connecting to the running znginx container and running the bash command.

```bash
docker exec -it znginx bash
```

We need to explicitly run the bash command since the nginx image does not run the bash command by default. Instead by default it runs an entrypoint script to startup the web server.

## Docker run options for host mapping

### Port Mapping

When we ran the nginx container in the previous section we also used the -p option.

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
```

This option is used to map the port 8080 of the host to the internal network port 80 of the container that the web server running in the container is listening on.

The mapping allows us to connect to port 8080 using a web browser on our host machine with the actual http requests routed to port 80 within the running container.

### Volume Mapping

Another mapping option we can use with the docker run command is the -v option for mapping files and directories in the container to files and directories on our host.

Here we are using the -v option to map the contents of the `$(pwd)/content` directory on our host into `/usr/share/nginx/html/` directory in the running container.

```bash
docker run --rm -d --name znginx -p 8080:80 -v $(pwd)/content:/usr/share/nginx/html/ nginx
```

Additionally we can make the content mapped into the container read only, which means the container can not modify the content, by adding a third segment `ro` to the mapping.

```bash
docker run --rm -d --name znginx -p 8080:80 -v $(pwd)/content:/usr/share/nginx/html/:ro nginx
```

multiple mappings of the same kind can be specified:

```bash
docker run --rm -d --name znginx -p 8080:80 -v $(pwd)/content:/usr/share/nginx/html/:ro nginx -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro nginx
```

## Running the bash shell command in the ubuntu container

As shown in the previous section running the ubuntu container without specifying the bash command will run the default bash command of the image:

```bash
docker run --rm -it --name zubuntu ubuntu:20.10
```

That is equivalent to running the bash command globally:

```bash
docker run --rm -it --name zubuntu ubuntu:20.10 bash
```

Which is equivalent to running the bash command from it installation directory:

```bash
docker run --rm -it --name zubuntu ubuntu:20.10 /bin/bash
```

Running the ubuntu container in detached (non interactive) mode causes the container to stop immediately after running.

This is normal as the process running as PID 1 in the container exits immediately after running the container, which in turn stops the container.

```bash
docker run -d --name zubuntu ubuntu:20.04
#see the stoped container
docker ps -a
#remove the stopped container
docker rm zubuntu

#this time we will autoremove after itb stops
docker run --rm -d --name zubuntu ubuntu:20.04
#we will not see the stopped container since it is autoremoved by the --rm option
docker ps -a
```

## Running the bash shell in other common containers

```bash
docker run -ti --rm -v $PWD:/app -w /app node bash
docker run -ti --rm -v "$PWD":/app -w /app php:7.2-cli bash
docker run -ti --rm -v $PWD:/app -w /app composer bash
docker run -ti --rm -v $PWD:/app -w /app mysql bash
docker run -ti --rm -v $PWD:/app -w /app redis bash
```

The -w option is a working directory option.
It changes the current directory within the container to the `/app` directory when the container starts, so that when the bash command is run within the container, the present working directory would be `/app` directory.

Note that the volume mapping will ensure that the `/app` directory will exist in the container, if it does not already.

## Running an NGINX container

```bash
docker run --rm -d --name znginx -p 8080:80 nginx

#exec into running container to inspect the contents. type exit to exit the container
docker exec -it znginx bash

#stop the container using its name
docker stop znginx
```

## Running one off commands in your project directory

```bash
docker run --rm -v $PWD:/app -w /app composer install

docker run --rm -v $PWD:/app -w /app node npm install

docker run --rm -v $PWD:/app -w /app node yarn

docker run --rm -v $PWD:/app -w /app composer create-project --prefer-dist laravel/laravel:5.7.13 mylaravelproject

docker run --rm -v $PWD:/app -w /app composer php artisan make:auth

docker run --rm -v "$PWD":/app -w /app php:7.2-cli php -m

docker run --rm -v "$PWD":/app -w /app php:7.2-fpm php -m
```

The -w option is a working directory option.
It changes the current directory within the container to the `/app` directory when the container starts, so that the command that is executed uses the `/app` working directory as its present working directory.

Note that the volume mapping will ensure that the `/app` directory will exist in the container, if it does not already.

## The docker build context, Dockerfiles and .dockerignore

### docker build and the context path

context is the source directory used to build the docker image when we run the docker build command.

The source files specified in the COPY command of the docker file are relative to the context directory.

all the files in the context directory that are not specified in an optional `.dockerignore` file within the same directory are copied to the docker engine server where the image is built.

### docker build and the Dockerfile

by default the docker or docker-compose build command looks for a file named Dockerfile in the current directory that we run the build command in (not the docker context directory unless the context is a dot signifying the current directory)

we can however specify a explicit docker file using the -f flag for the build command to use

The form of the command is below:

docker build -f path/to/mydockerfile -t imagename path/to/contextdirectory

Example where the mydockerfile is a file in the current directory and the context is the current directory:

docker build -f mydockerfile -t imagename .

## Docker run options

run as with specified user and group id

-u $(id -u):$(id -g)

pass env value from host env vars

-e AWS_KEY

pass explicit env value

-e AWS_KEY=abcd

## Docker Volume mounting path rules

### Directory path rules

Say we have our application content in a `/app` directory and nginx document root is the `/var/www` directory.

Bind mounting a host directory:

> Note: if destination directory does not exist, there will be a runtime error

```bash
#mount /app directory at /var/www directory
#which means content of /app directory appear in the /var/www directroy
#source or destination trailing slash is not required

docker run -v /app/:/var/www/
docker run -v /app:/var/www
docker run -v /app/:/var/www
docker run -v /app:/var/www/

#if we are running the command from the /app directory we can use the present working directory

docker run -v $PWD:/var/www/
docker run -v $(pwd):/var/www/
docker run -v ${PWD}/:/var/www/
docker run -v $(pwd)/:/var/www/
docker run -v ./:/var/www/
docker run -v .:/var/www/
```

Mounting a named volume:

```bash
#source path does not start with a slash or dot and is a volume name
#means content of /var/www directory are mapped to the storage area of the named volume
#destination trailing slash is not required

docker run -v app:/var/www/
docker run -v app:/var/www
```

### File path rules

> Note: a host file can not be mapped to a container directory without specifying the file name in the container directory

```bash
#server.conf on host is mounted as default.conf in conf.d directory of container
#replaces existing container default.conf
docker run -v /app/nginx/server.conf:/etc/nginx/conf.d/default.conf

#default.conf on host is mounted as default.conf in conf.d directory of container
#replaces existing container default.conf
docker run -v /app/nginx/default.conf:/etc/nginx/conf.d/default.conf

#server.conf on host is mounted as server.conf in conf.d directory of container
docker run -v /app/nginx/server.conf:/etc/nginx/conf.d/server.conf

#nginx.conf on host is mounted as server.conf in conf.d directory of container
docker run -v /app/nginx/nginx.conf:/etc/nginx/conf.d/server.conf
```

> Note: Same path rules apply when specifying volume paths in a docker-compose file.

## Docker COPY command path rules

Say are in a application directory with two sub directories named config and content. Each of those directories have content that we need to copy to a docker image we are building. The directory also has a default dockerfile.

Then from within the application directory we run the docker build command specifying the build context to be the current directory.

```bash
docker build -t myappimage .
```

Within the dockerfile the COPY command has the following path rules.

Directory COPY rules:

```Dockerfile
#copy content of the content directory into /usr/share/nginx/html/
#if /usr/share/nginx/html/ doesn’t exist, it is created as a directory
#if a file of same name exists in /usr/share/nginx/html/ it will be overwritten
#source directory ending with a slash is optional
#destination needs to end in a slash
COPY content /usr/share/nginx/html/
COPY content/ /usr/share/nginx/html/
```

File COPY rules:

```Dockerfile
#copy host default.conf file to docker image default.conf
#creates the directory and file in the image if it does not exist
COPY config/default.conf /etc/nginx/conf.d/default.conf

#copy host nginx.conf file to docker image, renamed to default.conf
#creates the directory and file in the image if it does not exist
COPY config/nginx.conf /etc/nginx/conf.d/default.conf
```

## Resources

[NGINX Docker Image](https://hub.docker.com/_/nginx)

[how-to-use-the-official-nginx-docker-image](https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image)

[deploying-nginx-on-docker](https://www.nginx.com/blog/deploying-nginx-nginx-plus-docker)

[https://jonathan.bergknoff.com/journal/run-more-stuff-in-docker](https://jonathan.bergknoff.com/journal/run-more-stuff-in-docker)

[how-to-share-data-between-the-docker-container-and-the-host](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host)

[https://docs.docker.com/reference](https://docs.docker.com/reference)

[understanding-volume-instruction-in-dockerfile](https://stackoverflow.com/questions/41935435/understanding-volume-instruction-in-dockerfile)

[docker-tips-about-the-build-context](https://medium.com/better-programming/docker-tips-about-the-build-context-dbc76505e178)
