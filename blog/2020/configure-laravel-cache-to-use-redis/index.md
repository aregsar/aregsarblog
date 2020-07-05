# Configure Laravel Cache To Use Redis

April 14, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed for newer versions of Laravel

This post is part of a series of posts listed below that show how to setup your Laravel project to use Redis:

[Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis)

[Configure Laravel Session To Use Redis](https://aregsar.com/blog/2020/configure-laravel-session-to-use-redis)

[Configure Laravel Cache To Use Redis](https://aregsar.com/blog/2020/configure-laravel-cache-to-use-redis)

[Configure Laravel Queue To Use Redis](https://aregsar.com/blog/2020/configure-laravel-queue-to-use-redis)

[My Laravel Redis Configuration](https://aregsar.com/blog/2020/my-laravel-redis-configuration)

[Create Laravel Project With Multiple Redis Stores](https://aregsar.com/blog/2020/create-laravel-project-with-multiple-redis-stores)

In this article I will show you how to convert the Laravel Cache configuration to use Redis.

By default Laravel uses the file driver to cache data. This will not be performant for high traffic sites.

## The Laravel Cache facade/provider configuration

Below are annotated snippets of the out of the box Laravel cache configuration and database configuration files:

The Laravel cache configuration file `config/cache.php` has a list of stores as defined by the `stores` setting:

```php
 // config/cache.php
 'default' => env('CACHE_DRIVER', 'file'),
 'stores' => [
        // This is the redis store driver
        'redis' => [
            // uses the redis driver defined in config/database.php
            'driver' => 'redis',
            // uses the 'cache' named connection of the 'redis' driver from config/database.php
            'connection' => 'cache',
        ],
        'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],
    ],
```

The configuration file also has default cache store setting defined by the `default` setting in the `config/cache.php` file.

> Note: the `CACHE_DRIVER` environment variable used to set the cache `default` is a misnomer because it indicates that it is the setting for a cache `driver` where in actuality it is setting the cache to a `store` that is one of the stores specified in the `stores` setting. The actual cache driver will be the driver specified within the selected store.

The `redis` store has a `driver` setting in `config/cache.php` which corresponds to the `redis` driver in the `config/database.php` file. The `redis` store also has a `connection` setting in `config/cache.php` which corresponds to the `cache` connection of the `redis` driver in the `config/database.php` file.

```php
// config/database.php
// This is the 'redis' driver
'redis' => [
        // This is the redis driver connection named 'cache'
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => '0',
            'prefix'=>'c:'
        ],
    ],
```

> Note that the `cache` connection in `config/database.php` always uses a value of `0` for its `database` setting since redis clusters only support a single database per redis server. In order to differentiate between redis keys for queue items vs other types of items, we use a key prefix specified by the additional `prefix` setting.

## Changing the default Laravel facade/provider cache driver to use Redis

To let the Laravel cache facade\provider use Redis by default we need to change the value of the `default` setting in `config/cache.php` to use the `redis` cache store show below by setting the `CACHE_DRIVER` variable in the .env  file to `redis`:

```ini
CACHE_DRIVER=redis
```

## Using the default Laravel cache store

The snippet below uses the default cache store specified in `config/cache.php`:

```php
$value = Cache::get('foo');
```

## Using an explicit cache store for Laravel Cache

We can explicitly select a store from the stores specified in `config/cache.php` to override the default cache configuration when using the cache provider or facade.

The snippet below explicitly uses a the `file` store specified in `config/cache.php`:

```php
$value = Cache::store('file')->get('bar');
```

## Adding and using additional cache stores

We can add additional `redis` cache stores in `config/cache.php` that are configured to use other `redis` driver connections in `config/database.php`.

Below is an example of adding a `redis2` store to `config/cache.php` that references a `cache2` redis connection added to the `redis` driver in `config/database.php`:

In config/database.php we have added the 'cache2' redis driver:

```php
// config/database.php
'redis' => [
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],
        'cache2' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

    ],
```

And in config/cache.php we have added a `redis2` store that refers to this new connection: 

```php
 // config/cache.php
 'stores' => [
        'redis' => [
            // uses the redis driver defined in config/database.php
            'driver' => 'redis',
            // uses the 'cache' configuration of the 'redis' driver in config/database.php
            'connection' => 'cache',
        ],
        'redis2' => [
            // uses the redis driver defined in config/database.php
            'driver' => 'redis',
            // uses the 'cache' configuration of the 'redis' driver in config/database.php
            'connection' => 'cache2',
        ]
    ],
```

Given the above settings we can explicitly use the new `redis2` store like so:

```php
$value = Cache::store('redis2')->get('bar');
```

## Changing the default redis store

We could also change the `default` setting in `config/cache.php` to use a different cache store instead of the out of the box configured `redis` store.

For instance we can use the `redis2` cache store we added to `config/database.php` above and set it to the `default` setting in `config/cache.php`:

```php
 'default' => 'redis2',
```

Now the following code will use `redis2` store as the default store:

```php
$value = Cache::get('foo');
```
