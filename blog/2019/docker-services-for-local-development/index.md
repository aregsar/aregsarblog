# docker services for local development

[docker services for local development](https://aregsar.com/blog/2019/docker-services-for-local-development)

## Introduction

> This article assumes that you have Docker installed on your OS. The instructions to install docker can be found at docker.io.

In this article I will show how to run docker containers using the command line and using docker compose.

These containers will expose services running in the container that you can connect during the development of the web applications running on your local system. 

This way you can avoid installing, upgrading, un-installing and re-installing services that your application needs on your local system. In addition you can run these services in a container that uses the same OS that you will be using in production.

Each service will be running in its own docker container and will be configured to expose a TCP port that the locally running application can connect to.

In addition some of the containers will be configured to map directories in their internal file system to local directories using docker volumes. This is done to persist data on your local system when you stop running your application.

## Managing Docker Containers on your system

Before I talk about specific docker container, it is worthwhile to mention some commands used to manage the containers on your system.

`docker container run` is the command used to run docker containers. If the image for the container is not on your system, it will be automatically downloaded before running the container.

The many options for this command will be listed when I show you how to run the command in the following sections.

Aside from the docker run command, the following commands can be used to list, start, stop and remove containers created by the docker run command:

list all running containers:

`docker container ls`

list all containers:

`docker container ls -a`

stop a container:

`docker container stop <container-name>`

start a container again:

`docker container start <container-name>`

re-start a running container:

`docker container restart <container-name>`

remove a stopped container, run:

`docker container rm <container-name>`

To manage the docker images installed as a result of the docker run command, the following two commands are useful:

list all images:

`docker image ls`

remove an image:

`docker image rm <image-name>`

Finally, here are some some useful commands to clean up system resources:

Remove all stoped containers:
`docker container prune`

Remove all containers:
`docker container rm $(docker container ls -aq)`

Remove dangling images:
`docker image prune`

Remove unused images"
`docker image prune -a`

Remove unused volumes:
`docker volume prune`

## Running a up a MariaDb database container

let's first created a directory from which we will run our docker commands.
mkdir books && cd books


Now let's create a directory where we will map the volume where the data for the database will be stored
mkdir books-db

### Using the docker run command

To run a MariaDb container we can the run the following command:

execute docker run command from root directory:

```bash
docker container run -d \
--name mariadb_books \
-p 8081:3306 \
-v $(pwd)/books-db:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-e MYSQL_DATABASE=books \
-e MYSQL_USER=books \
-e MYSQL_PASSWORD=123456 \
mariadb:10.4
```
The container should be running

Verify by running  `docker container ls`

The -d option tells the container to run in the background.

The --name option is the name assigned to the container that can be used to stop, start, restart or remove the container.

The -p option maps port 8081 on the local system to port 3306 in the container that is the port that the mariadb service is listening for connections. By using this mapping we can connect to the service by using the 127.0.0.1:8081 host and port combination on our local machine.

The -e options are mariadb specific settings key value pairs that will be used by the mariadb service when it is started in the container.

Finally the last line is the docker image name and version of the container that will be run. If the image does not exist on the local system, it will be downloaded before the container is run.

The -v option specifies the volume mapping
TODO TODO


We can stop and remove the container by running:

```bash
docker container stop
docker container rm mariadb_books
```

### Using the docker compose file

First add a docker-compose.yml file to the book directory

`touch docker-compose.yml`

Then add the following into the file:

```yaml
version: "3.1"
services:
    mariadb:
      image: mariadb:10.4
      container_name: mariadb_books
      volumes:
        - ./books-db:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=123456
        - MYSQL_DATABASE=books
        - MYSQL_USER=books
        - MYSQL_PASSWORD=123456
      ports:
        - "8081:3306"
```

> Note that the options in the docker compose file for the `mariadb` service is precicly the same as the options we passed to the docker run command.

Now we can execute docker-compose from the books directory to start running the container:

`docker-compose up -d`

The container should be running

Verify by running  `docker container ls`

We can stop the container by running:

`docker-compose down`

Now we can use regular docker commands to remove the container and its associated image if desired.

====================

Change:

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

To:

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8081
DB_DATABASE=movies
DB_USERNAME=movies
DB_PASSWORD=panama

==================

php artisan tinker
>>> DB::connection()->getPdo();



