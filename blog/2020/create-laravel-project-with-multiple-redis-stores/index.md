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

The queue connections prefix appends a `QUEUE_PREFIX_VERSION` environment setting if it exists in the `.env` file. This is useful during immutable deployment of new versions of an application where each new version can use its own queue prefix so the old versions of the queue workers can be removed after the new versions of the queue workers have been deployed and after the old queue items that use the old prefix have been processed.

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

I we can see, similar to configuring the cache, the `redis` connection sets its `driver` to `redis` and sets its `connection` to the `queue` redis connection that we added to the `redis` driver in `config/database.php`.

## The final version of the configuration files

The final redis configuration settings for all redis related configuration files is shown below:

### config/database.php

```php
<?php

use Illuminate\Support\Str;

return [

    /*
    |--------------------------------------------------------------------------
    | Default Database Connection Name
    |--------------------------------------------------------------------------
    |
    | Here you may specify which of the database connections below you wish
    | to use as your default connection for all database work. Of course
    | you may use many connections at once using the Database library.
    |
    */

    'default' => env('DB_CONNECTION', 'mysql'),

    /*
    |--------------------------------------------------------------------------
    | Database Connections
    |--------------------------------------------------------------------------
    |
    | Here are each of the database connections setup for your application.
    | Of course, examples of configuring each database platform that is
    | supported by Laravel is shown below to make development simple.
    |
    |
    | All database work in Laravel is done through the PHP PDO facilities
    | so make sure you have the driver for your particular database of
    | choice installed on your machine before you begin development.
    |
    */

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

    /*
    |--------------------------------------------------------------------------
    | Migration Repository Table
    |--------------------------------------------------------------------------
    |
    | This table keeps track of all the migrations that have already run for
    | your application. Using this information, we can determine which of
    | the migrations on disk haven't actually been run in the database.
    |
    */

    'migrations' => 'migrations',

    /*
    |--------------------------------------------------------------------------
    | Redis Databases
    |--------------------------------------------------------------------------
    |
    | Redis is an open source, fast, and advanced key-value store that also
    | provides a richer body of commands than a typical key-value system
    | such as APC or Memcached. Laravel makes it easy to dig right in.
    |
    */

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

### config/cache.php

```php
<?php

use Illuminate\Support\Str;

