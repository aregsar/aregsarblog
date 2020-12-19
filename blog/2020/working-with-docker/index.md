# Working With Docker

December 19, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[working with docker](https://aregsar.com/blog/2020/working-with-docker)

## Running the bash shell in ubuntu container

```bash
docker run --rm -it --name zubuntu ubuntu:20.04
docker run --rm -it --name zubuntu ubuntu:20.04 bash
docker run --rm -it --name zubuntu ubuntu:20.04 /bin/bash

docker run --rm --name zubuntu ubuntu:20.04
docker run --rm -d --name zubuntu ubuntu:20.04
```

## Running the bash shell in other common containers

```bash
docker run -ti --rm -v $PWD:/app -w /app node bash
docker run -ti --rm -v "$PWD":/app -w /app php:7.2-cli bash
docker run -ti --rm -v $PWD:/app -w /app composer bash
docker run -ti --rm -v $PWD:/app -w /app mysql bash
docker run -ti --rm -v $PWD:/app -w /app redis bash
```

## Running the NGINX container

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
docker run --rm -d --name znginx -p 8080:80 nginx:latest
docker run --rm -d --name znginx -p 8080:80 library/nginx:latest
docker exec -it znginx bash

#bind mount volume maps container directory to host directory
docker run --rm -d --name znginx -p 8080:80 -v $(pwd)/content:/usr/share/nginx/html/:ro nginx
docker run --rm -d --name znginx -p 8080:80 -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro nginx
```

## Using a Dockerfile to build a custom NGINX container

The directory structure below where we have a docker file and content directory

```bash
app/docker/nginx/Dockerfile
app/docker/nginx/content/
app/docker/nginx/content/index.html
app/docker/nginx/config/
app/docker/nginx/config/nginx.conf
```

```Dockerfile
#Dockerfile
FROM nginx
#if /usr/share/nginx/html doesn’t exist, it is created as a file
#copy content of the content directory into /usr/share/nginx/
#COPY content /usr/share/nginx/html
#copy content of the content directory into /usr/share/nginx/html/
#if /usr/share/nginx/html/ doesn’t exist, it is created as a directory
COPY content /usr/share/nginx/html/
COPY content/index.html /usr/share/nginx/html/index.html
#overwrite the main nginx.conf file
COPY config/nginx.conf /etc/nginx/nginx.conf
#overwite the conf.d/default.conf file
COPY config/nginx.conf /etc/nginx/conf.d/default.conf
```

```bash
cd app/docker/nginx
run docker build -t mynginx .
docker run --rm -d --name znginx -p 8080:80 mynginx
docker run --rm -d --name znginx -p 8080:80 mynginx nginx -g 'daemon off;'
```

## Running one off commands in your project direcrory

```bash
docker run --rm -v $PWD:/app -w /app composer install

docker run --rm -v $PWD:/app -w /app node npm install

docker run --rm -v $PWD:/app -w /app node yarn

docker run --rm -v $PWD:/app -w /app composer create-project --prefer-dist laravel/laravel:5.7.13 mylaravelproject

docker run --rm -v $PWD:/app -w /app composer php artisan make:auth

docker run --rm -v "$PWD":/app -w /app php:7.2-cli php -m

docker run --rm -v "$PWD":/app -w /app php:7.2-fpm php -m
```

## The inspect command

```bash
docker inspect nginx
docker inspect nginx:latest
docker inspect library/nginx:latest
docker inspect library/nginx

docker inspect ubuntu:20.04
docker inspect library/ubuntu:20.04

#inspect the build time configuration an image
docker inspect image_name_or_id

#inspect the runtime configuration of a container
docker inspect container_name_or_id
```

## Common commands when working with containers

```bash
#list all containers (ommiting the flag will list only running containers)
docker ps -a

#list all images
docker images

#stop one or more running containers by name or id
docker stop container_name_or_id

#remove one or more stopped containers by name or id
docker rm container_name_or_id

#stop all running containers
docker stop $(docker ps -q)

#remove all stoped containers
docker container prune -f

#remove all images
docker image prune -a -f
docker rmi $(docker images -q)

docker system info
```

## Listing all container commands

```bash
#list available container commands
docker container

#list available image commands
docker image

#list available volume commands
docker volume

#list available network commands
docker network

Examples:

docker network ls
docker network prune
```

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

## Resources

https://hub.docker.com/_/nginx
https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image/
https://docs.docker.com/storage/bind-mounts/
