# working with nginx docker container

[working with nginx docker container](https://aregsar.com/blog/2020/working-with-nginx-docker-container)

$ docker run -it --rm -d -p 8080:80 --name web nginx

http://localhost:8080

## Customizing the server

The directory structure below where we have a docker file and content directory

```bash
app/docker/nginx/Dockerfile
app/docker/nginx/content/
app/docker/nginx/content/index.html
app/docker/nginx/config/
app/docker/nginx/config/nginx.conf
```

$ docker run -it --rm -d -p 8080:80 --name web -v "$(pwd)/nginx/content":/usr/share/nginx/html nginx

$ docker run -it --rm -d -p 8080:80 --name web -v "$(pwd)/nginx/content":/usr/share/nginx/html -v "$(pwd)/nginx/config/nginx.conf":/etc/nginx/conf.d/default.conf nginx

$ docker run -it --rm -d -p 8080:80 --name web -v "$(pwd)/nginx/content/index.html":/usr/share/nginx/html/index.html -v "$(pwd)/nginx/config/nginx.conf":/etc/nginx/conf.d/default.conf nginx

=================

FROM nginx:latest
COPY $PWD/nginx/content/index.html /usr/share/nginx/html/index.html

FROM nginx
COPY $PWD/nginx/config/nginx.conf /etc/nginx/conf.d/default.conf
COPY $PWD/nginx/content /usr/share/nginx/html

FROM nginx
COPY $PWD/nginx/config/nginx.conf /etc/nginx/conf.d/default.conf
COPY $PWD/nginx/content/index.html /usr/share/nginx/html/index.html

## Running the NGINX container

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
docker run --rm -d --name znginx -p 8080:80 nginx:latest
docker run --rm -d --name znginx -p 8080:80 library/nginx:latest

#exec into running container to inspect the contents
docker exec -it znginx bash

#volume maps host content directory to container directory as read only
#volume maps host config directory to container directory as read only
docker run --rm -d --name znginx -p 8080:80 -v $(pwd)/content:/usr/share/nginx/html/:ro -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro nginx
```

## Using a Dockerfile to build a custom NGINX container

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

We will use the docker build command to build our custom image using our dockerfile and then we will run this custom image instead of the stock nginx image.

```bash
cd app/docker/nginx
run docker build -t mynginx .
docker run --rm -d --name znginx -p 8080:80 mynginx
# here we are explicitly running the default command run by default by the base nginx image
docker run --rm -d --name znginx -p 8080:80 mynginx nginx -g 'daemon off;'
```
