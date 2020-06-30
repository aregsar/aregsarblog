# Laravel App With Redis In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this blog post I will show you how to run a local Redis data store in a docker container.

We will configure the Redis docker container to persist data across container restarts.

We will also configure our Laravel application to use this Redis store for application data storage, for Laravel Session storage, for the Laravel Cache store and as a queue for Laravel queued jobs and queued notifications.

## Creating the docker-compose file

> Note: skip to the next section if you already have a `docker-compose.yml` file in the root directory

In this section we will create a new `docker-compose.yml` file in the Laravel project root directory:

```bash
touch docker-compose.yml
echo 'version: "3.1"' >> docker-compose.yml
echo 'services:' >> docker-compose.yml
```

## Creating the data directory

> Note: skip to the next section if you already have a data directory in the root directory

To store the Redis data on our local host machine we will create a new data directory in the Laravel project root directory:

```bash
echo '/data' >> .gitignore
mkdir data
```

## Adding the Redis service to docker-compose

To run the Redis container we will add the following docker compose service to the `docker-compose.yml` file.

```yaml
  redis:
      image: redis:alpine
      container_name: myapp-redis
      command: redis-server --appendonly yes --requirepass "${REDIS_PASSWORD}"
      volumes:
      - ./data/redis:/data
      ports:
        - "8002:6379"
```

Note that the redis port 6379 is mapped to port 8002 on our local host machine. 
Our Laravel application running on our host machine will be configured to connect to this mapped port.

Also note the `REDIS_PASSWORD` setting in the redis startup command. This setting will be defined in the applications `.env` file. Since the `docker-compose.yml` file is in the same directory as the `.env` file, it will make use of this setting.

Finally note that the `--appendonly yes` in the redis startup command tells redis data to persist its data in the `/data` directory in the container. This directory is part of the default redis configuration.

The docker compose volumes mapping maps the `/data` directory where redis stores its data to be mapped to the `./data/redis` directory on the host machine, allowing the data to remain across container restarts.

The `/data` directory is automatically created when redis persists its data for the first time.

## Redis connections configuration

In this section I will show you the application configuration required to connect to redis.

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
            'prefix' => 'q:',
        ],
    ],
```

We have separate connections for Laravels `session`, `cache` , `queue` facades and a `default` connection for our application code to use via the Laravel `Redis` facade.

The `database` key for each connection is always set to `0` to be compatible with running in a Redis cluster in production where only one database is allowed.

> Note: As configured all the connections use the same redis server with their own key prefix. However each can be configured to connect to a separate `REDIS_HOST` which can be useful for scaling production systems. For example we could use a separate environment variable such as `REDIS_CACHE_HOST` instead of `REDIS_HOST` for the `cache` connection to allow the `cache` connection to connect to a separate redis server. In this scenario we could set the `prefix` to use an empty string.

Once the redis connections are configured in `config/database.php` then Laravels Session, Cache and Queue facade configurations specified in the `session.php`, `cache.php` and `queue.php` files, will use their corresponding `session`,`cache` and `queue` redis connections from the `config/database.php` file to connect to the redis server.

## Adding the environment variables used by the redis connections

As a final step for the redis connection configuration we need to add the environment variables used by the redis connections in the `config/database.php` file to the `.env` file.

```ini
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=myapp
REDIS_PORT=8002
```

Note that the `REDIS_PASSWORD` is also used in the `docker-compose.yml` file to set the password for the redis server running in docker.

Also note that the `REDIS_PORT=8002` is the same port number that the redis port was mapped to in the `docker-compose.yml` file.

Since we are running in our local environment the `REDIS_HOST` is set to localhost.

## Redis session configuration

In this section we configure Laravel to use the `redis` driver to store session information in Redis instead of the default file based session.

We also configure the Laravel session to use the `session` Redis connection configuration from the `config/database.php` file.

Below is the session configuration in `config/session.php`:

```php
//uses the redis driver from config/database.php
'driver' => env('SESSION_DRIVER', 'redis'),
 //uses the 'session' connection from the redis driver in config/database.php
