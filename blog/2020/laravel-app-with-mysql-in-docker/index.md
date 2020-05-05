# Laravel App With MySQL In Docker

May 5, 2020



echo "/data" >> .gitignore
mkdir data
touch docker-compose.yml


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


DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp


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



There are the relevent settings:

'host' => env('DB_HOST', '127.0.0.1'),
'port' => env('DB_PORT', '8001'),
'database' => env('DB_DATABASE', 'myapp'),
'username' => env('DB_USERNAME', 'myapp'),
'password' => env('DB_PASSWORD', 'myapp'),


docker-compose up -d


## Connecting using mysql and mysqlsh client

> Note: make sure no space between the -p option an password

Using mysqlsh cli:

sudo mysqlsh --sql -h localhost -P 8001 -u root -pmyapp -D myapp
sudo mysqlsh --sql -h localhost -P 8001 -u myapp -pmyapp -D myapp




Using mysql cli:

sudo mysql -h localhost -P 8001 -u root -pmyapp myapp
sudo mysql -h localhost -P 8001 -u myapp -pmyapp myapp

> Note you may be asked to type in your macOS password to execute the command


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


## Test Connection using artisan tinker

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



## Test connection with Laravel route closure


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


## Running laravel authentication database migrations 


php artisan migrate

/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome -a radar.test

Register a user and log in

## Persisting data between docker container runs

docker-compose down

Try connecting to see failure

docker-compose up -d

See logged in user again

Check data/mysql directory to see the persisted database files



