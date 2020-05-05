# Laravel App With MySQL In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Creating files and directories

> Note if you have already a docker-compose.yml then these steps are not required

echo '/data' >> .gitignore
mkdir data
touch docker-compose.yml
echo `version: "3.1"' >> docker-compose.yml
echo `services:' >> docker-compose.yml

## Adding the mysql service to docker-compose

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

## Adding the environment variables for the mysql service

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
```

## Changing mysql configuration settings for the mysql service

```php
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
```

There are the relevent settings:

```php
'host' => env('DB_HOST', '127.0.0.1'),
'port' => env('DB_PORT', '8001'),
'database' => env('DB_DATABASE', 'myapp'),
'username' => env('DB_USERNAME', 'myapp'),
'password' => env('DB_PASSWORD', 'myapp'),
```

## Run the Docker services

```bash
docker-compose up -d
```

## Connecting using mysql and mysqlsh client

> Note: make sure no space between the -p option an password

Using mysqlsh cli:

```bash
sudo mysqlsh --sql -h localhost -P 8001 -u root -pmyapp -D myapp
```

```bash
sudo mysqlsh --sql -h localhost -P 8001 -u myapp -pmyapp -D myapp
```

Using mysql cli:

```bash
sudo mysql -h localhost -P 8001 -u root -pmyapp myapp
```

```bash
sudo mysql -h localhost -P 8001 -u myapp -pmyapp myapp
```

> Note you may be asked to type in your macOS password to execute the command

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
```

```bash
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
                $db = DB::connection()->getPdo();
            }
            catch (PDOException $e) {
                self::fatal(
                    "An error occurred while connecting to the database. ".
                    "The error reported by the server was: ".$e->getMessage()
                );
            }
```

## Running authentication database migrations

```bash
php artisan migrate
```

Launch the browser

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome -a myapp.test
```

> Note: I am using Laravel Valet to host Laravel applications

Register a user and log in

## Persisting data between docker container runs

```bash
docker-compose down
```

Try connecting to see failure

```bash
docker-compose up -d
```

See logged in user again

Check data/mysql directory to see the persisted database files
