# Redis Configure Laravel Queue

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

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

### Step 8 - Use the redis queue

## Anatomy of queue connections

I have annotated the configurations so you can see how the overall connection settings work together to establish a connection from the application to the Redis server used as the queue.

```ini
#set to default connection in config/queue.php
QUEUE_CONNECTION=redis
```

config/queue.php

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
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

In config/database.php

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

## How the default queue connection works

The Laravel queue api by default uses the `default` connection setting declared at the root level in config/queue.php to select one of the many connections declared in the `connections` array also declared at the root of the file.

Out of the box the `default` setting selects the out of the box `sync` connection, which tells the framwork to dispatch jobs synchronously without using a queue.

Howevere now we changed the `default` setting to select the `redis` connection that also comes out of the box. Queue connections must specify `redis` as the value of their `driver` setting to distinguish them as redis queue connetions. This value refers to the `redis` driver section in the config/database.php file where the redis server connections are specified.

The `redis` queue connection itself specifies a `connection` setting that should be set to a value that refers to one of the redis server connections of the redis driver in config/database.php file. In our case we set to the connection named `queue` that we added to the config/database.php file.

> If the connection descriptions become confusing, think of the connections in the connections array of queue.php as high level queue connections that the Laravel application can use. And think of the redis connections in the database.php file as low level connections to the Redis server that can be used by the high level queue connections by connecting to the low level connection using the queue connections internal connection setting.

## Additional non default queue connections

By default the Laravel queue methods use the default queue connection. However we can add more redis queue connections in queue.php file that can be explicitly passed by name to the Laravel queue methods to select a different queue to use to dispatch a job, instead of the default one.

Lets add a new queue connection:

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
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],

    ],
```

As you can see we added a `redis2` connection that has all the same settings of the `redis` connnection.

Now we can explicitly pass the name to the Laravel Queue facade like so: `Queue::connection('redis2')->dispatch($job);`.

Since both connections use the same settings,this will write to the same redis server using then same redis key prefix of `default`. Remember that the queue name `default` becomes the redis key prefix.

The Laravel methods will allow you to ovveride the queue name (the redis prefix) to write to different redis queues. But we can explicitlt set a different name to the `queue` setting of the `redis2` connection so we only need to specify the connection name:

We can change the name to sparate out the two queues:

```php
 'connections' => [

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

Now the connection will use a `default2` as the prefix.

## Using separate Redis connections

In the previous section we created a new queue connection in queue.php that uses the same redis connection from database.php.

If we wanted to scale out our queues so that each queue connection uses its individual redis connection all we need is to add a new redis commections and point one of the queue connections to this new redis connection

First lets add a queue2 connction to database.php.

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

Note that the queue2 connection also uses a REDIS_HOST_2 otherwise both connections would point to the same server which would defeat the purpose of scaling out.

The `q`: prefix allows us to distingush between keys on the same Redis server that may have been written by the redis cache or session, if the server is also used for those purposes.

Now we can refer to queue2 redis connection from the redis2 queue connection

```php
 'connections' => [
         'redis2' => [
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
            'queue' => 'default',
            'retry_after' => 90,
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
```

Note that I am using the queue name setting of 'queue' => 'default' for both queue connections. Since we are using two separate redis connections there will be no conflict between the queue names.

## Using the queue

Examples of using the queue with the Queue facade and with Job classes

1-Using the default queue connection 'redis' with its default queue 'default'
Queue::push(new InvoiceEmail($order));
PodcastJob::dispatch($podcast);

2-Overriding the the default queue connection's 'default' queue

//push on the 'email' queue instead of the default 'default' queue
Queue::pushOn('email', new InvoiceEmail($order));
PodcastJob::dispatch($podcast)->onQueue('email');

Note: the overriding queue name can be passed in on the fly. We don't need to have a queue named 'email' defined anywhere.

3-Overriding the default queue connection with the 'redis2' connection
//the connection in this context refers to the queue connection from config/queue.php (not the underlying redis connection in config/database.php)

//use the redis2 connection
$connection = Queue::connection('redis2')->push(new InvoiceEmail($order));
PodcastJob::dispatch($podcast)->onConnection('redis2');

4-Overriding the defaut queue connection with 'redis2' and overriding the default queue of the 'redis2' connection
$connection = Queue::connection('redis2')->pushOn('email', new InvoiceEmail($order));
PodcastJob::dispatch($podcast)->onConnection('redis2')->onQueue('podcasts');

Explicit overriding in Job class:
class WelcomeEmailJob implements App\Jobs\ShouldQueue
{
//The queue connection that should handle the job.
//this refers to the queue connection from config/queue.php (not the underlying redis connection in config/database.php)
public $connection = 'redis2';
}

php artisan queue:work --tries=3
