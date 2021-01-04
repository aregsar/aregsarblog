# Working With NGINX Docker Image

January 4, 2021 by [Areg Sarkissian](https://aregsar.com/about)

[Working With NGINX Docker Image](https://aregsar.com/blog/2020/working-with-nginx-docker-container)

In this post I detail how you can use the Nginx docker image to configure and run the a web server in your local development environment.

## Running the Nginx container

```bash
docker run --rm -d --name znginx -p 8080:80 nginx

#copy the original nginx default.conf file from the running docker container to a file named default.conf in our current directory to inspect
docker cp znginx:/etc/nginx/conf.d/default.conf default.conf

#exec into running container by the container name to inspect the contents (type exit to exit the running container)
docker exec -it znginx bash
```

navigate to http://localhost:8080

```bash
docker stop znginx
```

Alternative equivalent image names:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx:latest
docker run --rm -d --name znginx -p 8080:80 library/nginx:latest
```

## Overriding the nginx default content and configuration

The directory structure below where we have a docker file and content directory

```bash
mkdir app && cd app
touch ./docker/nginx/content/
touch ./docker/nginx/content/index.html
touch ./docker/nginx/config/
touch ./docker/nginx/config/nginx.conf
```

```bash
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/nginx/content":/usr/share/nginx/html:ro nginx

docker stop znginx

#use a different configuration that serves from document root at /var/www/ directory. Put contents of ./nginx/content into /var/www
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/nginx/content":/var/www:ro -v "$(pwd)/nginx/config/nginx.conf":/etc/nginx/conf.d/default.conf:ro nginx

docker stop znginx

#use a different configuration that serves from document root at /var/www/ directory. Put ./nginx/content/index.html onto /var/www/index.html
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/nginx/content/index.html":/var/www/index.html:ro -v "$(pwd)/nginx/config/nginx.conf":/etc/nginx/conf.d/default.conf:ro nginx

docker stop znginx

docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/nginx/content":/var/www:ro -v "$(pwd)/nginx/config/default.conf":/etc/nginx/conf.d/default.conf:ro nginx

docker stop znginx
```

## Building a custom image

Adding a docker file:

```bash
touch ./docker/nginx/Dockerfile
```

```dockerfile
FROM nginx:latest
COPY $PWD/nginx/content /usr/share/nginx/html
```

```dockerfile
FROM nginx:latest
COPY $PWD/nginx/content/index.html /usr/share/nginx/html/index.html
```

```dockerfile
FROM nginx:latest
COPY $PWD/nginx/config/default.conf /etc/nginx/conf.d/default.conf
COPY $PWD/nginx/content /var/www
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

## Resources

https://www.digitalocean.com/community/tutorials/how-to-run-nginx-in-a-docker-container-on-ubuntu-14-04

[NGINX Docker Image](https://hub.docker.com/_/nginx)

[how-to-use-the-official-nginx-docker-image](https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image)

[deploying-nginx-on-docker](https://www.nginx.com/blog/deploying-nginx-nginx-plus-docker)
