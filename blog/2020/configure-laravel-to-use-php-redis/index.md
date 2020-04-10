# Configure Laravel To Use Php Redis

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Laravel To Use Php Redis](https://aregsar.com/blog/2020/configure-laravel-to-use-php-redis)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel
> Prerequisite: must have PHP and PECL installed. The MacOS Hombrew PHP installation installs PECL as well. On Ubuntu you need to install PECL separately using apt package manager. You can find Ubuntu install instructions and link to instructions for connecting to the DigitalOcean redis cluster at the end of the article.

## Default Redis driver in Laravel 7

As of Laravel 7 the framework comes preconfigured to use the phpredis driver instead of the old predis driver.

The Laravel docs suggest that future versions of the framework may not support predis because the package does not seem to be maintained any longer.

If you still prefer to use the old predis driver you can do so by first installing the predis package via composer:
`composer install predis/predis`

Then changing the default Laravel setting defined in `config/redis.php` from `'client' => 'phpredis'` to `'client' => 'predis'`

One advantage of using the default `phpredis` client is that is a native php extension which performs much better under heavy loads.

To use the default `phpredis` client you need to perform the additional step of installing the `phpredis` php extension. The steps to do so for MacOS and Ubuntu are detailed below

## Install Php Redis extension on MacOS

```bash
pecl install --force redis
```

> Note: The --force flag makes sure the extension is added regardless of any caching settings.

Check PECL if it can detect the extension:

```bash
pecl list
```

You should see:

```bash
redis     5.2.1   stable
```

Even though the `redis` extension is listed it might still not be detected by PHP.
To make sure we need to check with PHP itself:

```bash
php -m
```

Make sure `redis` is included in the output list of PHP installed extensions.

Also check the extension exists at `/usr/local/lib/php/pecl/20190902/redis.so` and check the top of the `php.ini` file at `/usr/local/etc/php/7.4/php.ini` includes the line `extension="redis.so"`.

If it does not exists, make sure to add it manually so  PECL or PHP detect the extension.

> Note: The path of `php.ini` file may be different your PHP installation.

## Install Php Redis extension on Ubuntu

> Installation instructions for php-redis extension can be found at `https://github.com/phpredis/phpredis/blob/develop/INSTALL.markdown` and `https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster`. The steps are detailed at the end of the article.

## The Redis database driver configuration

The Laravel redis configuration is specified in the `config/database.php` file.

The `config/database.php` file has a list of drivers.

This list has driver configurations for all types of data stores used within a Laravel application.

For instance driver named 'redis' for Redis data stores and a driver named 'connections' for SQL databases.

So the Redis driver configuration is specified by the `'redis'` key in the file as shown below:

```php
'redis' => [
        #driver uses phpredis client as default
        # no need to set REDIS_CLIENT=phpredis in .env to specify phpredis explicitly
        'client' => env('REDIS_CLIENT', 'ppredis'),

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

Some drivers like the 'redis' driver can have a 'client' setting `'client' => env('REDIS_CLIENT', 'phpredis'),` that represents the php client that it uses to connect.

By default the client is set to `phpredis` to use the `phpredis` php extension to connect to Redis.

As mentioned at the beginning of the article you can also set the client to or `predis` to use the `predis` composer package to connect to Redis.

Each driver can have one or more connections. Each connection represents the connection settings for the particular type of database that the driver supports.

So for example `'redis'` driver has the `'default'` and `'cache'` connections included by default and
the `'connections'` database driver has `'sqlite'` and `'mysql'` connections among others that are included by default.

You can add as many connections with different connection settings as you need to each driver.

The `'default'` redis connection is used by the Laravel Redis facade by default. The Laravel Redis facade can use any of the `'redis'` connections by explicitly referencing the connection by connection name.

Examples of using both the default connection and named connections will be shown in the following sections.

## Avoiding namespace conflicts when using the default phpredis client

The only down side of using the default `'phpredis'` redis client is that you can not use the Laravel redis facade alias within your application.

This is because the `phpredis` extension has a class named `Redis` that conflicts with the `Redis` facade class name. So in order to use the `Redis` facade in our application code we must either fully qualify the class name with the class namespace or import the namespace into the file where we use the facade class. Both of these cases is demonstrated below:

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

Alternatively you can rename the 'Redis' facade alias in `config/app.php` file so that it will not conflict with the class of the same name defined by `phpredis`:

So the original alias name `Redis` specified in `config/app.php`:

```php
aliases' => [
    'Redis' => Illuminate\Support\Facades\Redis::class,
]
```

Is changed to `ZRedis` for example:

```php
aliases' => [
    'ZRedis' => Illuminate\Support\Facades\Redis::class,
]
```

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

> Note: the default value of `redis` for the second argument is used because the .env file does not contain a REDIS_CLUSTER setting by default. If you define this variable in .env you must set its value to `redis` to tell the framework to use the managed cluster.

Since I am always using the DigitalOcean Redis cluster, I just hard code this value to `'cluster' => 'redis'`

Instructions to setup laravel for connecting with the digital ocean managed Redis cluster can be found at:
`https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster`

I have outlined the steps to install the PECL `phpredis` extension on Ubuntu from the article:

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

> Note: I am using PHP version 7.4, adjust version as needed as long is it is 7.2 or above

Open the .env file and add the Redis Cluster Credentials:

```ini
REDIS_HOST=tls://your_redis_host.db.ondigitalocean.com
REDIS_PASSWORD=your_redis_password
REDIS_PORT=25061
```

> Note: We need use the `tls://` part before the name of the Redis cluster
