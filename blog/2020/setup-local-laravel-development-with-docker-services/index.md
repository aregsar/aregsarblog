# Setup Local Laravel Development With Docker Services

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Setup Local Laravel Development With Docker Services](https://aregsar.com/blog/2020/setup-local-laravel-development-with-docker-services)

```bash
#show redis.conf file location 
redis-cli -p 6379 info | grep 'config_file'
# /usr/local/etc/redis.conf 

#show file directory in redis.conf file
cat /usr/local/etc/redis.conf | grep "dir "
# /usr/local/var/db/redis/



mysql
MariaDB> select @@datadir;
# /usr/local/var/mysql/
SELECT USER();
# aregsarkissian@localhost
SELECT DATABASE();
# NULL




mysql --help | grep "Default options" -A 1
# /usr/local/etc/my.cnf
# datadir=/etc/mysql/mysql.conf.d/mysqld.cnf
```

===========

In this post I will show how I run docker services that support my locally installed Laravel applications.

I run the following services in individual docker containers to support local Laravel application development:

MySQL or MariaDb as a database
Redis for sessions, caching and queued jobs 
Elastic Search as search index
MailHog as a local mail server

All the containers are run via Docker compose.

I use the `http://PhpDocker.io` service to generate the dockerfiles and docker compose file
for the containers. 

> Note: The `http://PhpDocker.io` service is also able to generate docker and configuration files for running `nginx` and `php-fpm` in docker containers to be able to also run the Laravel application in docker. However I don't use that feature as it eliminates a whole host of issues when developing applications locally.

On PhpDocker.io select MariaDb or MySql, Redis, Elastic Search and Mailhog.
Select any version of PHP which wont matter as we are going to delete the PHP related containers
Download package from PhpDocker.io and unzip it 
cd into it and delete the phpdocker folder 
copy the docker-compose.yml file into your laravel project
open the docker-compose.yml file.

You should see something like this:

```yaml
###############################################################################
#                          Generated on phpdocker.io                          #
###############################################################################
version: "3.1"
services:

    mailhog:
      image: mailhog/mailhog:latest
      container_name: radar-mailhog
      ports:
        - "8001:8025"

    redis:
      image: redis:alpine
      container_name: radar-redis

    mariadb:
      image: mariadb:10.4
      container_name: radar-mariadb
      working_dir: /application
      volumes:
        - .:/application
      environment:
        - MYSQL_ROOT_PASSWORD=panama
        - MYSQL_DATABASE=radar
        - MYSQL_USER=radar
        - MYSQL_PASSWORD=panama
      ports:
        - "8003:3306"

    elasticsearch:
      image: elasticsearch:6.5.4
      container_name: radar-elasticsearch

    webserver:
      image: nginx:alpine
      container_name: radar-webserver
      working_dir: /application
      volumes:
          - .:/application
          - ./phpdocker/nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      ports:
       - "8000:80"

    php-fpm:
      build: phpdocker/php-fpm
      container_name: radar-php-fpm
      working_dir: /application
      volumes:
        - .:/application
        - ./phpdocker/php-fpm/php-ini-overrides.ini:/etc/php/7.4/fpm/conf.d/99-overrides.ini


```

Remove the `webserver:` and `php-fpm:` sections
Remove the `working_dir: /application` under `mariadb:` section
Remove the `- .:/application` under the `volumes:` section in the `mariadb:` section
Add `- ./data/mariadb:/etc/mysql` under the `volumes:` section in the `mariadb:` section

You should be left with something that looks like this:

```yaml
###############################################################################
#                          Generated on phpdocker.io                          #
###############################################################################
version: "3.1"
services:

    mailhog:
      image: mailhog/mailhog:latest
      container_name: radar-mailhog
      ports:
        - "8001:8025"

    redis:
      image: redis:alpine
      container_name: radar-redis

    mariadb:
      image: mariadb:10.4
      container_name: radar-mariadb
      volumes:
        - ./data/mariadb:/etc/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=panama
        - MYSQL_DATABASE=radar
        - MYSQL_USER=radar
        - MYSQL_PASSWORD=panama
      ports:
        - "8003:3306"

    elasticsearch:
      image: elasticsearch:6.5.4
      container_name: radar-elasticsearch
```






