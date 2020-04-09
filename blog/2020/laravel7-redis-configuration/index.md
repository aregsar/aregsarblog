# Laravel7 Redis Configuration

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Laravel7 Redis Configuration](https://aregsar.com/blog/2020/laravel7-redis-configuration)

> prerequisite: must have PHP and PECL installed. MacOS Hombrew PHP installation installs PECL as well. For Ubuntu you need to install PECL separately using apt package manager


## Default Redis driver in Laravel 7

As of Laravel 7 the framework comes preconfigured to use the phpredis driver instead of the old predis driver.

The Laravel docs suggest that future versions of the framework may not support predis because the package does not seem to be maintained any longer.

If you still prefer to use the old predis driver you can do so by first installing the predis package via composer:
`composer install predis/predis`

Then changing the default laravel setting in config/redis.php
to 'client' => 'predis' from 'client' => 'phpredis'

One advantage of using the default 'phpredis' client is that is a native php extension which performs much better under heavy loads.

To use the default 'phpredis' client you need to perform the additional step of installing the 'phpredis' php extension. The steps to do so for MacOS and Ubuntu are detailed below

## Install php-redis extension on MacOS

pecl install --force redis

pecl list
redis     5.2.1   stable

php -m
redis

/usr/local/lib/php/pecl/20190902/redis.so

/usr/local/etc/php/7.4/php.ini
extension="redis.so"

## Install php-redis extension on Ubuntu

> Installation instructions for php-redis extension can be found at `https://github.com/phpredis/phpredis/blob/develop/INSTALL.markdown`

## The Redis database driver configuraton

The redis configuration is in the `config/database.php` file

This file has driver configurations for all types of data stores used within a Laravel application.
The redis driver configuration is specified by the `'redis'` key in the file:

```php
'redis' => [

        #driver uses phpredis client as default
        # no need to set REDIS_CLIENT=phpredis in .env to specify phpredis explicitly
        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        # default connection
        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

        #cache connection
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),#uses different database then 'default' connection
        ],

    ],
```

> Note: the way drivers are specified in the config/database.php are inconsistent in naming
and structure. For instance the Redis driver is named 'redis' but the SQL database driver is named 'connections' instead of being named 'database'.

config/database.php has a list of drivers e.g. 'redis' or 'connections'

Some drivers like the 'redis' driver can have a 'client' setting `'client' => env('REDIS_CLIENT', 'phpredis'),` that represents the php client that it  uses to connect. By default the client is set to 'phpredis' to use the phpredis php extension to connect to redis. As mentioned above you can set it to or 'predis' to use the predis composer package to connect to Redis.

Each driver can also have one or more connections that represent the
connection settings for the particular type of database, for instance a relational database or a Redis database.

The 'default' and 'cache' are two connections for the 'redis' driver that are included by default.

You can add as many connections with different connection settings as you need.

The 'default' connection is used by the Laravel Redis facade by default. The Laravel Redis facade can explicitly use any of the redis connections by explicity referencing the connection name.

Examples of using both the default connection and named connections will be shown in the following sections.

## Avoiding namespace conflicts when using the default phpredis client

The only down side of sticking with the default 'phpredis' client is that you can not use the Laravel 'redis' facade alias within your application. 

This is because the 'phpredis' extension has a class named 'Redis' that conflicts with the 'Redis' facade class name. So in order to use the 'Redis` facade in our application code we must either fully qualify the class name with the class namespace or import the namespace into the file where we use the facade class as demonstrated below:

```php
# fully namespace qualified class name
Illuminate\Support\Facades\Redis::connection()->ping();
```

or

```php
# imported namespace
use Illuminate\Support\Facades\Redis;
Redis::connection()->ping();
```

Alternatively you can rename the 'Redis' facade alias in `config/app.php`

from:

```php
aliases' => [
    'Redis' => Illuminate\Support\Facades\Redis::class,
]
```

To

```php
aliases' => [
    'ZRedis' => Illuminate\Support\Facades\Redis::class,
]
```

Here I renamed the 'Redis' alias to 'ZRedis'

So now you can use it without requiring a namespace:

```php
ZRedis::connection()->ping();
```

## Connecting to Redis using default and explicit connections

If no connection is specified then the facade uses the `default` connection configuration.
```php
#uses the `default` connection from config/database.php
Illuminate\Support\Facades\Redis::set('name', 'Taylor');
```

```php
#uses `default` connection from config/database.php
Illuminate\Support\Facades\Redis::connection()->set('name', 'Taylor');
```

```php
#uses the explicit `cache` connection from config/database.php
Illuminate\Support\Facades\Redis::connection('cache')->set('name', 'Taylor');
```

## Configuring Laravel to use a managed Redis cluster

While you could add your own redis cluster configuration in the Clusters section. I like using a managed cluster. In order to use a managed cluster you only need to add line `'cluster' => env('REDIS_CLUSTER', 'redis')` to the 'options' array in the 'redis' driver in config/database.php:

'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],

The line `'cluster' => env('REDIS_CLUSTER', 'redis')`
tells the framework to use the default clustering capability of your managed Redis cluster. 

> Note: the default value of 'redis' for the second argument is used because the .env file does not contain a REDIS_CLUSTER setting by default. If you define this is variable in .env you must set its value to 'redis' to tell the framework to use the managed cluster.

Instructions to setup laravel for connecting with the digital ocean managed Redis cluster can be found at:
`https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster`
