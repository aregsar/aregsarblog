# Configure Laravel Queue To Use Redis

April 17, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed for newer versions of Laravel

This post is part of a series of posts listed below that show how to setup your Laravel project to use Redis:

[Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis)

[Configure Laravel Session To Use Redis](https://aregsar.com/blog/2020/configure-laravel-session-to-use-redis)

[Configure Laravel Cache To Use Redis](https://aregsar.com/blog/2020/configure-laravel-cache-to-use-redis)

[Configure Laravel Queue To Use Redis](https://aregsar.com/blog/2020/configure-laravel-queue-to-use-redis)

[My Laravel Redis Configuration](https://aregsar.com/blog/2020/my-laravel-redis-configuration)

[Create Laravel Project With Multiple Redis Stores](https://aregsar.com/blog/2020/create-laravel-project-with-multiple-redis-stores)

In this article I will show you how to change the out of the box Laravel Queue configuration that uses the sync driver to instead use Redis as the queue driver for background jobs.

By default Laravel queue facade\provider is configured to use a `sync` driver for synchronously executing queued jobs.

This will not work in a production environment when jobs need to be queued to a data store to be picked up and executed in the background.

Also we may want to run feature tests against the same setup as we have in production.

## The queue facade/provider configuration

Below are annotated snippets of the out of the box Laravel queue configuration and database configuration files:

The Laravel cache configuration file `config/queue.php` has a list of connections as defined by the `connections` setting:

```php
// selects default sync connection from connections array in this file by using the QUEUE_CONNECTION=sync in the .env file or by removing QUEUE_CONNECTION=sync from the .env file
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
            // this name will be used as a Redis key prefix so we can have different queues with the same Redis connection
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
]
```

The `redis` store has a `driver` setting in `config/queue.php` which corresponds to the `redis` driver in the `config/database.php` file. The `redis` store also has a `connection` setting in `config/queue.php` which corresponds to the `default` connection of the `redis` driver in the `config/database.php` file.

From `config/database.php` file:

```php
    'redis' => [
        //out of the box 'default' redis connection
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

    ],
```

## changing the default Laravel Queue connection to use Redis

We can change the default Laravel queue store configuration to use the Redis key value store for its queue by changing the `default` setting specified in the `config/queue.php` configuration file shown below by setting the `QUEUE_CONNECTION` variable in the .env  file to `redis`:

```ini
QUEUE_CONNECTION=redis
```

## changing the default redis connection to use a different cache store

The `redis` connection in `config/queue.php` file has a `'connection' => 'default'` setting that selects the `default` connection of the `redis` driver defined in `config/database.php`.

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
            'database' => '0',
            'prefix' => 'q:'
        ],
    ],
```

> Note that the `queue` connection in `config/database.php` always uses a value of `0` for its `database` setting since redis clusters only support a single database per redis server. In order to differentiate between redis keys for queue items vs other types of items, we use a key prefix specified by the additional `prefix` setting.

## Using the Laravel default queue connection

The following uses the default connection setting in `config/queue.php`:

```php
Queue::push(new InvoiceEmail($order));
```

## Overriding the default queue connection

We can explicitly selecting a queue connection specified in `config/queue.php` by the queue connection name that we like to use.

This will override the default queue connection set to the `default` setting in `config/queue.php`.

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
