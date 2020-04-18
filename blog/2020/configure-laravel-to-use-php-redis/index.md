# Configure Laravel To Use Php Redis

April 10, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: These are installation instructions for Laravel 7. The post will get updated as needed newer versions of Laravel
> Prerequisite: must have PHP and PECL installed. The MacOS Hombrew PHP installation installs PECL as well. On Ubuntu you need to install PECL separately using apt package manager. You can find Ubuntu install instructions and link to instructions for connecting to the DigitalOcean redis cluster at the end of the article.
> You might find the redis-cli cli docs at `https://redis.io/topics/rediscli` useful for working with Redis. Alternatively Medis `http://getmedis.com/` app and `https://tableplus.com/blog/2018/06/best-redis-gui-client-tableplus.html` are nice GUI redis clients for working with redis data.

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

Some drivers like the 'redis' driver can have a 'client' setting `'client' => env('REDIS_CLIENT', 'phpredis'),` that represents the php client that it uses to connect.

By default the client is set to `phpredis` to use the `phpredis` php extension to connect to Redis.

As mentioned at the beginning of the article you can also set the client to or `predis` to use the `predis` composer package to connect to Redis.

Each driver can have one or more connections. Each connection represents the connection settings for the particular type of database that the driver supports.

So for example `'redis'` driver has the `'default'` and `'cache'` connections included by default and
the `'connections'` database driver has `'sqlite'` and `'mysql'` connections among others that are included by default.

You can add as many connections with different connection settings as you need to each driver.

The `'default'` redis connection is used by the Laravel Redis facade by default. 

The Laravel Redis facade can use any of the `'redis'` connections by explicitly referencing the connection by connection name.

Examples of using both the default connection and named connections are shown in the following sections.

## Avoiding namespace conflicts when using the default phpredis client

There is one downside of using the default `'phpredis'` Redis client. The downside is that you will not be able to use the default Laravel Redis facade alias specified in the `config/app.php` file within your application unless it is renamed.

This is because the `phpredis` extension has a class named `Redis` that conflicts with the `Redis` facade class name.

If you choose not to rename the alias, then you need to use the `Redis` facade class directly using the full facade class namespace.

To use the `Redis` facade in our application code we must either fully qualify the class name with the class namespace or import the namespace into the file where we use the facade class. Both of these cases is demonstrated below:

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

Alternatively, as mentioned before you can rename the Redis facade alias in `config/app.php` file so that it will not conflict with the class name defined by the `phpredis` extension:

The original alias name `Redis` specified in `config/app.php` as shown here:

```php
aliases' => [
    'Redis' => Illuminate\Support\Facades\Redis::class,
]
```

As an example I have changed the name to `ZRedis`:

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

When using the redis facade, if no connection is explicitly specified, then the facade uses the `default` connection configuration of the `'redis'` driver.

Here the default connection is used:

```php
Illuminate\Support\Facades\Redis::set('name', 'Taylor');
```

Here also the default connection is used:

```php
Illuminate\Support\Facades\Redis::connection()->set('name', 'Taylor');
```

But here the `'cache'` connection from `config/database.php` is used:

```php
Illuminate\Support\Facades\Redis::connection('cache')->set('name', 'Taylor');
```

## Configuring Laravel to use a managed Redis cluster

While you could add your own redis cluster configuration in the Clusters section of the `'redis'` driver, I like using a managed cluster. 

In order to use a managed cluster you only need to add the line `'cluster' => env('REDIS_CLUSTER', 'redis')` to the 'options' array in the `'redis'` driver in `config/database.php` shown below:

```bash
'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],
```

The line `'cluster' => env('REDIS_CLUSTER', 'redis')` tells the framework to use the default clustering capability of a managed `redis` cluster.

> Note: the default value of `redis` for the second argument is used because the default `.env` file does not contain a `REDIS_CLUSTER` environment setting. If you define this variable in the `.env` file, you must set its value to `redis` to tell the framework to use a managed cluster.

