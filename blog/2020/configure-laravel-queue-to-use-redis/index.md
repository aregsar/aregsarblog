# Configure Laravel Queue To Use Redis

April 17, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel 

In this article I will show you how to convert the Laravel Queue configuration to use Redis to queue background jobs.

By default Laravel queue facade\provider is configured to use a `sync` driver for synchronously executing queued jobs. This will not be work in a production environment when jobs need to be queued to a data store to be picked up and executed in the background.

## The queue facade/provider configuration

The file `config/queue.php` contains multiple queue connections specified in the `connections` setting. The file also contains a `default` connection setting for the queue that is set to the `sync` connection via the out of the box `QUEUE_CONNECTION` .env file setting and the default second parameter of the env() function.

The `sync` connection has a `driver` setting of `sync` indicating that the queue will operate synchronously, immediately executing any queued jobs.

There is also a `redis` connection. This connection has a `driver` setting that is set to the `redis` driver. This `redis` driver is defined in the `config/database.php` file.
The `redis` connection also has a `connection` setting set to `default`. This `default` connection is also defined in the `config/database.php` file as one of the connections of the `redis` driver.

From `config/queue.php` file:

```php
// selects default sync connection from connections array in this file
// by using the QUEUE_CONNECTION=sync in the .env file or by removing QUEUE_CONNECTION=sync from the .env file

 'default' => env('QUEUE_CONNECTION', 'sync'),

 'connections' => [

        'sync' => [
            'driver' => 'sync',
        ],

        'redis' => [
            // hard coded 'redis' driver from config/database.php
            'driver' => 'redis',
            // hard coded 'default' connection from 'redis' driver connection in config/database.php
            'connection' => 'default',
            // queue name that is set to `default` since no REDIS_QUEUE setting is defined in .env file
            // this name will be used as a Redis key prefix so we can have different queues with the same Redis connection (no need to change this setting)
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
]
```

From `config/database.php` file:

```php
    'redis' => [
        #out of the box 'default' redis connection
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

## changing the default Laravel Queue connection to use Redis

We can change the default Laravel queue store configuration to use the Redis key value store for its queue by changing the `default` setting specified in the `config/queue.php` configuration file shown below:

```php
'default' => env('QUEUE_CONNECTION', 'sync'),
```

We want to change the `QUEUE_CONNECTION` in the `.env` file from `QUEUE_CONNECTION=sync` to `QUEUE_CONNECTION=redis` to set the `default` setting to use the `redis` connection defined in `config/queue.php`.

> Note: We don't want to hard code the `default` setting to `redis` and we want to keep the value of the default parameter passed to env() to `sync` so that we will be able to remove the `QUEUE_CONNECTION` variable from the .env file if we need the queue to operate synchronously for development testing.

## changing the default redis connection to use a different cache store

The `redis` connection in `config/queue.php` file has a  `'connection' => 'default'` setting that selects the `default` connection of the `redis` driver defined in `config/database.php`.

We can change this so that the queue connection in `config/queue.php` can use a separate redis driver connection that we can add to `config/database.php`.

Below we have shown the new configuration that adds a new `redis` driver connection named `queue` to `config/database.php` and reference this new connection from the `connection` setting of the `redis` queue connection in `config/queue.php`.

I have also added a whole new queue connection named `redis2` which references a `queue2` redis driver connection.

The annotations in the configuration code snippets below should serve to clarify what is described above.

From `config/queue.php` file:

```php
// config/queue.php
 'connections' => [
        'redis' => [
            // hard coded 'redis' driver specified in config/database.php
            'driver' => 'redis',
            // hard coded newly added 'queue' connection from 'redis' driver connection in config/database.php
            'connection' => 'queue',
            // this specifies the default key prefix for this 'redis' connecton that is used for the queue keys
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],'redis2' => [
            // hard coded 'redis' driver specified in config/database.php
            'driver' => 'redis',
            // hard coded newly added 'queue' connection from 'redis' driver connection in config/database.php
            'connection' => 'queue2',
            // this specifies the default key prefix for this 'redis2' connecton that is used for the queue keys
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ]
]
```

From `config/database.php` file:

```php
// config/database.php
    'redis' => [
        // newly added `queue` connection (uses same connection setttings as the default connection)
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            // uses a separate database for the queue
            'database' => env('REDIS_CACHE_DB', '2'),
        ],
    ],
```

> Note that the `queue` connection in `config/database.php` uses a value of `2` for its `database` setting to indicate a different Redis database. Otherwise there would be no point for adding this new connection

## Using the Laravel default queue connection

The following uses the default connection setting in `config/queue.php`:

```php
Queue::push(new InvoiceEmail($order));
```

## Overriding the default queue connection

We can explicitly selecting a queue connection specified in `config/queue.php` by the queue connection name that we like to use.

This will override the default queue connection set to the `default` setting.

```php
$connection = Queue::connection('redis2')->push(new InvoiceEmail($order));
```

Alternatively can use a queue instance from queue manager:

```php
$queueManager = app('queue');
$queueManager->connection('redis')->push(new InvoiceEmail($order));
```

Both above use the default queue for the connection they are using, which is specified by the `queue` setting of each connection in `config/queue.php`.

## Overriding the connection queue

We can override the default queue of the queue connection that is used by the facade by explicitly passing in a queue name.

The code below names the queue explicitly but uses the default connection setting in config/queue.php. 

```php
Queue::pushOn('email', new InvoiceEmail($order));
```

The code below names the queue explicitly and  also overrides the default queue connection setting in config/queue.php.

```php
$connection = Queue::connection('redis2')->pushOn('email', new InvoiceEmail($order));
```

## Using connection queue prefix key for Redis Cluster queue

When using a Redis cluster the `queue` value used as the prefix key must be wrapped
in `{}` brackets as shown below:

```php
// The redis connection
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
```

As we can see the default queue prefix key is set to `'{default}'` instead of `'default'`.

## Dispatching queued jobs with different queue configurations

```php
Job::dispatch();
```

```php
Job::dispatch()->onQueue('emails');
```

```php
PodcastJob::dispatch($podcast)->onConnection('queue');
```

```php
PodcastJob::dispatch($podcast)->onConnection('queue')->onQueue('podcasts');
```

Instead of calling onConnection we can configure the job class to specify the queue connection:

```php
class ProcessPodcast implements App\Jobs\ShouldQueue
{
    //The queue connection that should handle the job.
    public $connection = 'queue';
}
```

We can process the job via the queue:work command

```bash
php artisan queue:work --tries=3
```
