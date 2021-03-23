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

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

#verify the master nginx process is running
ps aux | grep nginx
#will only see the master process
root      3269  0.0  0.0  58128  1520 ?        Ss   23:17   0:00 nginx: master process nginx

#type nginx to start worker processes (defalts to daemon on)
#nginx -g 'daemon on;'
nginx



#verify the master and worker nginx processes are running
root      3269  0.0  0.0  58128  1520 ?        Ss   23:17   0:00 nginx: master process nginx
www-data  3270  0.0  0.1  58456  3204 ?        S    23:17   0:00 nginx: worker process
www-data  3271  0.0  0.1  58456  3204 ?        S    23:17   0:00 nginx: worker process

#install curl to test the server
apt-get install -y curl

#test the server
curl localhost
```

We see the following response:

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

```bash
#stop the server workers and master process
nginx -s stop

#will not show any nginx processes
ps aux | grep nginx
```

```bash
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
# /etc/nginx/conf.d – directory of configuration files included by main nginx.conf.(initially empty)
# /etc/nginx/sites-available – List of all websites configured through Nginx.
# /etc/nginx/sites-enabled – List of websites actively being served by Nginx.
# /etc/nginx/snippets - configuration snippets to include in server blocks

# /var/log/nginx/access.log – Access logs tracking every request to your server.
# /var/log/ngins/error.log – A log of any errors generated in Nginx.

cd /etc/nginx/sites-available
ls -al

#default server configiration
-rw-r--r-- 1 root root 2412 Aug 25 13:09 default

cd sites-enabled
ls -al
#symlink to server configiration
lrwxrwxrwx 1 root root   34 Jan 12 23:14 default -> /etc/nginx/sites-available/default
cat default
```

We see the following configuration file:

```nginx
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
listen 80 default_server;
listen [::]:80 default_server;

# SSL configuration
#
# listen 443 ssl default_server;
# listen [::]:443 ssl default_server;
#
# Note: You should disable gzip for SSL traffic.
# See: https://bugs.debian.org/773332
#
# Read up on ssl_ciphers to ensure a secure configuration.
# See: https://bugs.debian.org/765782
#
# Self signed certs generated by the ssl-cert package
# Don't use them in a production server!
#
# include snippets/snakeoil.conf;

root /var/www/html;

# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html;

server_name _;

location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}

# pass PHP scripts to FastCGI server
#
#location ~ \.php$ {
#include snippets/fastcgi-php.conf;
#
## With php-fpm (or other unix sockets):
#fastcgi_pass unix:/run/php/php7.4-fpm.sock;
## With php-cgi (or other tcp sockets):
#fastcgi_pass 127.0.0.1:9000;
#}

# deny access to .htaccess files, if Apache's document root
# concurs with nginx's one
#
#location ~ /\.ht {
#deny all;
#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#listen 80;
#listen [::]:80;
#
#server_name example.com;
#
#root /var/www/example.com;
#index index.html;
#
#location / {
#  try_files $uri $uri/ =404;
#}
#}

```

## Resources

[Working With Docker](https://aregsar.com/blog/2020/working-with-docker)

[Working With NGINX Docker Image](https://aregsar.com/blog/2021/working-with-nginx-docker-image)

[how-to-install-nginx-on-ubuntu-20-04](https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04)

[install-and-configure-nginx](https://ubuntu.com/tutorials/install-and-configure-nginx)

[how-to-install-nginx-on-ubuntu-18-04-quickstart](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04-quickstart)

[how-to-install-nginx-ubuntu-18-04](https://www.linode.com/docs/guides/how-to-install-nginx-ubuntu-18-04)

[nginx.com/install](https://www.nginx.com/resources/wiki/start/topics/tutorials/install)

[nginx cli](https://www.nginx.com/resources/wiki/start/topics/tutorials/commandline)

[nginx command line options](http://nginx.org/en/docs/switches.html)
