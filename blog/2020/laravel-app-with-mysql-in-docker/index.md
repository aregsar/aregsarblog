# Laravel App With MySQL In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show you how to run a MySQL database server in a local docker container. I will also show how you can configure the container to persist the database data using docker volumes so that the data is persistent across container restarts.

## Creating the docker-compose file

> Note: skip to the next section if you already have a docker-compose.yml file in the root directory

In this section we will create a new `docker-compose.yml` file in the Laravel project root directory:

```bash
touch docker-compose.yml
echo 'version: "3.1"' >> docker-compose.yml
echo 'services:' >> docker-compose.yml
```

## Creating the data directory

> Note: skip to the next section if you already have a data directory in the root directory

In this section we will create a new `/data` directory in the Laravel project root directory:

```bash
echo '/data' >> .gitignore
mkdir data
```

The data directory is where the MySQL data files will persist.
Since we don't want to commit the data files to our code repository, we need to add the the directory to our .gitignore file.

## Adding the mysql service to docker-compose

In this section we will add the MySQL docker compose service configuration to the `docker-compose.yml` file.

```yaml
mysql:
  image: mysql:8.0
  container_name: myapp-mysql
  volumes:
    - ./data/mysql:/var/lib/mysql
  environment:
    - MYSQL_ROOT_PASSWORD=myapp
    - MYSQL_DATABASE=myapp
    - MYSQL_USER=myapp
    - MYSQL_PASSWORD=myapp
  ports:
    - "8001:3306"
```

Note that the MySQL server port 3306 internal to the docker network is mapped to port 8001 on our localhost so that we can connect to the instance on localhost:8000.

Also Note that we have docker volumes mapping that maps the docker directory `/var/lib/mysql` where MySQL stores its data to the `./data/mysql` directory in our project data directory that we created on localhost. This is how the data is persisted to our local machine.

When we run the container for the first time, MySQL will automatically create a database named `myapp` since we provided the `MYSQL_DATABASE=myapp` environment variable.

## Adding environment variables to connect to the mysql service

In order for our application to connect to the MySQL server running in the docker container we need to configure environment variables and setup the application database connection configuration.

First need to set the following environment settings in the `.env` file:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
```

Next we need to update the mysql configuration settings in the `config\database.php` file:

```php
# the default database connection setting
'default' => env('DB_CONNECTION', 'mysql'),

'connections' => [
# the default database connection
'mysql' => [
        'driver' => 'mysql',
        'url' => env('DATABASE_URL'),
        'host' => env('DB_HOST', '127.0.0.1'),
        'port' => env('DB_PORT', '8001'),
        'database' => env('DB_DATABASE', 'myapp'),
        'username' => env('DB_USERNAME', 'myapp'),
        'password' => env('DB_PASSWORD', 'myapp'),
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
    ],
]
```

## Run the Docker services

Now we are ready to run the MySQL container by running the following bash command:

```bash
docker-compose up -d
```

All the containers should startup as can be seen by running the following command:

```bash
docker ps -a
```

In the following sections, I list various ways of connecting to the MySQL server running in the container. These include connecting with command line clients, connecting from php code from within our Laravel project as well as connecting with artisan Tinker and artisan database migrations.

## Connecting using mysqlsh cli

> Note: make sure no space between the -p option an password

Using mysqlsh cli:

```bash
sudo mysqlsh --sql -h localhost -P 8001 -u root -pmyapp -D myapp
```

```bash
sudo mysqlsh --sql -h localhost -P 8001 -u myapp -pmyapp -D myapp
```

## Connecting using mysql cli

> Note: make sure no space between the -p option an password

```bash
sudo mysql -h localhost -P 8001 -u root -pmyapp myapp
```

```bash
sudo mysql -h localhost -P 8001 -u myapp -pmyapp myapp
```

> Note you may be asked to type in your macOS password to execute the command

## Connecting with TablePlus to running mysql container

Open TablePlus and create a connection with the following:

click create a new connection
select MySQL database option
click create
type in `myapp` for the connection name

Enter the following credentials from the `.env` file:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
```

click on the test button to test connection
click connect button to connect to MySQL running in the container

## databases view as root user

Once connected as root we can execute a the show databases command to see all application and system databases. We would not see the system databases if we were connected as a non root user.

```bash
SQL > show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| radar              |
| sys                |
+--------------------+
```

## Test Connection using artisan tinker

```bash
php artisan tinker
>>> DB::connection()->getPdo();
=> PDO {#3043
     inTransaction: false,
     attributes: {
       CASE: NATURAL,
       ERRMODE: EXCEPTION,
       AUTOCOMMIT: 1,
       PERSISTENT: false,
       DRIVER_NAME: "mysql",
       SERVER_INFO: "Uptime: 59  Threads: 2  Questions: 9  Slow queries: 0  Opens: 116  Flush tables: 3  Open tables: 37  Queries per second avg: 0.152",
       ORACLE_NULLS: NATURAL,
       CLIENT_VERSION: "mysqlnd 7.4.4",
       SERVER_VERSION: "8.0.20",
       STATEMENT_CLASS: [
         "PDOStatement",
       ],
       EMULATE_PREPARES: 0,
       CONNECTION_STATUS: "127.0.0.1 via TCP/IP",
       DEFAULT_FETCH_MODE: BOTH,
     },
   }
>>>
```

## Test connection from Laravel application

```php
   try {
        var_dump(DB::connection()->getPdo());
    } catch (PDOException $e){
        var_dump($e->getMessage());
    }

```

## Running authentication database migrations

```bash
php artisan migrate
```

This command will user authentication tables by running the applications existing migration files against the MySQL database.

If we created the project UI using the `--auth` flag the application will contain the UI for registering a user and log in.

So we should be able to run the application and launch a browser to register a user and log in.

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome -a myapp.test
```

> Note: I am using Laravel Valet to host Laravel applications so I did not have to explicitly run the artisan serve command to serve the application.

## Persisting data between docker container runs

To check that the database data is persisted on our local machine we need to stop then restart the containers and check if the data is still available.

We can stop the contaier by running:

```bash
docker-compose down
```

If we try to connecting to the MySQL server now we should see it fail to connect since the server is not running anymore.

We need to check the `./data/mysql` directory where we should see the persisted database files.

We can then bring the MySQL service back up by running:

```bash
docker-compose up -d
```

And now we should see that the database connection is working again and that the tables and data that we previously had in the database should still be there.

## Executing and Testing the MySQL the container from the command line

```bash
#run mariadb container and run the mysql cli in the running container
docker-compose exec mysql mysql -u myapp -p myapp
# will display my_app_name database that was created upon starting the mysql service
show databases;
```

While mysql container is running cd into your laravel app that is configured to connect to the mysql instance running on this localhost container.

Now open a new terminal window and run:

```bash
php artisan tinker
DB::connection()->getPdo();
```

You should see the connection info.

Switch to the terminal window where the mysql container is running and type `exit` to exit the mysql cli and consequently stop the running container.

If this was the first time running the mysql container, back in the project root terminal window cd into the `./data/mysql/` directory to see the saved database files.