'connection' => 'session',
```

We also need to set the `SESSION_DRIVER` in `.env` file to `redis`.

```ini
SESSION_DRIVER=redis
```

## redis cache configuration

In this section we configure Laravel to use the `redis` driver to cache data in Redis.

We also configure the Laravel session to use the `cache` Redis connection configuration from the `config/database.php` file.

Note that unlike the Laravel session that only has one session store, the Laravel cache can have multiple cache stores so we have a `default` setting that has a value that selects the store to be used as the default store.
In the configuration below we have only specified a single cache store named `redis` so the `default` store value is set to that store. The store itself specifies what driver and what connection to use from the `config/database.php` file.

Below is the cache configuration in `config/cache.php`:

```php
  //use the 'redis' cache store in this file
  'default' => env('CACHE_DRIVER', 'redis'),
  'stores' => [
    //this is the 'redis' cache store
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

> Note: By adding more connections in the `config/database.php` file pointing to separate redis servers, we could add additional cache stores in the `config/cache.php` configuration and set the store `connection` to the new connections from the `config/database.php` file. This will be useful for scaling the app in production.

We also need to set the CACHE_DRIVER in `.env` file to `redis`.

```ini
CACHE_DRIVER=redis
```

## redis queue configuration

In this section we configure Laravel to use the `redis` driver to queue jobs and notifications using Redis.

We also configure the Laravel session to use the `queue` Redis connection configuration from the `config/database.php` file.

Note that unlike the Laravel session that only has one session store, the Laravel queue can have multiple queue stores (which are actually labeled `connections`). So we have a `default` setting that has a value that selects the store/connection to be used as the default store/connection.

In the configuration below we have specified two queue store/connections named `job` and `app`. We have set the `default` store\connection value to the `job` store/connection.
The store/connection itself specifies what `driver` and what `connection` to use from the `config/database.php` file.
Additionally the store/connection specifies a default `queue` name to be used in addition to the `connection`.
For instance the `job` store/connection specifies the `{job}` as the default queue name to be used when queueing jobs. Similarly the the `app` store/connection specifies the `{app}` as the queue name as its default queue.
The queue name is used as a prefix to the redis key used to store the queued item. This prefix is distinct from the queue prefix specified in the `config/database.php` file and will be prepended to that prefix.

Below is the queue configuration in `config/queue.php`:

```php
  //uses the 'job' queue connection in this file
  'default' => env('QUEUE_CONNECTION', 'job'),
  //this is a misnomer, it should actually be named `stores` as it is in the cache config in `config/cache.php`
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

We also need to set the `QUEUE_CONNECTION` in `.env` file to `job` to select the default store/connection as the `job` store/connection.

```ini
QUEUE_CONNECTION=job
```

> The `job` and `app` store/connections use the same `redis` connection from the `config/database.php` file thus using the same redis server. However we could use separate redis servers for each if we define additional redis connections in the `config/database.php` file and use those as separate redis connections for each cache store/connection. This will be useful when scaling your application in production.

>The store/connection set to the `default` is used by default by the queue facade and job dispatch classes without having to be explicitly named in our code. The non default store/connections need to be explicitly specified.

> Note that the queue names are wrapped in brackets. This ensures that when using a redis cluster, redis hashes the name
and uses the hash to put all keys with the same hash in the same bucket. See `https://redis.io/topics/cluster-spec#keys-hash-tags`

## Run the Docker services

To run the redis docker container from the root directory of the application run:

```bash
docker-compose up -d
```

## Connecting to the redis server using the redli CLI

We can connect to the redis server using the redli redis command line client.
Once connected we can execute the ping command to see if the server responds.

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

## Connecting to the redis server using redis-cli CLI 

We can connect to the redis server using the standard redis command line client.
Once connected we can execute the ping command to see if the server responds.

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

## Connecting with artisan

We can connect to the redis server using the Laravel artisan commands.

Here we are implicitly connecting to redis using the `default` connection by not specifying the connection. We are also connecting by specifying specific connections.

```bash
php artisan tinker
>>> Illuminate\Support\Facades\Redis::connection()->ping();
>>> Illuminate\Support\Facades\Redis::connection("default")->ping();
>>> Illuminate\Support\Facades\Redis::connection("session")->ping();
>>> Illuminate\Support\Facades\Redis::connection("cache")->ping();
>>> Illuminate\Support\Facades\Redis::connection("queue")->ping();
```

> Note: Even though we are using different connection names, as configured all the connections are using the same redis server instance.

## Connecting with TablePlus to running redis container

We can also connect to the redis server using the TablePlus data store management UI.

Open TablePlus and create a connection with the following:

click create a new connection
select redis
click create
type in my_app_name for the application
Enter the redis credentials and host and port number.
Test the connection and finally connect if the test passes.

## Persisting data between docker container runs

We can stop the redis server by stoping its container using the following docker compose command:

```bash
docker-compose down
```

Now we can try connecting to redis and should see a failure since the server is down.

We can bring the docker services back up again.

```bash
docker-compose up -d
```

The connections to redis should be working again.

We can also check the `./data/redis` directory that was created by the redis server, to see the persisted redis files.

If we stored data in the redis server before we shut the server down, we should see that the data still persists.

The following sections show how redis is accessed from within a Laravel application.

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

We can access session variables stored in redis using the Session facade:

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

Queue access methods use the configuration settings from `config/queue.php`:

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

> Note: WelcomeEmailJob has a queue connection property that will override the default `job` connection if set. By default `WelcomeEmailJob::dispatch()` will dispatch to the queue setting of the connection that is set to its connection property. If no connection is set to the property, then it will use the queue setting of default connection.

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

If we so desire, we can have additional cache and queue redis store/connections that use different redis connections from `config/database.php`.

In order to do that we can define additional redis connections in `config/database.php` then configure cache and queue store/connections in `config/cache.php` and `config/queue.php` to use those redis connections.

For example below we have added the `cache2` and a `queue2` connections in `config/database.php` that connect to a different redis server.

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
            'prefix' => env('QUEUE_PREFIX_VERSION', ''),
        ],
    ],
```

Then in `config/cache.php` we can add a `redis2` cache store that uses the `cache2` redis connection from `config/database.php`:

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

Also in `config/queue.php` we can add a `job2` queue connection that uses the `queue2` redis connection from `config/database.php`:

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

As we have seen Laravels Redis configuration structure allows us considerable flexibility in how we cab scale our redis usage.
