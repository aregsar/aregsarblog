# Laravel App With Redis In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Creating the docker-compose file

> Note if you have already a docker-compose.yml then skip to the next section.

```bash
touch docker-compose.yml
echo `version: "3.1"' >> docker-compose.yml
echo `services:' >> docker-compose.yml
```

## Creating the data directory

> Note if you have already a the data directory then skip to the next section.

```bash
echo '/data' >> .gitignore
mkdir data
```

## Adding the mysql service to docker-compose

```yaml
  redis:
      image: redis:alpine
      container_name: myapp-redis
      command: redis-server --appendonly yes --requirepass myapp
      volumes:
      - ./data/redis:/data
      ports:
        - "8002:6379"
```

## Adding the environment variables for the redis service

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=myapp
REDIS_PORT=8002
QUEUE_PREFIX_VERSION=V1:
```

Note: We can remove the SESSION_DRIVER, CACHE_DRIVER, QUEUE_CONNECTION keys as their values are hard coded to redis in the session, cache and queue configuration files. 

## redis connection configuration

Below is the redis driver configuration in `config/database.php`:

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
            'prefix' => 'q:'.env('QUEUE_PREFIX_VERSION', 'V1:'),
        ],
    ],
```

We have separate connections for session, cache , queue and a default application specific redis connection.
They all use the same redis server with their own key prefix.

The the session, cache and queue configurations will use the corresponding redis connection specified here.

## redis session configuration

Below is the session configuration in `config/session.php`:

```php
//uses the redis driver from config/database.php
'driver' => env('SESSION_DRIVER', 'redis'),
 //uses the 'session' connection from the redis driver in config/database.php
'connection' => 'session',
```

Then add set the SESSION_DRIVER in .env file to redis

```ini
SESSION_DRIVER=redis
```

## redis cache configuration

Below is the cache configuration in `config/cache.php`:

```php
  //use the 'redis' cache store in this file
  'default' => env('CACHE_DRIVER', 'redis'),
  'stores' => [
    //this is the 'redis' cache connection
    'redis' => [
                //uses the redis driver from config/database.php
                'driver' => 'redis',
                //uses the 'cache' connection from the redis driver in config/database.php
                'connection' => 'cache',
            ],
   ],
  //this is the redis cache key prefix applied to all cache keys
  'prefix' => '',
```

Then add set the CACHE_DRIVER in .env file to redis

```ini
CACHE_DRIVER=redis
```

## redis queue configuration

Below is the queue configuration in `config/queue.php`:

```php
  //uses the 'job' queue connection in this file
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
    ],
```

Then add set the QUEUE_CONNECTION in .env file to job

```ini
QUEUE_CONNECTION=job
```

> Note that the queue names are wrapped in brackets. This ensures that when using a redis cluster, redis hashes the name
and uses the hash to put all keys with the same hash in the same bucket. See `https://redis.io/topics/cluster-spec#keys-hash-tags`
> Note: The job and app queues  use the same redis connection. The job connection is used when using the queued job dispatch and the queue facade without specifying a connection name

## Run the Docker services

```bash
docker-compose up -d
```

## Connecting redli CLI

```bash
#redli -h host -a password -p port
#redli -h 127.0.0.1 -a mypassword -p 8002
redli -p 8002 -a mypassword
127.0.0.1:8002> ping
# pong
127.0.0.1:8002> set rdl:test "abcd"
127.0.0.1:8002> get rdl:test
# abcd
127.0.0.1:8002> exit
# 127.0.0.1:8002> quit
```

## Connecting redis-cli CLI 

Testing redis with following commands:

```bash
redis-cli -p 8002 -a mypassword
127.0.0.1:8002> ping
# pong
127.0.0.1:8002> set rds:test "abcd"
127.0.0.1:8002> get rds:test
# abcd
127.0.0.1:8002> exit
# 127.0.0.1:8002> quit
```

## Connecting with TablePlus to running redis container

Open TablePlus and create a connection with the following:

click create a new connection
select redis
click create
type in my_app_name for the 
Enter the following credentials:

## Connecting with artisan

```bash
php artisan tinker
>>> Illuminate\Support\Facades\Redis::connection()->ping();
>>> Illuminate\Support\Facades\Redis::connection("default")->ping();
>>> Illuminate\Support\Facades\Redis::connection("session")->ping();
>>> Illuminate\Support\Facades\Redis::connection("cache")->ping();
>>> Illuminate\Support\Facades\Redis::connection("queue")->ping();
```

## Persisting data between docker container runs

```bash
docker-compose down
```

