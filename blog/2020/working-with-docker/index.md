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

## Docker run options

run as with specified user and group id

-u $(id -u):$(id -g)

pass env value from host env vars

-e AWS_KEY

pass explicit env value

-e AWS_KEY=abcd

## Docker volume mapping path rules

Say we have our application content in a `/app` directory and nginx document root is the `/var/www` directory.

The path rules and path format when mounting a directory volume are as follows:

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

#mount a named volume
#source path does not start with a slash or dot and is a volume name
#means content of /var/www directory are mapped to the storage area of the named volume
#destination trailing slash is not required

docker run -v app:/var/www/
docker run -v app:/var/www
```

mount a file

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

```bash
#copy host default.conf file to docker image default.conf
#created the directory and file in the image if it does not exist
COPY config/default.conf /etc/nginx/conf.d/default.conf

#copy host nginx.conf file to docker image default.conf
COPY config/nginx.conf /etc/nginx/conf.d/default.conf
```

## Resources

[NGINX Docker Image](https://hub.docker.com/_/nginx)

[how-to-use-the-official-nginx-docker-image](https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image)

[deploying-nginx-on-docker](https://www.nginx.com/blog/deploying-nginx-nginx-plus-docker)

[https://jonathan.bergknoff.com/journal/run-more-stuff-in-docker](https://jonathan.bergknoff.com/journal/run-more-stuff-in-docker)

https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host

https://docs.docker.com/reference/

https://stackoverflow.com/questions/41935435/understanding-volume-instruction-in-dockerfile
