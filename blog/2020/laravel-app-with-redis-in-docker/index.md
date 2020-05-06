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
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
#SESSION_DRIVER=redis
```

## Changing redis configuration settings for the redis service

Below is the redis driver configuration in `config/database.php`:

```php
  //the redis driver
  'redis' => [
        //the redis client (requires installing the phpredis.so extension using pecl)
        'client' => 'phpredis',

        'options' => [
            //this setting is only effective when using a managed redis cluster. No impact if redis cluster is not used.
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            //this setting is only effective when using predis client. No impact if predis client is not used.
            'prefix' => '',
        ],

        //connection used by the redis facade
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_DB', '0'),
            //redis key perfix for this connection
            'prefix' => 'D',
        ],
        //connection used by the cache facade when redis cache is configured in config/cache.php
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_CACHE_DB', '0'),
            //redis key perfix for this connection
            'prefix' => 'C',
        ],
        //connection used by the queue when redis cache is configured in config/queue.php
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_QUEUE_DB', '0'),
            //redis key perfix for this connection
            'prefix' => '{Q1}',
        ],
        //connection used by the session when redis cache is configured in config/session.php
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => env('REDIS_SESSION_DB', '0'),
            //redis key perfix for this connection
            'prefix' => 'S',
        ],
    ],
```

Below is the cache configuration in `config/cache.php`:

```php
  //use the 'redis' cache connection in this file
  'default' => 'redis',
  //this is the 'redis' cache connection
  'redis' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'cache' connection from the redis driver in config/database.php
            'connection' => 'cache',
        ],
  //this is the redis cache key prefix for applied to all cache keys
  //set to empty since key prefix is set at the redis connection level config/database.php
  'prefix' => '',
```

Below is the queue configuration in `config/queue.php`:

```php
  //use the 'redis' queue connection in this file
  'default' => 'redis',
  'connections' => [
        //this is the 'redis' queue connection
        'redis' => [
            //uses the redis driver from config/database.php
            'driver' => 'redis',
            //uses the 'queue' connection from the redis driver in config/database.php
            'connection' => 'queue',
            //this is the redis queue key prefix applied to all queue keys
            //set to empty since key prefix is set at the redis connection level config/database.php
            'queue' => '',
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
```

https://redis.io/topics/cluster-spec#keys-hash-tags

## Test

```php
use Illuminate\Support\Facades\Redis;

public function redisTest()
{
    $redis = Redis::connection();

    try{
        var_dump($redis->ping());
    } catch (Exception $e){
        $e->getMessage();
    }
}
```