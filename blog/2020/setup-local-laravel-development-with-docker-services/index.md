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



mysql -u root -p
mysql> select @@datadir;
# /var/lib/mysql/

# for mariadb on macOD
# /usr/local/var/mysql/



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




