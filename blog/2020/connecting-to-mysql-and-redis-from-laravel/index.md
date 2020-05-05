# connecting to mysql and redis from laravel

[connecting to mysql and redis from laravel](https://aregsar.com/blog/2020/connecting-to-mysql-and-redis-from-laravel)

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


========================

echo "/data" >> .gitignore
mkdir data
touch docker-compose.yml


mysql:
  image: mysql:8.0
  container_name: radar-mysql
  volumes:
    - ./data/mysql:/var/lib/mysql
  environment:
    - MYSQL_ROOT_PASSWORD=radar
    - MYSQL_DATABASE=radar
    - MYSQL_USER=radar
    - MYSQL_PASSWORD=radar
  ports:
    - "8001:3306"


DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_DATABASE=radar
DB_USERNAME=radar
DB_PASSWORD=radar

'mysql' => [
        'driver' => 'mysql',
        'url' => env('DATABASE_URL'),
        'host' => env('DB_HOST', '127.0.0.1'),
        'port' => env('DB_PORT', '8001'),
        'database' => env('DB_DATABASE', 'radar'),
        'username' => env('DB_USERNAME', 'radar'),
        'password' => env('DB_PASSWORD', 'radar'),
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


docker-compose up -d

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



sudo mysqlsh --sql -h localhost -P 8001 -u root -pradar -D radar
sudo mysqlsh --sql -h localhost -P 8001 -u radar -pradar -D radar

#sudo mysql -h localhost -P 8001 -u radar -pradar radar
#sudo mysql -h localhost -P 8001 -u root -pradar radar




type your macOS password

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


php artisan migrate

/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome -a radar.test


docker-compose down