Since I am always using the DigitalOcean Redis cluster, I just hard code this value in `config/database.php` to `'cluster' => 'redis'`:

```php
'options' => [
        'cluster' => 'redis',
    ],
```

Instructions to setup Laravel for connecting with the DigitalO cean managed Redis cluster can be found at:
`https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster`

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

> Note: I am using PHP version 7.4, adjust version as needed as long is it is 7.2 or above

Open the .env file and add the Redis Cluster Credentials:

```ini
REDIS_HOST=tls://your_redis_host.db.ondigitalocean.com
REDIS_PASSWORD=your_redis_password
REDIS_PORT=25061
```

> Note: We need use the `tls://` part before the name of the Redis cluster

An now we can test the connections add the following route to a new controller that you can add named `RedisController`:

```php
Route::get('/redis', 'RedisController@redisTest');
```

Then add the following `redisTest` method to the controller:

```php
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

## Setting the redis database in redis connections settings

Each Redis server you configure a connection for in the config file will have its own unique set of host and port settings. However multiple connections can be configured to connect to the same Redis server but using a different database.

The database value must be an integer between 0 and 15 because Redis supports only up to 16  databases. The connection string to redis will include the database as a path segment `redis://host:port/database` e.g. `redis://localhost:6379/0`

Here is the default out of the box connections configured in `config/database.php`:

```bash
'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '0'),
        ],
'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],
```

You can see that for laravel specifies `default` connection that defaults the database value to 0 and a `cache` connection that defaults the database value to 1.
This works because the common REDIS_CACHE_DB env key used for both connection databases is not specified by default in the .env file.

> Note: Alternatively we could change the env key name to 'REDIS_CACHE_DB_DEFAULT' for he 'default' connection database and 'REDIS_CACHE_DB_CACHE' for the 'cache' connection database and add them to the .env file as REDIS_CACHE_DB_DEFAULT=0 and REDIS_CACHE_DB_CACHE=1.

By using a unique database number for each connection, the `'default'` connection with database 0 can be used for the Redis provider/facade that the application uses to directly work with the Redis data store and the `'cache'` connection with database 1 can be used by the Laravel Cache facade for caching data.

We can acheive this separation by configuring The Laravel Cache provider/facade to use the `'redis'` driver with the `'cache'` connection.

In this way both the default and cache connections can use the same DigitalOcean clustered server but store their data in separate databases.

In my apps I like to also add two other Redis driver connections with separate databases each for the Laravel Session and Queue Facades to use.

```bash
'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '0'),
        ],
'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

'queue' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_QUEUE_DB', '2'),
        ],

'session' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_SESSION_DB', '3'),
        ],
```

> Note: the `database` setting for the `queue` and `session` settings is also updated to use `REDIS_QUEUE_DB` and `REDIS_SESSION_DB` environment settings accordingly

In separate articles I will show how to configure the Laravel Session, Cache and Queue to use the redis driver with specific redis connections.

## Setting per connection key prefix when using the PhpRedis driver

According to Laravel 7 docs PhpRedis also supports the following additional connection parameters: prefix, persistent, read_timeout and timeout

Below I have added a data key prefix to the 'cache' connection while the 'default' connection does not use a prefix.

```bash
'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '0'),
        ],
'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
            'prefix' => 'cache:'
        ],
```

With this configuration, all keys used by the Laravel application when using the `'cache'` connection will have the prefix `cache:` prepended to the key string.

> Note: Instead of using databases to segment your redis data based on the connection database setting you can make all connections use the same database with each connection setting its own unique key prefix.

## Setting the key prefix if using the predis driver

If we change the default redis driver to use the old predis driver per connection key prefix setting does not work.

For predis we can only set a common prefix inside the 'options' setting::

```bash
'options' => [
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],
```

The default out of the box .env file does not contain REDIS_PREFIX key so the second parameter of the env() function specifies a prefix using the APP_NAME. You can remover the out of the box prefix setting if you don't want a prefix.
