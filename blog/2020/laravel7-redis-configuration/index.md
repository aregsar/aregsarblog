# Laravel7 Redis Configuration

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


##########################################################################################
# LARAVEL CACHE AND QUEUE CONFIG





##################################################################################################

config/cache.php:

has list of stores e.g redis or file stores
each store has a driver like 'redis' (declared in config/database.php) or 'file'
and possibly a connection(from one of the named connections associated with the driver
in config/database.php) like 'default' or 'cache' if the driver is one that is
specified in config/database.php
 



   'stores' => [
        # we can explicitly
        'redis' => [
            # uses the redis driver defined in config/database.php
            'driver' => 'redis',#the 'redis' driver in config/database.php
            'connection' => 'cache',#uses the 'cache' configuration of the 'redis' driver in config/database.php
        ],
        'file' => [
            #uses the file driver
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],

    ],


#select a store from stores specified in config/cache.php
$value = Cache::store('file')->get('foo');

    #uses the 'default' store specified in config/cache.php by default
    $value = Cache::get('key');

    #setting in config/cache.php selects the file store
    'default' => 'file'

    #setting in config/cache.php selects the redis store by default
    'default' => 'redis'

    #setting in config/cache.php  selects a store based on env variable
    #the second argument sets the default to the file store if
    #the env variable is not present
    'default' => env('CACHE_DRIVER', 'file'),

    The last statement that uses the env variable is present when a new larave
    project is created

######################################

 config/queue.php has conections
'redis' => [
            'driver' => 'redis',
            'connection' => 'default',
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],

also has a default  connection selected by name from the connections in  config/queue.php
'default' => env('QUEUE_CONNECTION', 'sync'),

#uses the default connection setting in config/queue.php which includes the queue name to be used
#the connection also specifies the driver and associated connection specified in config/database.php
Queue::push(new InvoiceEmail($order));

#names the queue explicitly but uses the default connection setting in config/queue.php
#overrides the queue name specified in the default connection setting
Queue::pushOn('emails', new InvoiceEmail($order));

#explicitely selecting a queue connection from config/queue.php connections
$connection = Queue::connection('connection_name')->push(new InvoiceEmail($order));

#Alternatively can get a queue instance from queue manager
$queueManager = app('queue');
$queue = $queueManager->connection('redis');
$queue->push(new InvoiceEmail($order));







###############################################################################



config/session.php:

    # set SESSION_DRIVER=redis in .env
    # uses the 'redis' driver defined in config/database.php
    'driver' => env('SESSION_DRIVER', 'file'),


    #uses the 'cache' configuration of the 'redis' driver in config/database.php
    # add SESSION_CONNECTION=default or SESSION_CONNECTION=cache to .env
    # don't know if null parameter defaults to 'default' connection if SESSION_CONNECTION is not specified
    #'connection' => env('SESSION_CONNECTION', null),



#################################################################################

#config for the Cache facade used in the application
config/cache.php:

    # when using cache facade without specifying store this will be used to refer to
    # the 'store' we are using by default
    # set CACHE_DRIVER=redis in .env to set the default store to 'redis'
    # refers to one of the stores specified in the 'stores' array in this file
    # if env var not specified, the second param 'file' refers to the 'file' store in the 'stores' array in this file
    'default' => env('CACHE_DRIVER', 'file'),
   
    'stores' => [
        # we can explicitly
        'redis' => [
            # uses the redis driver defined in config/database.php
            'driver' => 'redis',#the 'redis' driver in config/database.php
            'connection' => 'cache',#uses the 'cache' configuration of the 'redis' driver in config/database.php
        ],
        'file' => [
            #uses the file driver
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],

    ],



################################################################################

config/queue.php:
 #selects default sync connection from connections array in this file
 #QUEUE_CONNECTION=redis in .env file
 #existing QUEUE_CONNECTION=sync in .env file
 'default' => env('QUEUE_CONNECTION', 'sync'),

 'connections' => [

        #sync connection
        'sync' => [
            'driver' => 'sync',
        ],

        #redis connection
        'redis' => [
            #can add and set from QUEUE_REDIS_DRIVER in .env
            'driver' => 'redis',#hardcodes 'redis' driver from config/database.php
            #can add and set from QUEUE_REDIS_CONNECTION in .env
            'connection' => 'default',#hard coded 'default' connection from 'redis' driver connection in config/database.php
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
]




