# Create Laravel Project With Multiple Redis Stores

April 18, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show how to create a production oriented Redis configuration in Laravel. 

The configuration will work locally and with standard or clustered Redis servers in production.

The configuration will configure separate redis connections for the Laravel session, cache, queue and application. In development all the connections will point to the same Redis server but will distinguish between objects stored by each connection by configuring a unique prefix for each connection. 

This same configuration can be used in production using a single Redis server. However it allows scaling the application to use separate Redis servers by making each connection connect to a different Redis server and even by adding more connections to split the Laravel cache and/or queue into separate caches and/or queues.

## The out of the box connection configuration

When we create a new Laravel project, the `config/database.php` file will contain two out of the box redis connections named `default` and `cache`:

```bash
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],
    ],
```

Both connections are configured to use the same Redis server.

The `default` connection has a `database` default of `0` and the `cache` connection has a `database` default of `1` assigning separate databases to each connection.

> When we change this configuration we will make all connections use database 0 because Redis clusters do not support more than one database.

The `default` connection will be used by default to access Redis when using the Laravel `Redis` provider in the application. By default when we use the Laravel `Redis` facade or helper without specifying the redis connection to use, the redis connection that is labeled `default` in `config/database.php` will be used.

## Reconfiguring Redis with separate connections

We want to configure connections for the Laravel `session`, `cache` and `queue` in `config/database.php`. In addition we want to have a separate connection for the application to use via the `Redis` facade.

We will keep the `default` connection for our application to use and we will keep the `cache` connection for the Laravel cache service to use. However we will change their configuration to use the same database and add a unique `prefix` setting to each.

We will also add two more connections named `queue` and `session` for the Laravel Session and Queue services to use. We will use the same database setting as all other connections for these and we will give them a each unique `prefix` setting.

The modified configuration in `config/database.php` looks like:

```php
    //the redis driver
    'redis' => [
        //the redis client (requires installing the phpredis.so extension using pecl)
        'client' => 'phpredis',

        'options' => [
            //this setting is only effective when using a managed redis cluster. No impact if redis cluster is not used.
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],
        //connection used by the redis facade
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'd:',
        ],
        //connection used by the cache facade when redis cache is configured in config/cache.php
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],
        //connection used by the session when redis cache is configured in config/session.php
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 's:',
        ],
        //connection used by the queue when redis cache is configured in config/queue.php
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:'.env('QUEUE_PREFIX_VERSION', ''),
        ],
    ],
```

As you can see all the `database` settings for all connections is set to 0 and we added an additional `prefix` setting for each connection, in order to separate our redis, cache, session and queue keys.

Each connection prefix uses a unique letter to effectively namespace qualify the key used by the redis client by prepending the prefix to the key.

> Note that the `'redis' => ['options' => [ 'prefix' => ...]]` setting in the out of the box `config/database.php` was removed since it only applies if we want to use the old `predis` composer package redis client.

## The redis cluster setting

The `'redis' => ['options' => [ 'cluster' => env('REDIS_CLUSTER', 'redis'),]]` setting, indicates that our Redis server connection is connecting to a self managed Redis cluster.

A self managed cluster connection connects to a cluster manager proxy that manages a cluster of Redis server instances, without requiring the redis client to manage the cluster nodes.

Its important to note that clustered Redis instances do not support multiple databases. They can only have one database which has the default database number 0. That is why we set the `database` setting for all the connections to 0.

> Note: If a Redis cluster is not being used, the `'redis' => ['options' => [ 'cluster' => env('REDIS_CLUSTER', 'redis'),]]` setting can be removed, however it will not effect using a standard redis instance if it remains.

## The Redis Client

The default out of the box Laravel configuration is configured to use the Php Redis extension.

`'client' => env('REDIS_CLIENT', 'phpredis')`

For that to work we need to make sure the Php Redis extension is installed.

The instructions for doing so are at [Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis)

Also when using the `Redis` facade class, unless we rename the facade alias as shown in the post from the link above, we need to use the fully qualified class name `Illuminate\Support\Facades\Redis`.

## Setting up the environment variables for our configuration file to use

We need to set the host and password settings used by the redis connection configuration in `config/database.php` file.

The values set below in `.env` file are for a local Redis instance running:

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

For Digitalocean Redis cluster production environment we need to change the values to:

```ini
REDIS_HOST=tls://<your_redis_host>.db.ondigitalocean.com
REDIS_PASSWORD=<your_redis_password>
REDIS_PORT=25061
```

## The session, cache and queue configuration files

The Laravel session, cache and queue each have their own individual configuration files that specify which driver and driver connection from the `config/database.php` file to use.

