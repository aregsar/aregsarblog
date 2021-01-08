# Working With NGINX Docker Image

January 8, 2021 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail how you can use the Nginx docker image to configure and run a web server on your local development environment.

## Running the Nginx container

We can run an nginx docker container using the official nginx docker image by running the following command:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
```

This is equivalent to running:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx:latest
```

The `nginx` name at the end of the command is the image name.

If the image does not exist on the host machine it will be pulled from the Docker hub.

If the image tag is not specified, then it is assumed to be the latest tag.

The --rm option will auto remove the container when it is stopped.

The -d option runs the container in detached mode which means the container is running in the background and control returns back to the terminal.

The --name option gives the running container a unique name that we can reference instead of the container id in other docker commands.

the -p option maps the local port 8080 on the host machine to port 80 inside the container where the nginx server is listening for incoming HTTP requests.

We can run the `docker ps` command to see the container running:

```bash
docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
49631bc95069        nginx               "/docker-entrypoint.…"   14 seconds ago      Up 13 seconds       0.0.0.0:8080->80/tcp   znginx
```

We can navigate to `localhost:8080` to see the displayed default nginx web page based on the default nginx configuration:

```bash
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```

We can use the `docker cp` command to copy files from the running container to our host machine for inspection:

Let's copy the the default configuration file from nginx to our current directory:

```bash
#copy the original nginx default.conf file from the running docker container to a file named default.conf in our current directory to inspect
docker cp znginx:/etc/nginx/conf.d/default.conf default.conf
```

The first argument to docker cp is the path to the file in the container prefixed by the container name and a colon.

The second argument is the path to the file we want to copy to on our host machine.

Once copied we can display the contents of the file:

```bash
cat default.conf

server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

Now lets use the `docker exec` command to connect to the running container using the container name.

The command allows us to execute the bash command in the running container to run an interactive bash shell within the container:

```bash
#exec into running container by the container name to inspect the contents (type exit to exit the running container)
docker exec -it znginx bash
```

The -it option specifies that we will run the bash command in interactive mode.

Once we execute this command our command prompt in the terminal will change to the terminal prompt of the running container and drop us into the root directory of the container:

```bash
root@ebc3b9243cfb:/#
```

We can run bash commands within the container.

Let's display the current directory and list the contents of the root directory:

```bash
root@ebc3b9243cfb:/# pwd
root@ebc3b9243cfb:/# ls -al
```

Once we are done interacting with the container we can exit the container and return to the terminal prompt on our host by typing the exit command:

```bash
root@ebc3b9243cfb:/# exit
```

Now that we are back in the local shell, we can stop the running container using the container name:

```bash
docker stop znginx
```

> Note: when we stop a container the running process in the container receives a signal to quit. Sometimes it takes a few seconds for the process to respond and shutdown. Usually if the process does not shut down within a short time, docker will shutdown the container.

After stoping the container let's see if we can show the stopped container:

```bash
docker ps -a
```

The -a flag show all running and stopped containers.

However we don't see the stopped container because it was automatically removed since we specified the --rm option when we ran the container.

Lets try running and stopping the container without the --rm option:

```bash
docker run -d --name znginx -p 8080:80 nginx
docker stop znginx
```

Now let's run the docker ps command with -a option again:

```bash
docker ps -a

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
89cc0a7c4f48        nginx:latest        "/docker-entrypoint.…"   49 seconds ago      Exited (0) 37 seconds ago                       znginx
```

This time we will see the stopped container since we did not specify the --rm option.

This time we have to explicitly remove the container ourselves using the docker rm command:

```bash
docker rm znginx
```

Now if we run the `docker ps -a` command again we will not see the stopped container anymore.

## Inspecting the nginx container

Let now run the nginx container again to see how it is configured:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
```

The first thing we can do is run the `docker inspect` command using the running container name or id to see how the original container image was built.

Running docker inspect on the container also shows any runtime command and options that are modify the build time image configuration at container startup.

When the docker run command is executed, an additional command can be specified after the docker image name. This command runs at container startup, overriding any default command specified in the docker image.

In our case we do not run any commands after specifying the nginx image name, so the default command that the image is built with will run at container startup.

> Note: In addition to running docker inspect using a running container name or id, we can run docker inspect using any docker image name to just see the build time configuration of an image.

```bash
docker inspect znginx
```

