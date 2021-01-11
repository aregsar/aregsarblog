# install nginx on ubuntu linux

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

January 1, 2021

[install nginx on ubuntu linux](https://aregsar.com/blog/2021/install-nginx-on-ubuntu-linux)

In this post I will detail installation, configuration and running of the NGINX web server on the Docker Ubuntu image.

This will be done to simulate installing and running nginx on a ubuntu cloud server, so the instructions in this post will apply to cloud servers as well.

Since we are simulating a cloud server installation, we can ignore the recommendation to only run one process per docker container.

## The Ubuntu Docker image

We can run the official Ubuntu docker image by running the following command:

```bash
docker run --rm -it --name zubuntu ubuntu:20.10
```

Install NGINX

```bash
# add-apt-repository not available by default
# add-apt-repository -y ppa:nginx/development
#update available repository package list and version
apt-get update
#install the server
apt-get install -y nginx
#check nginx version
nginx -v
#verify nginx configuration is correct
nginx –t
#verify the nginx process is running
ps aux | grep nginx


##########################
# apt-get -y install sudo
# apt-get -y install apt-transport-https
# apt-get -y install ca-certificates
# apt-get -y install curl
# apt-get install -y software-properties-common
# add-apt-repository -y ppa:nginx/development
# apt-get update
#########################
```

## Nginx installation directories

```bash


# /var/www/html – Website content as seen by visitors.
# /etc/nginx – Location of the main Nginx application files.
# /etc/nginx/nginx.conf – The main Nginx configuration file.
# /etc/nginx/conf.d – list of configuration files included by main nginx.conf.
# /etc/nginx/sites-available – List of all websites configured through Nginx.
# /etc/nginx/sites-enabled – List of websites actively being served by Nginx.
# /etc/nginx/snippets - configuration snippets to include in server blocks

# /var/log/nginx/access.log – Access logs tracking every request to your server.
# /var/log/ngins/error.log – A log of any errors generated in Nginx.

```

## Resources

[Working With Docker](https://aregsar.com/blog/2020/working-with-docker)

[Working With NGINX Docker Image](https://aregsar.com/blog/2021/working-with-nginx-docker-image)

[how-to-install-nginx-on-ubuntu-20-04](https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04)

[install-and-configure-nginx](https://ubuntu.com/tutorials/install-and-configure-nginx)

[how-to-install-nginx-on-ubuntu-18-04-quickstart](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04-quickstart)

[how-to-install-nginx-ubuntu-18-04](https://www.linode.com/docs/guides/how-to-install-nginx-ubuntu-18-04)

[nginx.com/install](https://www.nginx.com/resources/wiki/start/topics/tutorials/install)