In the following sections I show how to configure each to use the `redis` driver and which `redis` driver connection to use as their default redis connection.

## Configuring the Session Configuration file to use redis

The `driver` configuration in `config/session.php` selects the driver to use for the Laravel session.

```php
//will select the 'redis' driver in config/database.php. defaults to the file driver
 'driver' => env('SESSION_DRIVER', 'file'),
```

To make the driver select the `redis` driver we need to update the `.env` file setting:

```ini
SESSION_DRIVER=redis
```

Once we are using the `redis` driver, we can set the `connection` setting in `config/session.php` to the the `session` redis connection that we added to the `redis` driver in `config/database.php`:

```php
//sets the 'connection' to the 'session' connection of the 'redis' driver in config/database.php
'connection' => 'session',
```

## Configuring the Cache Configuration file to use redis

The `config/cache.php` configuration file specifies one or more stores that the Laravel cache service can use.

The `default` configuration in `config/cache.php` selects the default cache store to use by the Laravel cache service.

Each cache store specifies the driver to use for that store.

> Note: Unlike the Laravel session that can only have one store to use per application. The cache and queue can have more than one store and so must select a default store to use. This is why for the session we specify the driver to use directly at the top level of the configuration file. Whereas for the cache and queue we need to specify the driver to use at the cache store level

The out of the box cache configuration file `config/cache.php` defaults to the file store, storing each cached object in a file.

```php
//selects the 'redis' cache store in config/cache.php
'default' => env('CACHE_DRIVER', 'file'),
```

To make the default store select the `redis` store from the `stores` configuration in `config/cache.php` we need to update the `.env` file setting:

```ini
CACHE_DRIVER=redis
```

Now that we are using the `redis` store, we can set the `redis` store `driver` to `redis` to make the `redis` store use the `redis` driver from `config/database.php`.

We need to also set the `redis` store `connection` to the `cache` redis connection that we added to the `redis` driver in `config/database.php`:

```php
    'stores' => [
        'redis' => [
            //selects the 'redis' driver in config/database.php
            'driver' => 'redis',
            //selects the connection 'cache' of the 'redis' driver in config/database.php
            'connection' => 'cache',
        ],
    ],
```

Also we can change the `prefix` configuration setting in `config/cache.php` as shown:

```php
'prefix' => '',
```

This prefix setting is not used with the redis driver so it is safe to set it to empty string.

=======================================================

## Configuring the Queue Configuration file to use redis

Change the `default` configuration in `config/queue.php` to:

```php
//selects the 'redis' queue connection in config/queue.php
'default' => env('QUEUE_CONNECTION', 'sync'),
```

To make the default queue connection select the `redis` store we need to update the `.env` file setting:

```ini
QUEUE_CONNECTION=redis
```

Change the `connections` configuration in `config/queue.php` to:

```php
      'connections' => [
        //queue connection
        'redis' => [
            //selects the 'redis' driver in config/database.php
            'driver' => 'redis',
            //selects the connection 'queue' of the redis driver in config/database.php
            'connection' => 'queue',
            //default queue for this 'redis' queue connection. By setting the redis key prefix.
            'queue' => '{job}',
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
```

## The final redis configuration

The final redis configuration settings for all redis related configuration files is shown below:

`config/database.php`:

```php
<?php

use Illuminate\Support\Str;

return [

    'default' => env('DB_CONNECTION', 'mysql'),

    'connections' => [

        'sqlite' => [
            'driver' => 'sqlite',
            'url' => env('DATABASE_URL'),
            'database' => env('DB_DATABASE', database_path('database.sqlite')),
            'prefix' => '',
            'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
        ],

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

        'pgsql' => [
            'driver' => 'pgsql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '5432'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'prefix' => '',
            'prefix_indexes' => true,
            'schema' => 'public',
            'sslmode' => 'prefer',
        ],

        'sqlsrv' => [
            'driver' => 'sqlsrv',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', 'localhost'),
            'port' => env('DB_PORT', '1433'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'prefix' => '',
            'prefix_indexes' => true,
        ],
    ],

    'migrations' => 'migrations',

    //the redis driver
    'redis' => [
        //the redis client (requires installing the phpredis.so extension using pecl)
        'client' => 'phpredis',

        'options' => [
            //this setting is only effective when using a managed redis cluster. No impact if redis cluster is not used.
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],
        //connection used by the redis facade
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'd:',
        ],
        //connection used by the cache facade when redis cache is configured in config/cache.php
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],
        //connection used by the session when redis cache is configured in config/session.php
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 's:',
        ],
        //connection used by the queue when redis cache is configured in config/queue.php
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:'.env('QUEUE_PREFIX_VERSION', ''),
        ],
    ],
];
```