I have listed only a partial snippet output of the above docker inspect command:

```bash
[
    {
        "Id": "6183f96c4571d4b023c3dd00ab815bf5d5d6597fa99c1c8aa094c139140d584d",
        "Created": "2021-01-06T20:27:25.64038Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 2258,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-01-06T20:27:25.9872245Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "6183f96c4571",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.19.6",
                "NJS_VERSION=0.5.0",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e"
            },
            "StopSignal": "SIGQUIT"
        },
    }
]
```

Two interesting parts of the configuration are the Cmd and Entrypoint items:

```bash
   "Cmd": [
        "nginx",
        "-g",
        "daemon off;"
    ],
    "Entrypoint": [
        "/docker-entrypoint.sh"
    ],
```

As we can see at startup the container runs the `/docker-entrypoint.sh` script that is located at the root of the container filesystem.

In addition the contained runs the following default command at startup `nginx -g daemon off;`.

This causes main container startup process PID 1 to run the nginx server in non-background mode within the container. Since nginx would then be running in the foreground, the PID 1 process would keep running which would in turn keep the container running until it is explicitly is stopped.

If nginx was run in its standard background mode, the main container startup process PID 1, would fork to run the nginx server in background mode and would exit afterwards causing the container to exit and be stopped automatically.

Let's docker exec into the running container:

```bash
docker exec -it znginx bash
```

And Let's verify that the `docker-entrypoint.sh` exists in the root directory:

```bash
root@6183f96c4571:/# pwd
/
root@6183f96c4571:/# ls | grep docker-entrypoint.sh
docker-entrypoint.sh
root@6183f96c4571:/# exit
```

Back in our local shell we can stop the container which will be auto removed after being stopped:

```bash
docker stop znginx
docker ps -a
```

## Inspecting the nginx directory structure

Once again lets start by running the container and docker exec-ing into it:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx
docker exec -it znginx bash
```

We are dropped in again into the root directory of the container.

The nginx installation directory is `/etc/ningx`.

Let's list the content of this directory:

```bash
root@6183f96c4571:/# ls -l /etc/nginx
```

Here we see all the nginx configuration files and directories:

The main config file is located at `/etc/nginx/nginx.conf`.

Let's list the content of this file:

```bash
root@6183f96c4571:/# cat /etc/nginx/nginx.conf
```

The content of the file is listed below:

```bash
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

The last line in the configuration includes any files ending with `.conf` from the `/etc/nginx/conf.d/` directory.

The default installation of nginx only includes a `/etc/nginx/conf.d/default.conf` file.

Let's print out its content:

```bash
root@6183f96c4571:/# cat /etc/nginx/conf.d/default.conf
```

The content is shown below:

```bash
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

As we can see this file contains a nginx server block that configures a default virtual server for nginx that handles our web requests.

We can also see that there is a root document `root /usr/share/nginx/html;` specified within the `location /` block that matches to root URL of `/`.

> See the resources section at the bottom of this post for nginx configuration file specification resources.

The document root specifies the `/usr/share/nginx/html` as the document root from which nginx will search for and server static content files.

Let's take a look in this directory:

```bash
root@6183f96c4571:/# ls /usr/share/nginx/html
```

we can see an index.html file which is served by default when we navigate to `localhost:8080` in the browser.

Let's see that the content of this file matches the html file content that we see when we navigate to `localhost:8080`.

```bash
root@6183f96c4571:/# cat /usr/share/nginx/html/index.html
```

And indeed it is:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Welcome to nginx!</title>
    <style>
      body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>
      If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.
    </p>

    <p>
      For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br />
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.
    </p>

    <p><em>Thank you for using nginx.</em></p>
  </body>
</html>
```

This file is served by default when we navigate to the root URL `/` of `localhost:8080`, because of the `index index.html index.htm;` directive inside the `location /` block of the `default.conf` file.

You may have also noticed a `fastcgi_params` file in the `/etc/nginx/` directory. This file is included by the line `include fastcgi_params;` in a commented out section of the `default.conf` file.

If we uncomment that section the content of the `fastcgi_params` file will be included in the `default.conf` file. The content contains additional CGI parameters that will be passed to the PHP server specified in that section.

Let's see its content:

```bash
root@6183f96c4571:/# cat /etc/nginx/fastcgi_params
```

It lists various CGI parameters used with proxying requests to the PHP-FPM server.

