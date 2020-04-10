# Configure Laravel Cache To Use Redis

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Configure Laravel Cache To Use Redis](https://aregsar.com/blog/2020/configure-laravel-cache-to-use-redis)

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

---------------------
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







