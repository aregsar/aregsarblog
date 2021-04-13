# Redis Configure Laravel Cache

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

In this post I will detail the steps to take to configure the Laravel caching API to use a Redis server running in a local docker for caching data development.

Out of the box Laravel use file based caching by default.

## Steps to configure Laravel to use Redis for caching

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Install the Laravel Redis driver

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 2 - Setup the Redis sever docker service

Refer to step 1 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 3 - Set the connection settings for the docker Redis service

Refer to step 2 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 4 - Configure the Redis driver settings

Refer to step 4 in [Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel) to perform this step

### Step 5 - Renaming the Cache Driver setting in the .env file

Open the `.env` file and rename the `CACHE_DRIVER` setting to `CACHE_STORE` to accurately reflect that it is selecting one of the available cache stores from the `stores` array in `config/cache.php`.

Before:

```ini
#incorrectly named environment variable that selects the cache store
CACHE_DRIVER=file
```

After:

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=file
```

### Step 6 - Renaming the Cache Driver setting in the cache.php file

Open the `config/cache.php` file and rename the `CACHE_DRIVER` variable to `CACHE_STORE`.

The relevant snippet from `config/cache.php` file is shown below:

Before:

```php
    //Default Cache Store used by laravel caching funcions
    //Misnamed env var that selects the cache store (not cache driver)from the Cache Stores array below
    'default' => env('CACHE_DRIVER', 'file'),
    'stores' => [
        //this is the 'file' cache store selected by the 'default' setting above when CACHE_DRIVER=file in .env file or by the second argument when no key is specified
        'file' => [
                'driver' => 'file', //this is the actual cache driver which is not what is being selected
                'path' => storage_path('framework/cache/data'),
            ],
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

After:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the file cache store from the cache 'stores' array below using correctly named CACHE_STORE env setting
    'default' => env('CACHE_STORE', 'file'),
    'stores' => [
        //this is the 'file' cache store selected by the 'default' setting above when CACHE_STORE=file in .env file or by the second argument when no key is specified
        'file' => [
                'driver' => 'file', //the actual cache driver for the 'file' store
                'path' => storage_path('framework/cache/data'),
            ],
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

### Step 7 - Updating the CACHE_STORE setting to use the redis store

Update the `CACHE_STORE` setting in the .env file to `redis`.

Before:

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=file
```

After:

```ini
#correctly named environment variable that selects the cache store
CACHE_STORE=redis
```

The setting value is used by the `default` setting in `config/cache.php` file to select the `redis` store.

### Step 8 - Updating the default argument of the env helper

For the `default` setting in the `config/cache.php` file, update the default argument of the env() helper to `redis`.

Before:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the file cache store from the cache 'stores' array below using correctly named CACHE_STORE env setting
    'default' => env('CACHE_STORE', 'file'),
```

After:

```php
    //Default Cache Store used by laravel caching funcions
    //Selects the cache redis store from the Cache Stores array below using correctly named CACHE_STORE setting
    'default' => env('CACHE_STORE', 'redis'),
```

Now that we updated this default argument to `redis` the `default` setting will select the `redis` store even if the `CACHE_STORE` setting is not provided in the .env file.

At this point our `default` store setting selects the `redis` store that is using a redis driver connection named `cache` to connect to the Redis server when we use Laravels API.

The `cache` connection is configured in the redis driver settings in the config/database.php file which was configured in step 4.

How that all works will be described in the next to last section of this article.

### Step 9 - Use the cache service

Startup the docker service

```bash
docker-compose up -d
```

You should now be able to use caching with Redis in your Laravel application.

Start tinker:

```bash
php artisan tinker
```

Try the various caching methods of the the cache facade.

```php
//use default cache store from config/cache.php
Cache::put('key', 'value');
$value = Cache::get('key');
$hasKey = Cache::has('key');
Cache::forget('key');

//explicitly use the 'redis' store from config/cache.php
Cache::store('redis')->put('key', 'value');
$value = Cache::store('file')->get('foo');
$hasKey = Cache::store('redis')->has('key');
Cache::store('redis')->forget('key');
```

When done you can shutdown the docker service

```bash
docker-compose down
```

## Creating and using additional non default redis cache stores

The cache store configured by the `default` setting is used by the Laravel cache helper, Cache facade, CacheManager and Cache class methods by default without requiring and explicit cache store name to be passed in to the connection.

However we can create additional redis stores and explicitly pass them in to the cache methods to use different stores.

We can create additional redis stores in the stores array in `config/cache.php` file and configure them in the same way as we configured the default `redis` cache store.

We can then use these stores by passing them in explicitly by name to the Laravel cache helper, cache facade, CacheManager and Cache class methods.

Here is an example of configuring additional cache stores and cache connections:

In `config/database.php` add a `cache2` connection with a `c2` key prefix.

Both the original `cache` connection and the new `cache2` connection are shown below:

```php

'redis' => [

        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c:',
        ],

        'cache2' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => '0',
            //redis key prefix for this connection
            'prefix' => 'c2:',
        ],

    ],
```

> Note that both cache connections use the same Redis server host settings. We could for scalability add a different `REDIS_HOST_2` env setting to use a completely separate Redis server in production.

Next in `config/cache.php` add a `redis2` store that uses the new `cache2` connection.

Both the default `redis` store and the new `cache2` store are shown below:

```php
    'default' => env('CACHE_STORE', 'redis'),
    'stores' => [
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
        //additional redis store that we can explicitly pass for example to the laravel cache facade
         'redis2' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache2',//the new redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

With this setup we could explicitly pass the string `redis2` to the cache helper or facade methods to use the redis store that uses the `cache2` connection to access the Redis server.

## Looking at the configured settings in context

The `'default' => env('CACHE_STORE', 'redis')` setting was configured to select the `redis` store setting, in the `stores` array.

Within the `redis` store setting there is a driver and connection setting as shown below:

```php
    'stores' => [
        //the 'redis' cache store that is selected by the setting above when CACHE_STORE=redis in .env file
        'redis' => [
                'driver' => 'redis', //the actual cache driver for the 'redis' store
                'connection' => 'cache',//the redis connection setting as specified in the config/database.php file
                'lock_connection' => 'default',
            ],
    ]
```

The driver setting refers to the `redis` driver section in the `config/database.php` file.

The connection setting refers to the `cache` connection within the `redis` driver section in the `config/database.php` file.

For reference, the `redis` driver setting at the root settings level in `config/database.php` snippet along with its `cache` connection setting is shown below:

```php
'redis' => [
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            //database set to 0 since only database 0 is supported in redis cluster
            'database' => `0`,
            //instead add prefix setting to distinguish between connections
            //prefix not required if using one redis server/cluster host per connection
            'prefix' => 'c:'
        ],
    ],
```

Now from within your Laravel application, the `redis` store we configured in `configs/cache.php` is used by the Laravel `Cache::` facade, `cache()` helper or their underlying `CacheManager` and `Cache` classes by default without requiring the user to pass in a explicit cache store.

This is because the underlying CacheManager and Cache classes by default use the `default` store (from the config/cache.php file) which is the `redis` store (from the config/cache.php file) which is configured to use the `cache` redis driver connection (from config/database.php).

Internally the underlying `CacheManager` and `Cache` classes delegate the `RedisManager` and `Redis` classes to connect to Redis server. They pass the `cache` connection to the delegated Redis classes to override the `default` connection (from config/database.php) that is used by default to connect to the Redis server by the Redis classes.

## Key takeaways

The way the caches connect to your application in Laravel are through cache stores specified in the config/cache.php file.

Each cache store has an underlying driver.

If the driver for a cache store is configured as a `redis` driver then the cache store will have an underlying connection setting that refers to one of the available connections in the `redis` driver setting array within the config/database.php file.

The redis connections in the config/database.php file specify the connection parameters of the Redis server that the Laravel cache API methods will connect to.

This default connection can be overridden by passing in the name of another cache store, that is configured with a different connection name, to the Laravel cache API.
