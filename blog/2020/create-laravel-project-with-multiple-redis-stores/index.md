# Create Laravel Project With Multiple Redis Stores

April 18, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show how to create a production oriented redis configuration. The configuration will work locally and with standard or clustered Redis servers in production. 

The configuration will use separate connections each with their own  unique prefixes for session, cache , queue and application connections to distinguish between items stored for different purposes.

Also the configuration with separate connections, will be ready to scale to separate Redis server instances in production for the session, cache, queue and application in the future, simply by changing the connection parameters.

## Configuring individual redis database connections in config/database.php

This section I will show how to configure a separate connection for each of the Redis, Session, Cache and Queue providers used in a Laravel application.

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

The `default` connection has a `database` default of `0` and the `cache` connection has a `database` default of `1` to assign separate databases to those connections that are configured to use the same Redis server.

The `cache` connection in this `config/database.php` is set to be the default cache connection in the `config/cache.php` file.

The `default` connection will be used for all in app access to Redis that directly use the Laravel `Redis` provider.

Unlike the cache and queue configuration files, there is no configuration file that has a default connection setting that is set to the `default` redis connection in config/database.php.

So by default when we use the Laravel `Redis` facade,global helper or provider without specifying the redis connection to use, the redis connection in config/database.php that is labeled `default` will be used.

We need to add two more connections named `queue` and `session` for the Laravel `Session` and `Queue` providers to use. We also need to modify the `cache` connection to use the same redis connection `database` setting as all the other redis connections. This is because redis clusters used in production do not support multiple redis databases. We also need to add a `prefix` setting to all the connections to identify each separately.

For these reasons we will remove the out of box redis connections in database.php and replace them with the four connections below:

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

As you can see all the `database` settings for all connections is set to 0 and we added an additional `prefix` setting for each connection, in order to separate our redis, cache, session and queue keys. Each connection prefix uses a unique letter to effectively namespace qualify the key used by the client by appending the prefix to the key.

> Note that the `redis` => `options` => `prefix` setting was removed since it only applies if we wanted to use the old `predis` composer package redis client. The `prefix` setting is not required since we are using  the default `php-redis` php extension client.

## The redis cluster setting

The `redis` => `options` => `cluster` setting, indicates that our Redis server connection is connecting to a self managed Redis cluster.

A self managed cluster connection connects to a proxy cluster manager that that manages a cluster of redis instances, without requiring the redis client to manage the cluster nodes.

Its important to note that clustered redis instances do not support multiple databases. They only have one database which has the default database number 0. That is why we set the `database` setting for all the connections to 0.

> Note: If a redis cluster is not being used, the `redis` => `options` => `cluster` setting can be removed, however it will not effect using a standard redis instance if it remains.

## The Redis Client

The default out of the box Laravel configuration is configured to use the Php Redis extension.

`'client' => env('REDIS_CLIENT', 'phpredis')`

For that to work we need to make sure the Php Redis extension is installed.

The instructions for doing so are here: `put link here`

Also unless we rename the facade alias we need to add a 
`use Illuminate\Support\Facades\Redis;` statement in the file we use the `Redis` facade in.

## Configuring the Session Configuration file to use redis

Change the `driver` configuration in `config/session.php` to:

```php
//selects the 'redis' driver in config/database.php
 'driver' => env('SESSION_DRIVER', 'redis'),
```

Change the `connection` configuration in `config/session.php` to:

```php
//selects the 'session' connection of the 'redis' driver in config/database.php
'connection' => 'session',
```

## Configuring the Cache Configuration file to use redis

Change the `default` store configuration in `config/cache.php` to:

```php
//selects the 'redis' cache store in config/cache.php
'default' => env('CACHE_DRIVER', 'redis'),
```

Change the `prefix` configuration in `config/cache.php` to:

```php
'prefix' => '',
```

We can leave the redis store connection value in `config/cache.php` as is, since it is set to `cache` out of the box which references the out of the box `cache` redis connection in `config/database.php`

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

## Configuring the Queue Configuration file to use redis

Change the `default` configuration in `config/queue.php` to:

```php
//selects the 'redis' queue connection in config/queue.php
'default' => env('QUEUE_CONNECTION', 'redis'),
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

## Setting up the environment variables for our configuration file to use

The following environment variable need to be set to for the Redis configuration to use.

```ini
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
QUEUE_PREFIX_VERSION=V1:
```

Additionally we need to set the host and password settings. The values set below are for a local Redis instance running:

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

And these are Digitalocean Redis cluster settings:

```ini
REDIS_HOST=tls://<your_redis_host>.db.ondigitalocean.com
REDIS_PASSWORD=<your_redis_password>
REDIS_PORT=25061
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

