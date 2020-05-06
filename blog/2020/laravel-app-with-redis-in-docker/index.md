# Laravel App With Redis In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Creating files and directories

> Note if you have already a docker-compose.yml then these steps are not required

echo '/data' >> .gitignore
mkdir data
touch docker-compose.yml
echo `version: "3.1"' >> docker-compose.yml
echo `services:' >> docker-compose.yml

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
            'database' => env('REDIS_DB', '0'),
            //redis key prefix for this connection
            'prefix' => 'd1:',
        ],
        //connection used by the cache facade when redis cache is configured in config/cache.php
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_CACHE_DB', '0'),
            //redis key prefix for this connection
            'prefix' => 'c1:',
        ],
        //connection used by the queue when redis cache is configured in config/queue.php
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_QUEUE_DB', '0'),
            //redis key prefix for this connection
            'prefix' => 'v1:',
        ],
        //connection used by the session when redis cache is configured in config/session.php
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_SESSION_DB', '0'),
            //redis key prefix for this connection
            'prefix' => 's1:',
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
'driver' => 'redis',
 //uses the 'session' connection from the redis driver in config/database.php
'connection' => 'session',
```

## redis cache configuration

Below is the cache configuration in `config/cache.php`:

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
   ],
  //this is the redis cache key prefix for applied to all cache keys
  'prefix' => 'def',
```

## redis queue configuration

Below is the queue configuration in `config/queue.php`:

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
            //this is the redis queue key default prefix that is applied when using the 'job' connection. It can be overriden by explicitly passing the queue name.
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
            //this is the redis queue key default prefix that is applied when using the 'app' connection.It can be overriden by explicitly passing the queue name.
            'queue' => '{app}',
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
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
redli -h 127.0.0.1 -a 123456 -p 8002
127.0.0.1:8002> ping
# pong
127.0.0.1:8002> set tst:test "abcd"
127.0.0.1:8002> get tst:test
# abcd
127.0.0.1:8002> exit
# 127.0.0.1:8002> quit
```

## Connecting redis-cli CLI 

Testing redis with following commands:

```bash
redis-cli -p 8002
127.0.0.1:8002> ping
# pong
127.0.0.1:8002> set tst:test "abcd"
127.0.0.1:8002> get tst:test
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

## Test redis connections from Laravel application

```php
use Illuminate\Support\Facades\Redis;

public function redisTest()
{
    try{
        var_dump(Redis::connection()->ping());
        var_dump(Redis::connection("default")->ping());
        var_dump(Redis::connection("session")->ping());
        var_dump(Redis::connection("cache")->ping());
        var_dump(Redis::connection("queue")->ping());
    } catch (Exception $e){
        var_dump($e->getMessage());
    }
}
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


## cache tests

```php
$value = Cache::get('foo');

$value = Cache::store('redis')->get('foo');
```

## queue tests

We can connect using the default connection and explicit connection specified in `config/queue.php` using the Queue::push command.
We can also override the default `queue` name setting for each connection using the Queue::pushOn command.

```php
//push on the default 'job' queue of the default 'job' connection
Queue::push(new WelcomeEmail());
Queue::connection('job')->push(new WelcomeEmail());

//push on a high priority 'high' queue using the default 'job' connection
Queue::pushOn('high',new WelcomeEmail());
Queue::connection('job')->pushOn('high',new WelcomeEmail());

//push on the default 'app' queue of the 'app' connection
Queue::connection('app')->push(new WelcomeEmail());

//push on a high priority 'high' queue using the 'app' connection
Queue::connection('app')->pushOn('high',new WelcomeEmail());

//dipatch to the default 'job' queue of the default 'job' connection
//WelcomeEmailJob has a queue connection property that will override the default 'job' connection if set
WelcomeEmailJob::dispatch();

//dipatch to a high priority 'high' queue of the default 'job' connection
//WelcomeEmailJob has a queue connection property that will override the default 'job' connection if set
WelcomeEmailJob::dispatch()->onQueue('high');
```

https://laravel-news.com/laravel-jobs-queues-101
https://divinglaravel.com/pushing-jobs-to-queue
php artisan queue:work --queue=emails

First Laravel checks if a connection property is defined in your job class, using the property you can set which connection Laravel should send the queued job to, if no property was defined null will be used and in such case Laravel will use the default connection.



