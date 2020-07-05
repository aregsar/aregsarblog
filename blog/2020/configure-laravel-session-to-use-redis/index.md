# Configure Laravel Session To Use Redis

April 13, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel 

This post is part of a series of posts listed below that show how to setup your Laravel project to use Redis:

[Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis)

[Configure Laravel Session To Use Redis](https://aregsar.com/blog/2020/configure-laravel-session-to-use-redis)

[Configure Laravel Cache To Use Redis](https://aregsar.com/blog/2020/configure-laravel-cache-to-use-redis)

[Configure Laravel Queue To Use Redis](https://aregsar.com/blog/2020/configure-laravel-queue-to-use-redis)

[My Laravel Redis Configuration](https://aregsar.com/blog/2020/my-laravel-redis-configuration)

[Create Laravel Project With Multiple Redis Stores](https://aregsar.com/blog/2020/create-laravel-project-with-multiple-redis-stores)

In this article I will show you how to convert the Laravel Session configuration to use Redis.

## The session configuration

By default Laravel uses the file driver to store session data on the server. This will not be performant for high traffic sites.

Here is the default setting in config/session.php:

```php
    'driver' => env('SESSION_DRIVER', 'file'),
```

## Setting the Laravel session driver

We can change the default Laravel session store configuration to use the Redis key value store for its session by changing the 'driver' setting in the `config/session.php` configuration file by setting the `SESSION_DRIVER` variable in the .env  file to `redis`:

```ini
SESSION_DRIVER=redis
```

This changes the default value to use the `'redis'` driver defined in `config/database.php`.

Note: We want to keep the value of the default parameter passed to env() helper to `file` so that we will be able to remove the `SESSION_DRIVER` variable from the .env file if we need the session to use files for development testing.

```php
    'driver' => env('SESSION_DRIVER', 'file'),
```

## The Redis connection used by the Laravel Session

The `config/session.php` file also has a `'connection'` setting. 

This setting selects the Redis driver `connection` from the `config/database.php` that the `redis` session driver should use.

```php
    // uses the 'cache' configuration of the 'redis' driver in config/database.php
    // add SESSION_CONNECTION=default or SESSION_CONNECTION=cache to .env
    // don't know if null parameter defaults to 'default' connection if SESSION_CONNECTION is not specified
    'connection' => env('SESSION_CONNECTION', null),
```

By default the `SESSION_CONNECTION` environment variable setting is not defined in `.env` file. Also the default parameter value for the `'connection'` setting in `config/session.php` is null. So by default `'connection'` is set to null causing the `'default'` connection of the `'redis'` driver in `config/database.php` to be selected as the session connection by the Laravel session provider\facade.

## Explicitly configuring the session redis connection

Instead of allowing the session connection to use the default redis connection, I like to be more explicit.

The following steps set the session connection to a completely separate redis connection added to `config/database.php`.

First add a new connection named `session` to the available redis connections inside the `redis` driver section in `config/database.php`:

```bash
'redis' => [
    'session' => [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_CACHE_DB', '3'),
    ],
],
```

The new connection is named `session` and its `database` setting is set to `3` to use a different redis database then the redis database used by default connection since they both use the same redis server settings.

Next remove default the `SESSION_CONNECTION=file` setting from the `.env` file because it will not be used.

Finally in `config/session.php` I changed the default connection parameter from `null` to `'session'` :

```php
    'connection' => env('SESSION_CONNECTION', 'session' ),
```

This sets the default value to the new `session` connection added to `config/database.php`.

Alternatively in `config/session.php` you can hard code the session driver `connection` to `session`:

```php
    'connection' => 'session',
```

This sets the value to directly to the new `session` connection added to `config/database.php`.
