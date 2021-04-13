# Redis Configure Laravel Queue

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

In this post we will configure Laravel to use a Redis server running in a local docker container to queue jobs for background workers.

Out of the box the Laravel queue is configured to use synchronous mode which handles queued jobs inline as part of each web request.

## Steps to configure Laravel to use Redis for Queues

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Install the Laravel Redis driver

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 2 - Setup the Redis sever docker service

Refer to step 2 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 3 - Set the connection settings for the docker Redis service

Refer to step 3 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 4 - Configure the Redis driver settings

Refer to step 4 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 5 - Add a redis driver connection

Based on the last step we know that there are two existing connections out of the box in the `config/database.php` file.

One is the `default` connection used by the Laravel Redis API by default. The other is the `cache` connection used by the Laravel Cache API by default (since the `default` cache store is configured to use the `redis` store that uses this `cache` connection).

While we can configure the Laravel Queue APIto use one of these two existing redis driver connections from the `config/database.php` file, it is better to create a separate redis driver connection just for the queue.

This way if we need to scale out our application we can configure this connection to use its own separate Redis server.

Below I am showing the `config/database.php` file snippet that shows the connections within the `redis` driver connections array, before and after adding the new redis connection named `queue`.

Note that the before settings already contain the configuration changes that were made in step 4.

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
        //we added this connection specifically for use as the redis queue connection
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

Note that I have added a third redis driver connection named `queue` that will be the connection used by the Laravel queue connection.

Don't confuse the redis driver connection we added in the `config/database.php` file with the queue connection in the `config/queue.php` file that will use the redis driver connection.

The following steps detail the queue connection configuration.

### Step 6 - Configure the default queue connection

```ini
QUEUE_CONNECTION=sync
```

After:

```ini
QUEUE_CONNECTION=redis
```

Configured this way, the `QUEUE_CONNECTION` will select the `redis` queue connection declared in the `connections` array in the `config/queue.php` file shown in the next step.

### Step 7 - Modify the queue configuration

Configure the Laravel queue connection in `config/queue.php` to use the redis driver `queue` connection that we added to `config/database.php` in step 5.

Before:

