# Configure Laravel Session To Use Redis

April 13, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed for newer versions of Laravel 

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
    //selects a driver from the config/session.php file
    'driver' => env('SESSION_DRIVER', 'file'),
    //if redis is selected as the driver then selects the redis driver
    //connection from the config/session.php file
    'connection' => env('SESSION_CONNECTION', null),
    'files' => storage_path('framework/sessions'),
```

## Setting the Laravel session driver

We can change the default Laravel session store configuration to use the Redis key value store for its session by changing the 'driver' setting in the `config/session.php` configuration file by setting the `SESSION_DRIVER` variable in the .env  file to `redis`:

```ini
SESSION_DRIVER=redis
```

This changes the default value to use the `redis` driver defined in `config/database.php`.

## Setting the redis session connection

By default Laravel does not include the `SESSION_CONNECTION` setting in the .env file.

This means that if we set the  Laravel session driver to use redis we should also add the `SESSION_CONNECTION` setting to the .env file to explicitly set the redis driver connection that the redis driver should use.

The redis driver connections are specified in the `config/database.php` file.

If we do not explicitly set the `SESSION_CONNECTION` setting then the `default` redis connection from the `config/database.php` file will be used by default because the second parameter to `env('SESSION_CONNECTION', null)` helper is null.

I like to add a separate connection for the session in the `config/database.php` file and explicitly set the `SESSION_CONNECTION` to this connection.

To do so we first add a new connection named `session` to the available redis connections inside the `redis` driver section in `config/database.php`:

```bash
'redis' => [
    'session' => [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', '6379'),
                'database' => '0',
                'prefix' => 's:'
    ],
],
```

> Note that the `session` connection in `config/database.php` always uses a value of `0` for its `database` setting since redis clusters only support a single database per redis server. In order to differentiate between redis keys for session items vs other types of items, we use a key prefix specified by the additional `prefix` setting.

Now we can update the .env file to use this connection by adding the following `SESSION_CONNECTION` setting:

```ini
SESSION_CONNECTION=session
```

An alternative to adding the `SESSION_CONNECTION` setting would be just to hard code the connection in the `config/session.php` file:

```php
    'driver' => env('SESSION_DRIVER', 'file'),
    'connection' => 'session',
```