Try connecting to redis see failure

Bring the services back up.

```bash
docker-compose up -d
```

See the connections are working again

Check data/redis directory to see the persisted redis files

## Using the Redis facade

We can access Redis variables using the Redis facade:

```php
Illuminate\Support\Facades\Redis::connection()->ping();
Illuminate\Support\Facades\Redis::connection("default")->ping();
Illuminate\Support\Facades\Redis::connection("session")->ping();
Illuminate\Support\Facades\Redis::connection("cache")->ping();
Illuminate\Support\Facades\Redis::connection("queue")->ping();

Illuminate\Support\Facades\Redis::set('foo','bar');
$bar = Illuminate\Support\Facades\Redis::get('foo');
$has = Illuminate\Support\Facades\Redis::exists('foo');
Illuminate\Support\Facades\Redis::del('foo');
//expire in seconds
Illuminate\Support\Facades\Redis::expire('foo',60);
Illuminate\Support\Facades\Redis::expireat('foo', '1495469730');
Illuminate\Support\Facades\Redis::set('foo', 'bar', 'EX', 60);
Illuminate\Support\Facades\Redis::command('set', ['foo','bar']);
```

## Using the Session facade

We can access session variables using the Session facade:

```php
Illuminate\Support\Facades\Session::put('foo','bar');
$bar = Illuminate\Support\Facades\Session::get('foo');
$has = Illuminate\Support\Facades\Session::exists('users');
Illuminate\Support\Facades\Session::forget('foo');
$data = Illuminate\Support\Facades\Session::all();
```

Session variables are key value pairs that will be serialized to a single value and saved in Redis using a autogenerated session key, but only when used within the context of a Laravel web request.

Unlike the Redis, Cache and Queue facades. The Session facade when used outside the context of Laravel web request, only stores the session variables in a memory array.
So when used in Artisan Tinker shell or in phpunit tests, once the tinker session or unit test is finished, the in memory session array is cleared and the session data is never persisted to redis. 
This is the case for any other session stores that may be used as well.

When used within a Laravel web request, be it a route closure, controller action or middleware, at the end of the request the session variables are serialized and written to Redis using a auto generated redis key that the framework generates as the session key. This session key along with its value is then written to a session cookie which is used to get the session data back from redis using the session key on following requests.

So to see the actual session key and corresponding serialized values in Redis we need to use the session facade within a web request, at the end of which the session will be persisted to redis.

> Note: When a user logs in using laravel authentication, the auth framework writes a session variable named `web_login` which is serialized as part of the session and can be inspected after the session is serialized to redis.

When developing locally, you will only see a single session key value in redis as you switch between users. This is because Laravel sees the browser and all browser tabs as a single user based on the session cookie saved on the browser.
When you log out, the session cookie that contained the session key is cleared and the session key and value is cleared from redis. Then when you log in with another user, a new session key is generated and written to the session cookie.
So in order to simulate multiple users on the server each connected with their own browser, you need to open new instances of the browser in incognito mode. Once you do that you can see as many redis session key values as the number of open browser instances.

## Using the Cache facade

Cache variables will now be saved in Redis:

The default cache store is named `redis` and we can access it like so:

```php
Cache::put('foo','bar');
$bar = Cache::get('foo');
Cache::forget('foo');
Cache::put('tmp', 'bar', 60);
$has = Cache::has('tmp');
```

As we saw it is not necessary to explicitly reference the default store, however it may be useful to be able to do so if we define additional redis stores that use different redis connections

We can explicitly reference the `redis` cache store like so:

```php
Cache::store('redis')->put('foo','bar');
$bar = Cache::store('redis')->get('foo');
Cache::store('redis')->forget('foo');
```

## Using the Queue facade and Queue dispatch

Queue access methods use the configuration settings from `config.queue.php`:

First we need to create a Job class that can be queued for background processing. For testing we will just make the job log a message:

```php
class WelcomeEmailJob
{
    handle()
    {
        //todo: complete this
    }
}
```

Now we can queue this job to redis:

```php
//push on the default '{job}' queue of the default 'job' connection
Queue::push(new WelcomeEmailJob());

//push on the '{high}' queue of the default 'job' connection
//Note: the {high} queue setting of the default 'job' connection is set on the fly
Queue::pushOn('{high}',new WelcomeEmailJob());
```

We can explicitly specify the queue connection to override the default `job` connection:

```php
//push on the default '{app}' queue of the default 'app' connection
Queue::connection('app')->push(new WelcomeEmailJob());

//push on the '{high}' queue of the 'app' connection
//Note: the {high} queue setting of the selected 'app' connection is set on the fly
Queue::connection('app')->pushOn('{high}',new WelcomeEmailJob());
```