Some of them in the output are listed below:

```bash
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
```

Exit the container:

```bash
root@6183f96c4571:/# exit
```

Back in our local shell we can stop the container which will be auto removed after being stopped:

```bash
docker stop znginx
```

#check that the container is removed:

```bash
docker ps -a
```

## Overriding the nginx default content and configuration

We can use the docker run command with volume mapping to change the default nginx configuration. We can also change the content that is served using volume mapping.

Let's first setup a directory structure on our host machine to hold the new configuration and content files that we want to map to the running container:

```bash
mkdir app && cd app
mkdir -p ./docker/nginx/config
mkdir -p ./docker/nginx/content
touch ./docker/nginx/config/default.conf
touch ./docker/nginx/content/index.html
touch ./docker/nginx/content/about.html
```

Now we need to open the `./docker/nginx/config/nginx.conf` configuration file and add the following content:

```bash
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /var/www;
        index  index.html index.htm;
    }
}
```

I have changed the document root from `/usr/share/nginx/html` directory to a `/var/www` directory. The `/var/www` directory does not exist in the image but will be temporarily created when we run the container with the volume mapping option.

The volume mapping specified with the -v option in the docker run command maps an existing directory on the host machine to a directory in the container. If the directory in the container does not exist, it will be created at container startup but the container will revert to its original directory structure the next time it is run without the volume option.

Next we need to open the `./docker/nginx/content/index.html` content file and add the following:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Updated nginx!</title>
  </head>
  <body>
    <h1>Updated nginx!</h1>
  </body>
</html>
```

Finally we need to open the additional `./docker/nginx/content/about.html` content file and add the following:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>About nginx!</title>
  </head>
  <body>
    <h1>About nginx!</h1>
  </body>
</html>
```

Now we are ready to replace the content and configuration when we run the container.

First lets just override the default `index.html` page by mapping the file that we setup on the host to the default `/usr/share/nginx/html/index.html` file in the document root directory of the container.

From the app directory run the following command:

```bash
#use a different configuration that serves from document root at /var/www/ directory. Put ./nginx/content/index.html onto /var/www/index.html
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/docker/nginx/content/index.html":/usr/share/nginx/html/index.html nginx
```

Navigate to `localhost:8080` to view the new `index.html` file content.

Stop the container.

```bash
docker stop znginx
```

As a more practical alternative, instead of mapping the single `index.html` file, we can map the entire contents of the content directory that we setup on the host to the default `/usr/share/nginx/html` document root directory in the container.

From the app directory run the following command:

```bash
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/docker/nginx/content":/usr/share/nginx/html nginx
```

Navigate to `localhost:8080` to view the new `index.html` file content. Also Navigate to `localhost:8000/about.html`. We can see the additional content file.

Stop the container:

```bash
docker stop znginx
```

Next Let's override the default configuration file in the container with the one that we created that specifies a new document root `/var/www`.

To do so we need to map the `default.conf` file we created on the host to the container.

We will also need to map our content files from the host to the new document root in the container.

From the app directory run the following command:

```bash
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/docker/nginx/content":/var/www -v "$(pwd)/docker/nginx/config/default.conf":/etc/nginx/conf.d/default.conf nginx
```

Navigate to `localhost:8080` to view the new `index.html` file content. Also Navigate to `localhost:8000/about.html`. We can see the additional content file.

The `/var/www` directory does not exist in the nginx docker image, but is created automatically when we run the container.

Let's docker exec into the container to verify this:

```bash
docker exec -it znginx bash
root@80c8cf23bd51:/#
```

Now verify that there is a /var/www directory that has our volume mapped content files:

```bash
root@80c8cf23bd51:/# ls /var/www
about.html  index.html
```

Exit the container and stop it.

```bash
root@80c8cf23bd51:/# exit
docker stop znginx
```

Finally let's rename our `default.conf` on the host machine to `nginx.conf` and run the command again:

```bash
mv ./docker/nginx/config/default.conf ./docker/nginx/config/nginx.conf
docker run --rm -d --name znginx -p 8080:80 -v "$(pwd)/docker/nginx/content":/var/www -v "$(pwd)/docker/nginx/config/nginx.conf":/etc/nginx/conf.d/default.conf nginx
```

In this case the host file name `nginx.conf` on the host gets copied to `default.conf` in the container, overriding the original `default.conf` file.

