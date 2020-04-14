# Configure Laravel Queue To Use Redis

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Laravel Queue To Use Redis](https://aregsar.com/blog/2020/configure-laravel-queue-to-use-redis)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel 

In this article I will show you how to convert the Laravel Queue configuration to use Redis to queue background jobs.

By default Laravel queue facade\provider is configured to use a `sync` driver for synchronously executing queued jobs. This will not be work in a production environment when jobs need to be queued to a data store to be picked up and executed in the background.

## The queue configuration

The file `config/queue.php` contains multiple queue connections specified in the `connections` setting. The file also contains a `default` connection setting for the queue that is set to the `sync` connection via the out of the box `QUEUE_CONNECTION` .env file setting and the default second parameter of the env() function.

The `sync` connection has a `driver` setting of `sync` indicating that the queue will operate synchronously, immediately executing any queued jobs.

There is also a `redis` connection. This connection has a `driver` setting that is set to the `redis` driver. This `redis` driver is defined in the `config/database.php` file.
The `redis` connection also has a `connection` setting set to `default`. This `default` connection is also defined in the `config/database.php` file as one of the connections of the `redis` driver.

From `config/queue.php` file:

```php
#selects default sync connection from connections array in this file
#by using the QUEUE_CONNECTION=sync in the .env file or by removing #QUEUE_CONNECTION=sync from the .env file
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
        ],
]
```

From `config/database.php` file:

```php
    'redis' => [
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

## changing the default queue connection to use Redis

We can change the default Laravel queue store configuration to use the Redis key value store for its queue by changing the `default` setting specified in the `config/queue.php` configuration file shown below:

```php
'default' => env('QUEUE_CONNECTION', 'sync'),
```

We want to change the `QUEUE_CONNECTION` in the `.env` file from `QUEUE_CONNECTION=sync` to `QUEUE_CONNECTION=redis`.

> Note: We dont want to hard code the `default` setting to `redis` and we want to keep the value of the default parameter passed to env() to `sync` so that we can remove the `QUEUE_CONNECTION` variable from the .env file if we need the queue to operate synchronously for development testing.



## Using the Laravel queue

#uses the default connection setting in config/queue.php which includes the queue name to be used
#the connection also specifies the driver and associated connection specified in config/database.php
Queue::push(new InvoiceEmail($order));

#names the queue explicitly but uses the default connection setting in config/queue.php
#overrides the queue name specified in the default connection setting
Queue::pushOn('emails', new InvoiceEmail($order));

#explicitely selecting a queue connection from config/queue.php connections
$connection = Queue::connection('connection_name')->push(new InvoiceEmail($order));

#Alternatively can get a queue instance from queue manager
$queueManager = app('queue');
$queue = $queueManager->connection('redis');
$queue->push(new InvoiceEmail($order));




From `config/queue.php` file:

```php
 #selects redis connection from connections array in this file
 #by using the QUEUE_CONNECTION=redis setting in .env file
 'default' => env('QUEUE_CONNECTION', 'sync'),

 'connections' => [
        'redis' => [
            #hardcodes 'redis' driver from config/database.php
            'driver' => 'redis',
            #hard coded 'default' connection from 'redis' driver connection in config/database.php
            'connection' => 'default',
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
]
```

From `config/database.php` file:

```php
    'redis' => [
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

    ],
```