We can also use the Job dispatch method:

```php
//dipatch to the default '{job}' queue of the default 'job' connection
WelcomeEmailJob::dispatch();

//dipatch to the default '{app}' queue of the 'app' connection
WelcomeEmailJob::dispatch()->onConnection('app');

//dipatch to the '{high}' queue of the default 'job' connection
//Note: the {high} queue setting of the default 'job' connection is set on the fly
WelcomeEmailJob::dispatch()->onQueue('{high}');

//dipatch to the '{high}' queue of the 'app' connection
//Note: the {high} queue setting of the 'app' connection is set on the fly
WelcomeEmailJob::dispatch()->onConnection('app')->onQueue('{high}');
```

> Note: WelcomeEmailJob has a queue connection property that will override the default 'job' connection if set. By default `WelcomeEmailJob::dispatch()` will dispatch to the queue setting of the connection that is set to its connection property. If no connection is set to the property, then it will use the queue setting of default connection.

We can now inspect the serialized job class in redis.

## Processing queue items

We can run the artisan queue:work to process the jobs we queued to redis in the last section:

```bash
# process queue items from the default {job} queue of the default 'job' connection
php artisan queue:work
```

```bash
# process queue items from the explicit '{high}' queue of the default 'job' connection
php artisan queue:work --queue={high}
```

```bash
# process queue items from the default {app}  queue of the 'app' connection
php artisan queue:work -- app
```

```bash
# process queue items from the explicit '{high}' queue of the 'app' connection
php artisan queue:work --queue={high} -- app
```

## Adding Additional cache stores and queue connections

If we so desire, we can have additional cache and queue redis connection and stores that use different redis connections. In order to do that we can define additional redis connections in config/database.php.

For example below we have added the `cache2` and a `queue2` connections that connect to a different redis server.

```php
  //the redis driver
  'redis' => [

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

        //connection used by the queue when redis cache is configured in config/queue.php
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'q:'.env('QUEUE_PREFIX_VERSION', 'V1:'),
        ],

        //connection used by the cache facade when redis cache is configured in config/cache.php
        'cache2' => [
            'url' => env('REDIS_URL2'),
            'host' => env('REDIS_HOST2', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => '',
        ],
         //connection used by the queue when redis cache is configured in config/queue.php
        'queue2' => [
            'url' => env('REDIS_URL2'),
            'host' => env('REDIS_HOST2', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => env('QUEUE_PREFIX_VERSION', 'V1:'),
        ],
    ],
```

Then in config/cache.php we can add a `redis2` cache store that uses the `cache2` redis connection from config/database.php:

```php
  //use the 'redis' cache store in this file
  'default' => 'redis',
  'stores' => [
    //this is the 'redis' cache connection
    'redis' => [
                //uses the redis driver from config/database.php
                'driver' => 'redis',
                //uses the 'cache' connection from the redis driver in config/database.php
                'connection' => 'cache',
            ],
    //this is the 'redis2' cache connection
    'redis2' => [
                //uses the redis driver from config/database.php
                'driver' => 'redis',
                //uses the 'cache' connection from the redis driver in config/database.php
                'connection' => 'cache2',
            ],
   ],
```

Also in config/queue.php we can add a `job2` queue connection that uses the `queue2` redis connection from config/database.php:

```php
  //uses the 'job' queue connection in this file
  'default' => 'job',
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
        'job2' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'queue' connection from the redis driver in config/database.php
            'connection' => 'queue2',
            //this is the redis queue key default prefix that is applied when using this 'app' connection.It can be overriden by explicitly passing the queue name.
            'queue' => '{job}',
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
```

## Scaling out redis in production

The configuration used in this article uses a single redis server with database set to 0 so the same configuration can be used in production redis cluster.
Multiple redis connections are confgured in config/database.php each with a unique redis key prefix to differentiate between redis keys written by the session vs cache and queue facades. All the connections use the same host and port therbye using the same redis server.

This makes it easy to scale out these connections by assigning a different redis server for each connection.  
So sessions would have their own redis server and the default cache store will have its own redis server and the default queue will have its own redis server. 
We can even scale this out further by adding more cache stores and queue connections each with their own additional redis connection that uses a different redis server. 

Also In this scenario instead of each queue connection using other named queues in addition to their default queue, they could just use a separate queue connection with its own default queue instead of the named queue.

Queue workers can also we scaled by running them with the queue connection flag specifying the queue they need to service.
