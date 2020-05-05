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

`config/database.php`:

```php
  'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'myapp'), '_').'_database_'),
        ],

        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
            'prefix' => 'D:',
        ],

        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '0'),
            'prefix' => 'C:',
        ],
        'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_QUEUE_DB', '0'),
            'prefix' => 'Q1:',
        ],
        'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_SESSION_DB', '0'),
            'prefix' => 'S:',
        ],
    ],
```

`config/cache.php`:

```php
  'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],
  'default' => env('CACHE_DRIVER', 'redis'),
  'prefix' => '',
```

`config/queue.php`:

```php
  'default' => env('QUEUE_CONNECTION', 'redis'),
  'connections' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'queue',
            'queue' => '',
            //'queue' => '{default}',
            //'queue' => env('REDIS_QUEUE', 'default'),
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