return [

    /*
    |--------------------------------------------------------------------------
    | Default Cache Store
    |--------------------------------------------------------------------------
    |
    | This option controls the default cache connection that gets used while
    | using this caching library. This connection is used when another is
    | not explicitly specified when executing a given caching function.
    |
    | Supported: "apc", "array", "database", "file",
    |            "memcached", "redis", "dynamodb"
    |
    */
    'default' => env('CACHE_DRIVER', 'redis'),

    /*
    |--------------------------------------------------------------------------
    | Cache Stores
    |--------------------------------------------------------------------------
    |
    | Here you may define all of the cache "stores" for your application as
    | well as their drivers. You may even define multiple stores for the
    | same cache driver to group types of items stored in your caches.
    |
    */

    'stores' => [

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

        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
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

    /*
    |--------------------------------------------------------------------------
    | Cache Key Prefix
    |--------------------------------------------------------------------------
    |
    | When utilizing a RAM based store such as APC or Memcached, there might
    | be other applications utilizing the same cache. So, we'll specify a
    | value to get prefixed to all our keys so we can avoid collisions.
    |
    */

    'prefix' => '',

];
```

### config/session.php

```php
<?php

use Illuminate\Support\Str;

return [

    /*
    |--------------------------------------------------------------------------
    | Default Session Driver
    |--------------------------------------------------------------------------
    |
    | This option controls the default session "driver" that will be used on
    | requests. By default, we will use the lightweight native driver but
    | you may specify any of the other wonderful drivers provided here.
    |
    | Supported: "file", "cookie", "database", "apc",
    |            "memcached", "redis", "dynamodb", "array"
    |
    */

    'driver' => env('SESSION_DRIVER', 'redis'),

    /*
    |--------------------------------------------------------------------------
    | Session Lifetime
    |--------------------------------------------------------------------------
    |
    | Here you may specify the number of minutes that you wish the session
    | to be allowed to remain idle before it expires. If you want them
    | to immediately expire on the browser closing, set that option.
    |
    */

    'lifetime' => env('SESSION_LIFETIME', 120),

    'expire_on_close' => false,

    /*
    |--------------------------------------------------------------------------
    | Session Encryption
    |--------------------------------------------------------------------------
    |
    | This option allows you to easily specify that all of your session data
    | should be encrypted before it is stored. All encryption will be run
    | automatically by Laravel and you can use the Session like normal.
    |
    */

    'encrypt' => false,

    /*
    |--------------------------------------------------------------------------
    | Session File Location
    |--------------------------------------------------------------------------
    |
    | When using the native session driver, we need a location where session
    | files may be stored. A default has been set for you but a different
    | location may be specified. This is only needed for file sessions.
    |
    */

    'files' => storage_path('framework/sessions'),

    /*
    |--------------------------------------------------------------------------
    | Session Database Connection
    |--------------------------------------------------------------------------
    |
    | When using the "database" or "redis" session drivers, you may specify a
    | connection that should be used to manage these sessions. This should
    | correspond to a connection in your database configuration options.
    |
    */

    'connection' => 'session',

    /*
    |--------------------------------------------------------------------------
    | Session Database Table
    |--------------------------------------------------------------------------
    |
    | When using the "database" session driver, you may specify the table we
    | should use to manage the sessions. Of course, a sensible default is
    | provided for you; however, you are free to change this as needed.
    |
    */

    'table' => 'sessions',

    /*
    |--------------------------------------------------------------------------
    | Session Cache Store
    |--------------------------------------------------------------------------
    |
    | When using the "apc", "memcached", or "dynamodb" session drivers you may
    | list a cache store that should be used for these sessions. This value
    | must match with one of the application's configured cache "stores".
    |
    */

    'store' => env('SESSION_STORE', null),

    /*
    |--------------------------------------------------------------------------
    | Session Sweeping Lottery
    |--------------------------------------------------------------------------
    |
    | Some session drivers must manually sweep their storage location to get
    | rid of old sessions from storage. Here are the chances that it will
    | happen on a given request. By default, the odds are 2 out of 100.
    |
    */

    'lottery' => [2, 100],

    /*
    |--------------------------------------------------------------------------
    | Session Cookie Name
    |--------------------------------------------------------------------------
    |
    | Here you may change the name of the cookie used to identify a session
    | instance by ID. The name specified here will get used every time a
    | new session cookie is created by the framework for every driver.
    |
    */

    'cookie' => env(
        'SESSION_COOKIE',
        Str::slug(env('APP_NAME', 'laravel'), '_').'_session'
    ),

    /*
    |--------------------------------------------------------------------------
    | Session Cookie Path
    |--------------------------------------------------------------------------
    |
    | The session cookie path determines the path for which the cookie will
    | be regarded as available. Typically, this will be the root path of
    | your application but you are free to change this when necessary.
    |
    */

    'path' => '/',

    /*
    |--------------------------------------------------------------------------
    | Session Cookie Domain
    |--------------------------------------------------------------------------
    |
    | Here you may change the domain of the cookie used to identify a session
    | in your application. This will determine which domains the cookie is
    | available to in your application. A sensible default has been set.
    |
    */

    'domain' => env('SESSION_DOMAIN', null),

    /*
    |--------------------------------------------------------------------------
    | HTTPS Only Cookies
    |--------------------------------------------------------------------------
    |
    | By setting this option to true, session cookies will only be sent back
    | to the server if the browser has a HTTPS connection. This will keep
    | the cookie from being sent to you if it can not be done securely.
    |
    */

    'secure' => env('SESSION_SECURE_COOKIE'),

    /*
    |--------------------------------------------------------------------------
    | HTTP Access Only
    |--------------------------------------------------------------------------
    |
    | Setting this value to true will prevent JavaScript from accessing the
    | value of the cookie and the cookie will only be accessible through
    | the HTTP protocol. You are free to modify this option if needed.
    |
    */

    'http_only' => true,

    /*
    |--------------------------------------------------------------------------
    | Same-Site Cookies
    |--------------------------------------------------------------------------
    |
    | This option determines how your cookies behave when cross-site requests
    | take place, and can be used to mitigate CSRF attacks. By default, we
    | do not enable this as other CSRF protection services are in place.
    |
    | Supported: "lax", "strict", "none", null
    |
    */

    'same_site' => 'lax',

];
```

### config/queue.php

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Default Queue Connection Name
    |--------------------------------------------------------------------------
    |
    | Laravel's queue API supports an assortment of back-ends via a single
    | API, giving you convenient access to each back-end using the same
    | syntax for every one. Here you may define a default connection.
    |
    */
    'default' => env('QUEUE_CONNECTION', 'sync'),

    /*
    |--------------------------------------------------------------------------
    | Queue Connections
    |--------------------------------------------------------------------------
    |
    | Here you may configure the connection information for each server that
    | is used by your application. A default configuration has been added
    | for each back-end shipped with Laravel. You are free to add more.
    |
    | Drivers: "sync", "database", "beanstalkd", "sqs", "redis", "null"
    |
    */

    'connections' => [

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

    ],

    /*
    |--------------------------------------------------------------------------
    | Failed Queue Jobs
    |--------------------------------------------------------------------------
    |
    | These options configure the behavior of failed queue job logging so you
    | can control which database and table are used to store the jobs that
    | have failed. You may change them to any database / table you wish.
    |
    */

    'failed' => [
        'driver' => env('QUEUE_FAILED_DRIVER', 'database'),
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'failed_jobs',
    ],

];
```

## Testing the redis connections from a controller

Add the following route:

```php
Route::get('/redis', 'RedisController@redisTest');
```

Add a controller named `RedisController` then add the a `redisTest` method to it with the following code:

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

Navigating to `/redis` will run the code.
