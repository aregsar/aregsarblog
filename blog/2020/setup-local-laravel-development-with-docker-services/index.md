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
Add `./data/mariadb:/etc/mysql` under the `volumes:` section in the `mariadb:` section
Add `"8002:6379"` ports mapping under redis
Add `"8004:9300"` ports mapping under elasticsearch
Add `./data/redis:/etc/redis` volumes mapping section under redis

Finally change the environment values under `mariadb` to:

```ini
MYSQL_ROOT_PASSWORD=root
# for my_app_name substitute the name of your application
MYSQL_DATABASE=my_app_name
# for my_app_name substitute the name of your application
MYSQL_USER=my_app_name
MYSQL_PASSWORD=123456
```

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
      ports:
        - "8002:6379"
      volumes:
        - ./data/redis:/etc/redis

    mariadb:
      image: mariadb:10.4
      container_name: radar-mariadb
      volumes:
        - ./data/mariadb:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=123456
        - MYSQL_DATABASE=my_app_name
        - MYSQL_USER=my_app_name
        - MYSQL_PASSWORD=123456
      ports:
        - "8003:3306"

    elasticsearch:
      image: elasticsearch:6.5.4
      container_name: radar-elasticsearch
      ports:
        - "8004:9300"
```

In your Laravel applictaions `.env` file specify the following:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
# this is the left side of the mariadb port mapping in docker-compose.yml file
DB_PORT=8083
# for my_app_name substitute the name of the database that you want to be created 
DB_DATABASE=my_app_name
# for my_app_name substitute the name of your application
DB_USERNAMRE=my_app_name
DB_PASSWORD=123456

REDIS_HOST=127.0.0.1
REDIS_PORT=8002
REDIS_PASSWORD=123456
```

## Running the container

```bash
cd mylaravelapp
#run mariadb container and run the mysql cli in the running container
docker-compose exec mariadb mysql -u my_app_name -p 123456
# will display my_app_name database that was created upon starting the mariadb service
show databases;
#use my_app_name database
use my_app_name;
#should not contain any tables at this point
show tables;
```

While mariadb container is running cd into your laravel app that is configured to connect to the mariadb instance running on this localhost container.

Now open a new terminal window and run:

```bash
php artisan tinker
DB::connection()->getPdo();
```

You should see the connetion info.

Now switch to the terminal window where the mariadb continer is running and type `exit` to exit the mysql cli and consequently stop the running container.