Navigate to `localhost:8080` to view the new `index.html` file content. Also Navigate to `localhost:8000/about.html`. We can see the additional content file.

Stop the container:

```bash
docker stop znginx
```

## Building a custom image

Lets change to the nginx directory to create a custom docker image:

```bash
cd docker/nginx
```

Next let's create a file named `Dockerfile` that copies the index.html file in the content directory to the docker image:

```bash
echo 'FROM nginx:latest' > Dockerfile
echo 'COPY ./content/index.html /usr/share/nginx/html/index.html' >> Dockerfile
```

Let's use this `Dockerfile` file to build a custom image:

```bash
docker build -t mynginx .
```

The docker build command will build a new docker image by the name of `mynginx`. The command uses the `Dockerfile` we created in the current directory by default.

> The docker build command will be described in the next section.

We can list all the images on our system:

```bash
docker images
```

You should see the image tagged `mynginx` in the list of images.

Let's run the image that we just created:

```bash
docker run --rm -d --name znginx -p 8080:80 mynginx
```

We can navigate to localhost:8080 as usual to see our custom html page that we copied to the container image in the Dockerfile.

Let's stop the container with the usual command and delete the Dockerfile.

Lets remove the `mynginx` image as well:

```bash
rm Dockerfile
docker image rm mynginx
docker images
```

The `mynginx` image will no longer show up in the list of images.

Let's create a new `Dockerfile` file that copies the contents of the content directory into the docker image:

```bash
echo 'FROM nginx:latest' > Dockerfile
echo 'COPY ./content /usr/share/nginx/html' >> Dockerfile
```

Now we can run the same docker build command to build the image again with the new Dockerfile.

After we build the image we can run it as before.

Now Let's stop the container and delete the Dockerfile again as well as deleting the `mynginx` image as before.

Let's make one last Dockerfile that also copies a nginx configuration file that changes the default nginx document root:

```bash
echo 'FROM nginx:latest' > Dockerfile
echo 'COPY ./content /var/www' >> Dockerfile
echo 'COPY ./config/default.conf /etc/nginx/conf.d/default.conf' >> Dockerfile
```

Again we can build, run and then stop the container.

Note that even though the /var/www directory did not exist in the base image, it was created as part of the COPY command.

## Using a Dockerfile file to build a custom NGINX container

The docker build command can be used to build a custom image using our `Dockerfile` file and then we can run this custom image instead of the stock nginx image.

From the app directory we can change into the nginx directory and run the docker build command to build a new image:

```bash
cd docker/nginx
run docker build -t mynginx .
```

By default the docker run command uses the `Dockerfile` file that it finds in the directory that the command is run.

The period at the end of the command specifies the current directory as the build context. This tells the docker command to recursively copy the contents of the current directory to the docker engine where the image is built.

Source files referenced in the Dockerfile are relative to the build context directory.

The -t option tags the image with the name `mynginx`. This name can be then used to run the image:

```bash
docker run --rm -d --name znginx -p 8080:80 mynginx
```

## Running explicit commands

If we wanted to we can explicitly run a container startup command when we run the nginx container:

```bash
docker run --rm -d --name znginx -p 8080:80 nginx nginx -g 'daemon off;'
```

He we are explicitly running the default `nginx -g 'daemon off;'` command that would be run by default when the nginx image starts.

In effect we are overriding the same command that would have been run, thus nothing will change from a normal container run.

The first `nginx` in the docker run command is the docker image name.

The second `nginx` in the docker run command is the command that runs in the container which starts up the nginx server in the container.

The `-g 'daemon off;'` after the `nginx` command is the option that runs the main nginx server process in the foreground.

We could also run the same command with the custom docker image that we built in the last section:

```bash
docker run --rm -d --name znginx -p 8080:80 mynginx nginx -g 'daemon off;'
```

## Resources

https://www.digitalocean.com/community/tutorials/how-to-run-nginx-in-a-docker-container-on-ubuntu-14-04

[NGINX Docker Image](https://hub.docker.com/_/nginx)

[how-to-use-the-official-nginx-docker-image](https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image)

[deploying-nginx-on-docker](https://www.nginx.com/blog/deploying-nginx-nginx-plus-docker)

# working with nginx docker image

[working with nginx docker image](https://aregsar.com/blog/2020/working-with-nginx-docker-image)

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