`config/cache.php`:

```php
<?php

use Illuminate\Support\Str;

return [

    'default' => env('CACHE_DRIVER', 'redis'),

    'stores' => [

        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],

        'apc' => [
            'driver' => 'apc',
        ],

        'array' => [
            'driver' => 'array',
            'serialize' => false,
        ],

        'database' => [
            'driver' => 'database',
            'table' => 'cache',
            'connection' => null,
        ],

        'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],

        'memcached' => [
            'driver' => 'memcached',
            'persistent_id' => env('MEMCACHED_PERSISTENT_ID'),
            'sasl' => [
                env('MEMCACHED_USERNAME'),
                env('MEMCACHED_PASSWORD'),
            ],
            'options' => [
                // Memcached::OPT_CONNECT_TIMEOUT => 2000,
            ],
            'servers' => [
                [
                    'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                    'port' => env('MEMCACHED_PORT', 11211),
                    'weight' => 100,
                ],
            ],
        ],

        'dynamodb' => [
            'driver' => 'dynamodb',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
            'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
            'endpoint' => env('DYNAMODB_ENDPOINT'),
        ],
    ],

    'prefix' => '',
];
```

`config/session.php`:

```php
<?php

use Illuminate\Support\Str;

return [

    'driver' => env('SESSION_DRIVER', 'redis'),

    'connection' => 'session',

    'lifetime' => env('SESSION_LIFETIME', 120),

    'expire_on_close' => false,

    'encrypt' => false,

    'files' => storage_path('framework/sessions'),

    'table' => 'sessions',

    'store' => env('SESSION_STORE', null),

    'lottery' => [2, 100],

    'cookie' => env(
        'SESSION_COOKIE',
        Str::slug(env('APP_NAME', 'laravel'), '_').'_session'
    ),

    'path' => '/',

    'domain' => env('SESSION_DOMAIN', null),

    'secure' => env('SESSION_SECURE_COOKIE'),

    'http_only' => true,

    'same_site' => 'lax',
];
```

`config/queue.php`:

```php
<?php

return [

    'default' => env('QUEUE_CONNECTION', 'job'),

    'connections' => [
        //this is the 'job' queue connection
        'job' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'queue' connection from the redis driver in config/database.php
            'connection' => 'queue',
            //this is the redis queue key default prefix that is applied when using this 'job' connection. It can be overriden by explicitly passing the queue name.
            'queue' => '{job}',
            'retry_after' => 90,
            'block_for' => null,
        ],
        //this is the 'app' queue connection
        'app' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'queue' connection from the redis driver in config/database.php
            'connection' => 'queue',
            //this is the redis queue key default prefix that is applied when using this 'app' connection.It can be overriden by explicitly passing the queue name.
            'queue' => '{app}',
            'retry_after' => 90,
            'block_for' => null,
        ],

        'sync' => [
            'driver' => 'sync',
        ],

        'database' => [
            'driver' => 'database',
            'table' => 'jobs',
            'queue' => 'default',
            'retry_after' => 90,
        ],

        'beanstalkd' => [
            'driver' => 'beanstalkd',
            'host' => 'localhost',
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => 0,
        ],

        'sqs' => [
            'driver' => 'sqs',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
            'queue' => env('SQS_QUEUE', 'your-queue-name'),
            'suffix' => env('SQS_SUFFIX'),
            'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        ],
    ],

    'failed' => [
        'driver' => env('QUEUE_FAILED_DRIVER', 'database'),
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'failed_jobs',
    ],
];
```

## Testing the redis connections

The following code can be run using Laravel tinker:

An now we can test the connections add the following route to a new controller that you can add named `RedisController`:

```php
Route::get('/redis', 'RedisController@redisTest');
```

Then add the following `redisTest` method to the controller:

```php
//in RedisController file
use Illuminate\Support\Facades\Redis;

//in RedisController class
public function redisTest()
{
    $redis = Redis::connection();

    try{
        var_dump($redis->ping());

        Redis::incr('visits');
        Redis::set('name', 'areg');
        $name = Redis::get('name');
        $nameexists = Redis::exists('name');

        Session::all();
        Session::put('name', 'areg');
        $sname = Session::get('name');
        Session::forget('name');
        $snameexists = Session::exists('name');

        Cache::put('name','areg',60);
        $cname = Cache::get('name');
        Cache::put('name2','areg2',now()->addDay());
        Cache::forget('name');
        //$cnameexists = Cache::has('name');
        Cache::get('name3', fn () => 'areg3');
        //Cache::Clear();

    } catch (Exception $e){
        $e->getMessage();
    }
}
```
