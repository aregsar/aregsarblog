# My Laravel Local Docker Services Configuration

April 29, 2020 by [Areg Sarkissian](https://aregsar.com/about)

To run the MySql and Redis servers for your local Laravel project environment to connect to, add the following `docker-compose.yml` file to the root of your Laravel project:

```yaml
version: "3.1"
services:
    myapp-redis:
      image: redis:alpine
      container_name: app-redis
      command: redis-server --appendonly yes --requirepass 123456
      volumes:
      - ./data/redis:/data
      ports:
        - "8000:6379"

    myapp-mysql:
      image: mysql:8.0
      container_name: app-mysql
      volumes:
        - ./data/mysql:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=root
        # for MYSQL_DATABASE substitute the name of the database that you want to be created
        - MYSQL_DATABASE=myapp
        - MYSQL_USER=root
        - MYSQL_PASSWORD=123456
      ports:
        - "8001:3306"
```

> Note: the `redis-server –appendonly yes` command overrides the redis-server command with no arguments that the standard `redis:alpine` container runs. This allows us to persist the redis data using a docker volume mapping.

If we look at the standard mysql and redis dockerfiles we can see that there is a VOLUME defined that sets the mysql data directory to `/var/lib/mysql` and the redis data directory to `/data` in the container. By default when the container runs for the first time that volume is created within the container and the mysql and redis data files are persisted in those volume directories. By mapping the redis container `/data` directory to our docker host machine directory `./data/redis` and mapping the mysql container `/var/lib/mysql` directory to the host machine `./data/mysql` directory we can persist that data across docker container runs.

> Note: the host machine `./data/redis` and `./data/mysql` volumes will be created within the Laravel project directory that contains the docket-compose.yml file when we run the `docker-compose up -d` command

## Environment Variables for the Redis server container

In your Laravel application `.env` file specify the following:

```ini
# connecting to local instance of redis docker container
REDIS_HOST=127.0.0.1
# this is the left side of the redis container port mapping in docker-compose.yml file
REDIS_PORT=8000
REDIS_PASSWORD=123456
```

## Environment Variables for the MySql server container

In your Laravel application `.env` file specify the following:

```ini
# connecting to local instance of mysql docker container
DB_HOST=127.0.0.1
# this is the left side of the mysql container port mapping in docker-compose.yml file
DB_PORT=8001
# for my_app_name substitute the name of the database that you want to connect to.
# must match the MYSQL_DATABASE setting value for mysql container
DB_DATABASE=myapp
# must match the MYSQL_USER setting value for mysql container
DB_USERNAMRE=root
# must match the MYSQL_PASSWORD setting value for mysql container
DB_PASSWORD=123456
# set to name of `mysql` connection setting within the 'connections' setting in config/database.php
DB_CONNECTION=mysql
```

> Note the `mysql` connection setting within the `config/database.php` loads the database credentials using the DB_DATABASE, DB_USERNAMRE and DB_PASSWORD environment settings in the `.env` file

## The Laravel MySql configuration setting

For reference this is the mysql connection in the `config/database.php` file

```php
 'mysql' => [
            'driver' => 'mysql',
            # if env key does not exist env($nonexistingkey) returns null by default  
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
```
