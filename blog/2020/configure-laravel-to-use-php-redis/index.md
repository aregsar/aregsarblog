# Configure Laravel To Use Php Redis

April 10, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed for newer versions of Laravel

This post is part of a series of posts listed below that show how to setup your Laravel project to use Redis:

[Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis)

[Configure Laravel Session To Use Redis](https://aregsar.com/blog/2020/configure-laravel-session-to-use-redis)

[Configure Laravel Cache To Use Redis](https://aregsar.com/blog/2020/configure-laravel-cache-to-use-redis)

[Configure Laravel Queue To Use Redis](https://aregsar.com/blog/2020/configure-laravel-queue-to-use-redis)

[My Laravel Redis Configuration](https://aregsar.com/blog/2020/my-laravel-redis-configuration)

[Create Laravel Project With Multiple Redis Stores](https://aregsar.com/blog/2020/create-laravel-project-with-multiple-redis-stores)

In this article I will show you how to setup the PHP REDIS driver so that your Laravel Project will be able to connect to Redis servers using the officially recommended configuration.

## Prerequisites

You must have PHP and PECL installed. When you install PHP on MacOS via Homebrew, PECL will be included.

On Ubuntu you need to install PECL separately using apt package manager. You can find PECL installation instructions for Ubuntu at the end of the article.

> You might find the redis-cli cli docs at `https://redis.io/topics/rediscli` useful for working with Redis. Alternatively Medis `http://getmedis.com/` app and `https://tableplus.com/blog/2018/06/best-redis-gui-client-tableplus.html` are nice GUI redis clients for working with redis data.

## Default Redis driver in Laravel 7

As of Laravel 7 the framework comes preconfigured to use the `phpredis` driver instead of the old `predis` driver.

> Note: `phpredis` is a PHP extension installed using PECL whereas `predis` is a Composer package installed using Composer.

The Laravel docs suggest that future versions of the framework may not support `predis` because the package does not seem to be maintained any longer.

If you still prefer to use the old `predis` driver you can do so by first installing the predis package via composer:
`composer install predis/predis`

Then change the default Laravel setting defined in `config/redis.php` from `'client' => 'phpredis'` to `'client' => 'predis'`.

One advantage of using the preconfigured `phpredis` client is that it is a native php extension which performs much better under heavy loads.

To use the preconfigured `phpredis` client you need to perform the additional step of installing the `phpredis` php extension using PECL.

The steps to do so for MacOS and Ubuntu are detailed below

## Install Php Redis extension on MacOS

```bash
pecl install --force redis
```

> Note: The --force flag makes sure the extension is added regardless of any cached settings that may prevent installation.

List installed PHP extensions using PECL to verify installation:

```bash
pecl list
```

You should see the following extension listed:

```bash
redis     5.2.1   stable
```

Even though the `redis` extension is listed it might still not be detected by PHP.

To verify that the extension is detected by our PHP installation, we can list the PHP extensions using the PHP CLI:

```bash
php -m
```

Make sure `redis` is included in the output list of PHP installed extensions.

Also check that the extension file `redis.so` exists at the PECL extensions directory for your installation. On my Mac I checked that `/usr/local/lib/php/pecl/20190902/redis.so` exists.

Finally check that the `php.ini` file for your php installation
includes the line `extension="redis.so"`. On my Mac I checked that the `/usr/local/etc/php/7.4/php.ini` file contained the line `extension="redis.so"`.

If for some reason the line `extension="redis.so"` does not exists, make sure to add it manually to `php.ini` file so that PECL and PHP detect the extension.

## Install Php Redis extension on Ubuntu

> Installation instructions for php-redis extension on ubuntu can be found at [install-php-redis-extension-on-ubuntu](https://github.com/phpredis/phpredis/blob/develop/INSTALL.markdown) and [how-to-setup-laravel-with-digitalocean-managed-redis-cluster](https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster). The steps are detailed at the end of the article.

## The Redis database driver configuration

The Laravel redis configuration is specified in the `config/database.php` file.

The `config/database.php` file has a list of drivers.

This list has driver configurations for all types of data stores used within a Laravel application. For instance it contains a driver named `redis` for Redis data stores and a driver named `connections` for SQL databases.

The out of the box Redis driver configuration is shown below:

```php
'redis' => [
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

        # cache connection
        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),#uses different database then 'default' connection
        ],
    ],
```

The driver has a `client` setting `'client' => env('REDIS_CLIENT', 'phpredis')` that represents the php client that it uses to connect to the Redis server.

By default the client is set to `phpredis` so it will use the `phpredis` php extension to connect to Redis.

I generally hard code the client configuration to `phpredis` since it does not change often:

```php
'client' => 'phpredis',
```

> As mentioned at the beginning of the article you can set the client to `predis` instead `phpredis` so it will use the `predis` composer package to connect to the Redis server instead.

## Avoiding namespace conflicts when using the default phpredis client

There is one downside of using the default `'phpredis'` Redis client. 

The downside is that you will not be able to use the default Laravel `Redis` facade `alias` specified in the `config/app.php` file within your application, unless it is renamed.

This is because the `phpredis` extension has a class named `Redis` that conflicts with the `Redis` facade class name.

If we choose not to rename the alias when using the `Redis` facade in our application code, we must either fully qualify the class name with the class namespace or import the namespace.

Both of these cases are demonstrated below:

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

## Renaming the Redis facade alias

As mentioned in previous section you can rename the `Redis` facade alias in `config/app.php` file so that it will not conflict with the same class name defined by the `phpredis` extension:

The out of the box alias name `Redis` specified in `config/app.php` is shown below:

```php
aliases' => [
    'Redis' => Illuminate\Support\Facades\Redis::class,
]
```

As an example I have changed the alias name to `ZRedis` below:

```php
aliases' => [
    'ZRedis' => Illuminate\Support\Facades\Redis::class,
]
```

So now you can use the `Redis` facade without importing its namespace:

```php
\ZRedis::connection()->ping();
```

## The Redis configuration connections

The out of the box redis configuration  specifies two Redis connections named `default` and `cache`.

The `default` redis connection is used by the Laravel `Redis` facade by default.

The Laravel Redis facade can use any of the other named connections such as the `cache` connection by explicitly referencing the connection by the connection name.

You can add as many connections with different connection settings as you need to the redis driver as long as each has a unique name.

Both the `default` and `cache` connections use the same connection settings except for the `database` setting. So they are connecting to the same redis server instance, but each uses a different database on that instance.

> Note: Since Redis clusters do not support more than one database for the cluster, we will change this configuration to always use database number 0 and use an additional setting labeled `prefix` to segment redis keys based on usage type.

Examples of using both the default connection and named connections are shown in the following sections.

## Connecting to Redis using default and explicit connections

When using the redis facade, if no connection is explicitly specified, then the facade uses the `default` connection configuration of the `redis` driver.

Below are examples of using the default and named connections:

Here the default connection from `config/database.php` is used:

```php
Illuminate\Support\Facades\Redis::set('name', 'Taylor');
```

Here also the default connection is used:

```php
Illuminate\Support\Facades\Redis::connection()->set('name', 'Taylor');
```

But here the `cache` connection from `config/database.php` is used:

```php
Illuminate\Support\Facades\Redis::connection('cache')->set('name', 'Taylor');
```

## Configuring Laravel to use a managed Redis cluster

In order to use a managed cluster you only need to add the line `'cluster' => env('REDIS_CLUSTER', 'redis')` to the 'options' array in the `'redis'` driver in `config/database.php` shown below:

```php
'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],
```

> Note: This is included in the out of the box Laravel installation and has no impact if you are not using a managed cluster.

The line `'cluster' => env('REDIS_CLUSTER', 'redis')` tells the framework to use the default clustering capability of a managed Redis server cluster.

> Note: the default value of `redis` for the second argument is used because the default `.env` file does not contain a `REDIS_CLUSTER` environment setting. If you define this variable in the `.env` file, you must set its value to `redis` to tell the framework to use a managed cluster.

Since I am always using the DigitalOcean Redis cluster, I just hard code this value in `config/database.php` as shown:

```php
'options' => [
        'cluster' => 'redis',
    ],
```

## Installing the phpredis driver on a DigitalOcean Ubuntu VPS

Instructions to setup Laravel for connecting with the DigitalOcean managed Redis cluster can be found at [how-to-setup-laravel-with-digitalocean-managed-redis-cluster](https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster).

I have outlined the steps from the article to install the PECL `phpredis` extension on Ubuntu below:

The code below assumes PHP is already installed.

Install the `phpredis` extension:

```bash
sudo apt install php-pear
sudo apt install php-dev
sudo pecl install redis
```

Enable the new PHP extension by adding the following line to the php.ini:

```ini
extension = redis.io
```

Restart PHP-FPM:

```bash
sudo systemctl restart php7.4-fpm.service
```

> Note: I am using PHP version 7.4, adjust version as needed. As long is it is version 7.2 or above, these instructions should work

## Configuration settings to use the DigitalOcean Redis cluster

Open the Laravel project `.env` file and add the Redis Cluster Credentials:

```ini
REDIS_HOST=tls://your_redis_host.db.ondigitalocean.com
REDIS_PASSWORD=your_redis_password
REDIS_PORT=25061
```

> Note: We need use the `tls://` part before the name of the Redis cluster

We can test the connections by adding a controller and a route to connect to the Redis cluster from our application.

First add a controller named `RedisController` to the project.

```bash
php artisan make:controller RedisController
```

Next add the following route to the `routes\web.php` file

```php
Route::get('/redis', 'RedisController@redisTest');
```

Then add the following `redisTest` method to the controller:

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

Visit the `/redis` path to see the result.

## Setting per connection key prefix when using the PhpRedis driver

According to Laravel 7 docs the `PhpRedis` driver also supports the following additional connection parameters: `prefix`, `persistent`, `read_timeout` and `timeout`

Below I have added the `prefix` setting to the `cache` and `default` connections while changing both connections to use the same `database` setting:

```bash
'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => `0`,
            'prefix' => 'd:'
        ],
'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => `0`,
            'prefix' => 'c:'
        ],
```

With this configuration Laravel will add the `d:` prefix to the redis key value when the application connects to redis using the `default` connection and will add the `c:` prefix when connecting using the `cache` connection.

## Setting the key prefix if using the predis driver

the per connection key prefix setting mentioned in the previous setting will not work if we change the out of the box redis driver to use the old Composer based `predis` driver.

When using the old `predis` driver we can only set a common prefix for all connections.

The common prefix setting is part of the `options` setting as shown below:

```bash
'options' => [
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
```

This configuration exists for the out of the box installation but is not being used since the out of the box configuration used the `phpredis` driver.

To avoid confusion with the per connection prefix setting, I generally remove the common `prefix` setting from the `options` setting when using the `phpredis` driver.

## The redis connections settings

Each Redis connection in the config file will have its own set of Redis server host and port settings. All the connections can be configured to connect to the same Redis server or they can individually be configured to connect to separate Redis servers.
