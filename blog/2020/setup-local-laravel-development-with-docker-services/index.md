# Setup Local Laravel Development With Docker Services

April 23, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I will show how I run Docker services that support my locally installed Laravel applications.

I run the following services in individual Docker containers to support local Laravel application development:

+ MySQL database
+ Redis used for session, cache and queued job storage
+ MailHog Mail server to capture emails and view in dashboard
+ Elastic Search

> Note: Except for MailHog, these are the same services I use in production to maintain dev/prod parity

All the containers are run via Docker compose.

## Testing the mariadb the container

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

Now back in the application terminal window cd into the `data/mariadb` directory to see the saved database files.

## running all the containers

In the application directory, type:

```bash
docker-compose up -d
```

All the containers should startup as can be seen by running the following command:

```bash
docker ps -a
```

## Connecting with PHP to running MySQL container

Now open a new terminal window and run:

```bash
php artisan tinker
DB::connection()->getPdo();
```

Again you should see the connection info response.

You should be able to run migrations against the MySQL container:

```bash
php artisan migrate
```

## Connecting with TablePlus to running MariaDb container

Open TablePlus and create a connection with the following:

click create a new connection
select mysql
click create
type in my_app_name for the connection name

Enter the following credentials:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8083
DB_DATABASE=my_app_name
DB_USERNAME=my_app_name
DB_PASSWORD=123456
```

click on the test button to test connection
click connect button to connect to MySQL running in the container

## Connecting with local mysql CLI to running MariaDb container

```bash
mysql -u my_app_name -p -h 127.0.0.1 -P 8083
# the -D flag specifies the default database to connect to
#mysql -u my_app_name -p -h 127.0.0.1 -D my_app_name -P 8083
#show directory where the database files are stored
MariaDB> select @@datadir;
# /usr/local/var/mysql/
#show connected user
MariaDB> SELECT USER();
#show selected database
MariaDB> SELECT DATABASE();
# NULL
MariaDB> use my_app_name
MariaDB> SELECT DATABASE();
```

click on the test button to test connection
click connect button to connect to mariadb running in the container

## Connecting with local redli CLI to running redis container

```bash
#redli -h host -a password -p port
redli -h 127.0.0.1 -a 123456 -p 8002
127.0.0.1:8002> ping
# pong
127.0.0.1:8002> set tst:test "abcd"
127.0.0.1:8002> get tst:test
# abcd
127.0.0.1:8002> exit
# 127.0.0.1:8002> quit
```

## Connecting with local redis-cli CLI to running redis container

Testing redis with following commands

```bash
redis-cli -p 8002
127.0.0.1:8002> ping
# pong
127.0.0.1:8002> set tst:test "abcd"
127.0.0.1:8002> get tst:test
# abcd
127.0.0.1:8002> exit
# 127.0.0.1:8002> quit
```

## location of mysql data and config files within container

```bash
mysql -u my_app_name -p -h 127.0.0.1 -P 8083
#show directory where the database files are stored
> select @@datadir;
# on macOS
# /usr/local/var/mysql/
```

```bash
#show where the configuration files are located
mysql --help | grep "Default options" -A 1
# on macOS
# /usr/local/etc/my.cnf
```
