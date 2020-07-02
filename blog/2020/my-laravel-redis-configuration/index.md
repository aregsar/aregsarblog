# My Laravel Redis Configuration

April 28, 2020 by [Areg Sarkissian](https://aregsar.com/about)

This post contains all the configuration changes required to use Redis for Laravel application session, cache and queue services in addition to directly accessing Redis for application specific use cases using the Redis facade.

## The Laravel Redis configuration

Below is how the `redis` database driver is configured in `config/database.php`

```php
 //the redis client (requires installing the phpredis.so extension using pecl)
 'client' => 'phpredis',

 //the redis driver
 'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'options' => [
            'cluster' => 'redis',
        ],
        //the `default` connection of the redis driver
        //Note: The Laravel Redis facade is hard coded to use this connection unless its connection is explicitly specified
        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            `prefix` => `d:`
        ],
        //the `cache` connection of the redis driver
        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            `prefix` => `c:`
        ],
        //the `queue` connection of the redis driver
        'queue' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            `prefix` => 'q:',
        ],
        //the `session` connection of the redis driver
        'session' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            #since config/session.php does not have an app level prefix we can change
            #this prefx to `myapp:s:` if we share redis server between apps
            `prefix` => `s:`
        ],
    ],
```

> Note: the `prefix` setting is used instead of using different `database` number to be able to segment the keys. In production I use a managed redis cluster which does not support multiple databases so I avoid using more than one database in the redis server.

## Redis configuration environment variable settings

The `.env` file in the project root contains the environment variable settings for the `config/database.php` configuration file.

Below are the settings used for my development environment that connect to a local docker container running a Redis service exposed on port 8002 on my localhost.

```ini
REDIS_HOST=127.0.0.1
REDIS_PORT=8002
REDIS_PASSWORD=mypassword
```

The `REDIS_PASSWORD` setting is also used by the docker compose file that runs the Redis service.

## Digitalocean Redis environment settings

The following `.env` file environment settings are used to connect to a Digitalocean Managed Redis cluster:

```ini
REDIS_HOST=tls://<your_redis_host>.db.ondigitalocean.com
REDIS_PORT=25061
REDIS_PASSWORD=<your_redis_password>
```

> Note: the `tls:` scheme and TLS protocol port number is used since managed Redis only supports TLS connections

## Laravel session configuration to use Redis

The following is the Laravel session configured in the config/session.php configuration file to use redis

```php
    'driver' => env('SESSION_DRIVER', 'file'),
    //the `session` connection of the redis driver from config/database.php file
    'connection' => 'session',
    'files' => storage_path('framework/sessions'),
```

The `SESSION_DRIVER` setting in the  `.env` file explicitly specifies that the `redis` driver defined in config/database.php is used as the session driver.
If this setting is omitted, the session will work using the `file` driver saving sessions in the storage_path.

```ini
SESSION_DRIVER=redis
```

## Laravel cache configuration to use Redis

The following is the Laravel cache configured in the config/cache.php configuration file to use redis

```php
   #select the cache store from the `stores` setting below to be the default cache store
   #The name CACHE_DRIVER is a misnomer as it actually refers to the cache store
  'default' => env('CACHE_DRIVER', 'file'),

  'stores' => [
    'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],
    'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],
  ],

  # redis cache key prefix (used to segment between multiple apps)
  # set to empty string since I never share a redis server between apps
  'prefix' => '',
```

The `CACHE_DRIVER` setting in the `.env` file explicitly specifies the default cache store `redis` to use which in turn uses the `redis` driver defined in config/database.php. If this setting is omitted, the cache will work using the `file` store which uses the `file` driver saving sessions in the storage_path.

```ini
CACHE_DRIVER=redis
```

## Laravel queue configuration to use Redis

The following is the Laravel queue configured in the config/queue.php configuration file to use redis

```php
  #select the queue connection from the `connections` setting below to be the default cache store 
  'default' => env('QUEUE_CONNECTION', 'sync'),

  'connections' => [

        'sync' => [
            'driver' => 'sync',
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
],
```

The `QUEUE_CONNECTION` setting in the `.env` file explicitly specifies the default queue connection `job` to use which in turn uses the `redis` driver defined in config/database.php. If this setting is omitted, the queue will work using the `sync` connection which uses the `sync` driver.

```ini
QUEUE_CONNECTION=job
```

## Redis Cluster Queue key hash tags

> Note: TBD This may only apply when using client controlled clustering

From Laravel docs at `https://laravel.com/docs/7.x/queues`

If your Redis queue connection uses a Redis Cluster, your queue names must contain a key hash tag. This is required in order to ensure all of the Redis keys for a given queue are placed into the same hash slot:

'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],

The brackets `{default}` are required for redis to set the hash tag.

## Connecting with PHP to running redis container

```bash
php artisan tinker
Redis::connection()->ping();
```

## Connecting with TablePlus to running redis container

Open TablePlus and create a connection with the following:

click create a new connection
select redis
click create
type in my_app_name for the connection name

Enter the following credentials:

```ini
REDIS_HOST=127.0.0.1
REDIS_PORT=8002
REDIS_PASSWORD=mypassword
#REDIS_PASSWORD=null if we ommit the --requirepass flag for the redis command
```

## location of redis and mysql data and config files within container

```bash
#show redis.conf file location
redis-cli -p 8002 info | grep 'config_file'
# on macOS
# /usr/local/etc/redis.conf

#show file directory in redis.conf file
cat /usr/local/etc/redis.conf | grep "dir "
# on macOS
# /usr/local/var/db/redis/
```
