# Redis Configure Laravel Queue

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

In this post we will configure Laravel to use a Redis server running in a local docker container to queue jobs for background workers.

Out of the box the Laravel queue is configured to use synchronous mode which handles queued jobs inline as part of each web request.

## Steps to configure Laravel to use Redis for Queues

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Install the Laravel Redis driver

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 2 - Setup the Redis sever docker service

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 3 - Set the connection settings for the docker Redis service

Refer to step 2 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 4 - Configure the Redis driver settings

Refer to step 4 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 5 - adding a redis connection for the session

Based on the last step we know that there are two existing connections out of the box in the `config/database.php` file.

One is the `default` connection used by the Laravel Redis API by default and the other is the `cache` connection used by the Laravel Cache API by default (since the `default` cache store is configured to use the `redis` store that uses this `cache` connection).

While we can configure the queue to use one of these two existing redis driver connections from the `config/database.php` file, it is better to create a separate connection just for the queue.

This way if we need to scale out our application we can configure this connection to use its own separate Redis server.

Below I am showing the `config/database.php` file snippet showing the connections within the redis driver connections array before and after adding the connection for use by the queue.

Note that the before settings contain the configuration changes that were made in step 4.

Before:

```php
'redis' => [
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'd:',
        ],

        'cache' => [
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],

    ],
```

After:

```php
'redis' => [
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'd:',
        ],

        'cache' => [
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],
        //we added this connection specifically for use as the redis session connection
         'queue' => [
            'url' => env('REDIS_URL'),//This setting is not used
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:',
        ],

    ],
```

Note that I have added a third connection named `queue` that we will use for the session driver connection.

### Step 6 - Configure the default queue connection to use redis

```ini
QUEUE_CONNECTION=sync
```

After:

```ini
QUEUE_CONNECTION=redis
```

Now the QUEUE_CONNECTION will select the `redis` queue connection declared in the `connections` array in the config/queue.php file shown in the next step.

### Step 7 - Modify the configuration in config/queue.php

Configure the Laravel queue connection in `config/queue.php` to use the queue connection of the Redis server that we added to `config/database.php`.

We will also make a slight change by removing the unused REDIS_QUEUE variable used to and hard code the queue name to the fallback value.

Before:

```php
 'default' => env('QUEUE_CONNECTION', 'sync'),
 'connections' => [
        'sync' => [
            'driver' => 'sync',
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default',
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],

    ],
```

After:

```php
 'default' => env('QUEUE_CONNECTION', 'sync'),
 'connections' => [
        'sync' => [
            'driver' => 'sync',
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'queue',
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

## How the default queue connection works

The Laravel queue api by default uses the `default` connection setting declared at the root level in config/queue.php to select one of the many connections declared in the `connections` array also declared at the root of the file.

Out of the box the `default` setting selects the out of the box `sync` connection, which tells the framwork to dispatch jobs synchronously without using a queue.

Howevere now we changed the `default` setting to select the `redis` connection that also comes out of the box. Queue connections must specify `redis` as the value of their `driver` setting to distinguish them as redis queue connetions. This value refers to the `redis` driver section in the config/database.php file where the redis server connections are specified.

The `redis` queue connection itself specifies a `connection` setting that should be set to a value that refers to one of the redis server connections of the redis driver in config/database.php file. In our case we set to the connection named `queue` that we added to the config/database.php file.

> If the connection descriptions become confusing, think of the connections in the connections array of queue.php as high level queue connections that the Laravel application can use. And think of the redis connections in the database.php file as low level connections to the Redis server that can be used by the high level queue connections by connecting to the low level connection using the queue connections internal connection setting.

## Additional no default queue connections

By default the Laravel queue methods use the default queue connection. However we can add more redis queue connections in queue.php file that can be explicitly passed by name to the Laravel queue methods to select a different queue to use to dispatch a job, instead of the default one.