```php
 'default' => env('QUEUE_CONNECTION', 'sync'),
 'connections' => [
        'sync' => [
            'driver' => 'sync',
        ],
        'redis' => [
            'driver' => 'redis',
            //uses the outof the box 'default' redis driver connection from config/database.php
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
 //this is the default queue connection that refers to the 'redis' queue connection in the `connections` array below
 //becuase we configured QUEUE_CONNECTION=redis
 'default' => env('QUEUE_CONNECTION', 'sync'),
 'connections' => [
        'sync' => [
            'driver' => 'sync',
        ],
        //this is the out of the box queue connection named redis
        'redis' => [
            'driver' => 'redis',
            //This is the redis driver connection we added to config/database.php
            'connection' => 'queue',
            //This queue connection also has a default implicit queue name that is named 'default'.
            //It can be overriden by explicitly passing a queue name using the Laravel queue API.
            //the value is appended as a key prefix to the key for the item placed in the queue
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

## Anatomy of the Laravel Queue configuration

I have annotated all the configurations we made in the previous sections so you can see how the overall connection settings work together to establish a connection from the application calls to the Redis server used as the queue.

In .env file we have the default queue connection setting that will be used in config/queue.php:

```ini
#set to default connection in config/queue.php
QUEUE_CONNECTION=redis
```

In config/queue.php we have the default queue connection and other available queue connections.

The queue connections that use the 'driver' => 'redis' setting have a default connection setting that refers one of the redis connections in config/database.php.

```php
//set 'redis' from QUEUE_CONNECTION in .env
//so selects the 'redis' connection from  'connections' array below
 'default' => env('QUEUE_CONNECTION', 'sync'),
 'connections' => [
        'sync' => [
            'driver' => 'sync',
        ],
        //the 'redis' connection selected by the 'default' setting above
        'redis' => [
            //the driver is set to 'redis' that refers to the 'redis' driver in config/database.php
            'driver' => 'redis',
            //connection refers to the 'queue' connection of the 'redis' driver in config/database.php
            'connection' => 'queue',
            //name of the queue we are accessing. Set to 'default' to represent a default queue
            //the value is used as the redis key prefix to distinquish between queues for all queue connections that use the same redis connection from config/database.php
            //the value can be any queue name we want since it is just a prefix that indicates a queue namespace
            //and does not refer to anything. For example we could name it: 'queue'=>'emails' for an email queue.
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

In config/database.php we have the redis connections that are referred to by queue connections that have a 'driver' => 'redis' setting in config/queue.php.

```php

//the redis driver that is referenced by the 'driver'=>'redis'setting of queue connections in config/queue.php.
'redis' => [
    //the redis connection named 'queue' that is referenced by the queue connection named 'redis' in the config/queue.php file
    'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:',
        ],
    ],
```

> If the connection descriptions become confusing, think of the connections in the config/queue.php file as high level queue connections that the Laravel application can use and think of the redis connections in the config/database.php file as low level connections to the Redis server that can be used by the high level queue connections.

## Additional non default queue connections

By default the Laravel queue methods use the default queue connection and by default this connection uses the default queue name 'default' that it is configured with.

We can explicity override the default queue connection and the default queue of any queue connection by explicitly passing values for then to the Laravel API methods. We will see how to do this in the next section.

We can override queue names on the fly by passing in a different queue name then the default queue name. However in order to override the default queue connection we need to declare additional named queue connections that we can use the name of to override the default connection.

So lets add more redis queue connections in queue.php file that can be explicitly passed by name to the Laravel queue methods to select a different queue to use to dispatch a job, instead of the default one.

Lets add a new `redis2` queue connection:

```php
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
         'redis2' => [
            'driver' => 'redis',
            'connection' => 'queue',
            'queue' => 'default2',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

Both queue connections use the same underlying redis connection `queue`. So we use a different `default2` queue name to distingush the redis keys used to store the queue items.

This is alright but a better approach is to make each queue connection use a different underlying redis connection.
So lets start over and add a the new `redis2` queue connection with a different underlying redis connection:

```php
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
         'redis2' => [
            'driver' => 'redis',
            'connection' => 'queue2',
            'queue' => 'default2',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

This time the `redis2` queue connection is using an undelying `queue2` redis connection that we need to add to config/database.php.
Also note that both queue connections still use different values for their underlying queue name setting, just in case the undelying redis connections are both configured to use the same Redis server host and port.

Now we need to add the `queue2` redis connection (that is referenced from the `redis2` queue connection above) to the config/database.php file:

```php
 'redis' => [
    'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:',
        ],
    'queue2' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST_2', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:',
        ],
    ],
```

As you can see we added the `queue2` redis connection that uses the `REDIS_HOST_2` environment variable so we can make the connection connect to a separate Redis server if we want.

In the next section I will show how to connect to the default queue and explicit queues by passing the additional connections to the Laravel Queue and Job APIs.

## Using the queued jobs in your Laravel application

Examples of using the queue with the Queue facade and with Job classes

Start Tinker:

```bash
php artisan tinker
```

1-Using the default queue connection `redis` with its default queue `default`

```php
Queue::push(new EmailInvoiceJob(new Invoice()));
EmailInvoiceJob::dispatch(new Invoice());
```

2-Overriding the `redis` default queue connection's `default` queue using a queue named `emails`

```php
//push on the 'email' queue instead of the default 'default' queue
Queue::pushOn('emails', new EmailInvoiceJob(new Invoice()));
EmailInvoiceJob::dispatch(new Invoice())->onQueue('emails');
```

Note: the overriding queue name can be passed in on the fly. We don't need to have a queue named 'email' defined anywhere.

3-Overriding the default queue connection with the `redis2` queue connection
//the connection in this context refers to the queue connection from config/queue.php (not the underlying redis connection in config/database.php)

```php
//use the redis2 connection
Queue::connection('redis2')->push(new EmailInvoiceJob(new Invoice()));
EmailInvoiceJob::dispatch(new Invoice())->onConnection('redis2');
```

4-Overriding the default queue connection with `redis2` queue connection and also overriding the default queue of the `redis2` connection at the same time with the `emails` queue name.

```php
Queue::connection('redis2')->pushOn('emails', new EmailInvoiceJob(new Invoice()));
EmailInvoiceJob::dispatch(new Invoice())->onConnection('redis2')->onQueue('emails');
```

## Setting the queue connection and queue name to use in Job classes

Instead of calling onConnection we can configure the job class to specify a non default queue connection from config/queue.php:

```php
//derive a job class
class EmailInvoiceJob implements App\Jobs\ShouldQueue
{
    //Set the queue connection to use
    public $connection = 'redis2';

    public __constructor($invoice)
    {
        //set a non default queue name if desired
        onQueue('emails')
    }
}
```

Now use the job as if it is queued to default connection and queue

```php
//push using overriden values
Queue::push(new EmailInvoiceJob(new Invoice()));
EmailInvoiceJob::dispatch(new Invoice());
```

## Running workers to process queued jobs

We can use the artisan queue:work command to run a single daemon process called a worker to retrieve job items from the queue and process them by calling the handle method of the job.

```bash
php artisan queue:work
```

Each time we call this command a new daemon process is run to process queue items. So calling it once will run a single worker and calling it twice will run to workers.

By default the worker will process use the default queue connection and use the default queue name of that connection.

Here is an example of running a worker process that uses the non default redis2 connection and also overrides the `default2` queue name with the queue name `emails`

```bash
php artisan queue:work --connection=redis2 --queueName=emails
```

We can tell each worker we run, how many times it can retry processing a queued job if the job fails:

```bash
php artisan queue:work --tries=3
```

In development we can use queue:listen instead of queue:work.

```bash
php artisan queue:listen
```

When we use queue:listen, the worker process recycles the code after each queue item it processes, so that any code changes we make are picked up the next time it processes an item.

The queue:work run worker on the underhand remains in memory for efficiancy in production environments so we would need to reload it after every code change using the command below:

```bash
php artisan queue:reload
```

In production we can launch multiple instances of a worker by using a program called supervidored. This program has a configiration file where you can set the number of worker processes and the queue:work command.
