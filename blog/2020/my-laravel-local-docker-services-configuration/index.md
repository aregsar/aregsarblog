# My Laravel Local Docker Services Configuration

April 29, 2020 by [Areg Sarkissian](https://aregsar.com/about)

To run MySql and Redis servers for your local Laravel project development environment, add the following `docker-compose.yml` file to the root of your Laravel project:

```yaml
version: "3.1"
services:
    myapp-redis:
      image: redis:alpine
      container_name: app-redis
      command: redis-server --appendonly yes --requirepass "${REDIS_PASSWORD}"
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
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # upon container first run a database with the name of MYSQL_DATABASE setting will be created
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        - "8001:3306"
```

> Note: the `redis-server –appendonly yes` command overrides the redis-server command with no arguments that the standard `redis:alpine` container runs. This allows us to persist the redis data using a docker volume mapping by configuring redis to asynchronously backup its in memory data to disk.

If we look at the Alpine MySQL dockerfile in docker hub we can see that there is a `VOLUME` defined that sets the mysql data directory to `/var/lib/mysql` in the container.
By default when the container runs for the first time, the `VOLUME` defined in the dockerfile is created within the container and the mysql data files are persisted volume directory.
By mapping the mysql container `/var/lib/mysql` directory to the host machine `./data/mysql` directory we can persist that data across docker container runs.

Similarly, if we look at the Alpine Redis dockerfile in docker hub we can see that there is a `VOLUME` defined that sets the redis data directory to `/data` in the container.
By default when the container runs for the first time, the `VOLUME` defined in the dockerfile is created within the container and the redis data files are persisted volume directory.
By mapping the redis container `/data` directory to our docker host machine `./data/redis` directory we can persist that data across docker container runs.

> Note: the host machines `./data/redis` and `./data/mysql` volumes will be created within the Laravel project directory that contains the `docket-compose.yml` file when we run the `docker-compose up -d` command.

## Environment Variables for the Redis server container

In your Laravel application `.env` file specify the following:

```ini
# connecting to local instance of redis docker container
REDIS_HOST=127.0.0.1
# this is the left side of the redis container port mapping in docker-compose.yml file
REDIS_PORT=8000
REDIS_PASSWORD=123456
```

Note that the `docker-compose.yml` file will read the `REDIS_PASSWORD` value from the .env file.

Also the `redis` connection setting within the `config/database.php` loads the database credentials using the REDIS_HOST, REDIS_PORT and REDIS_PASSWORD environment settings in the `.env` file.

For reference this is a sample redis connection configuration from the `config/database.php` file:

```php
    'redis' => [
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            'prefix' => '',
        ],
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

Note the `mysql` connection setting within the `config/database.php` loads the database credentials using the DB_DATABASE, DB_USERNAMRE and DB_PASSWORD environment settings in the `.env` file

For reference this is a sample mysql connection connection in the `config/database.php` file:

```php
 'mysql' => [
            'driver' => 'mysql',
            # if env key does not exist env($nonexistingkey) returns null by default  
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'myapp'),
            'username' => env('DB_USERNAME', 'myapp'),
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
