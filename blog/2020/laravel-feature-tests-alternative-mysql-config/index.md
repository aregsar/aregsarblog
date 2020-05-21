# Laravel Feature Tests Alternative MySQL Config

May 21, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In the previous two blog posts [Laravel Feature Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-feature-tests-with-mysql-in-docker) and [Laravel Run Parallel Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker) I showed you feature test configurations for running tests with a MySQL test database running in docker. The first post covered configurations for running tests normally the other for running tests in parallel.

In this post I will outline on other alternative configuration that can be used for either running tests normally or in parallel without having to create a `.env.testing` file as was required when running tests in parallel.

> See either of the blog posts mentioned above to update your docker-compose configuration file to run a test MySQL database server.

## Adding a test database connection to config/database.php

First we need to add our test database configuration to config/database.php by copying our default `mysql` configuration and renaming it to `test-mysql` and  change the `port` setting in the in the `test-mysql` connection to use the `TEST_DB_PORT` env variable setting.

The result in the config/database.php file is shown here:

```php
# the default database connection setting
'default' => env('DB_CONNECTION', 'mysql'),

'connections' => [
# the default database connection
'mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '8001'),
            'database' => env('DB_DATABASE', 'radar'),
            'username' => env('DB_USERNAME', 'radar'),
            'password' => env('DB_PASSWORD', 'radar'),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],
#the test database connection copy of the 'mysql' connection
'test-mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            # changed the port setting to use TEST_DB_PORT
            'port' => env('TEST_DB_PORT', '8001'),
            'database' => env('DB_DATABASE', 'radar'),
            'username' => env('DB_USERNAME', 'radar'),
            'password' => env('DB_PASSWORD', 'radar'),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],
]
```

Then we need to add the `TEST_DB_PORT` environment variable to the `.env` file, setting it to the docker mapped port value in the docker-compose.yml file for the test MySQL database server running in docker:

```ini
TEST_DB_PORT=8011
```

## Overriding DB_CONNECTION in phpunit

In order to run tests agains the test database we need to override the DB_CONNECTION setting in the `.env` file to use the `test-mysql` connection.
Since the `test-mysql` uses the `TEST_DB_PORT` setting, the tests will connect to the test database when run.

By overriding the DB_CONNECTION to use test-mysql we are changing the default
database connection used to run the migrations and tests:

```xml
<server name="DB_CONNECTION" value="test-mysql"/>
```

If we are not running our tests in parallel the this is the only setting we need to override to run our tests. We just need to make sure we are also using the `RefreshDatabase trait` in our tests.

If we are running our tests in parallel then we need to configure our database migrations as shown in the next section.

## Connecting database migrations to test database for parallel tests

As mentioned in the parallel testing blog post, we need to run migrations manually on the command line before running the tests instead of using the the `RefreshDatabase` trait to run the migrations prior running the tests.

When running the migrations manually we need to specify which database the migrations run against in the command.

Since we are not using a separate `.env.testing` file, we have to specify the database to use by using value the overridden `DB_CONNECTION` setting in the phpunit.xml file. The way we do that is to pass the test database value using the `database` flag in the migration command:

```bash
php artisan migrate --database=test-mysql
```

Again since the specified `test-mysql` connection uses the `TEST_DB_PORT` setting, the migrations will run against the test database running in docker.

Using this migration command we can follow the instructions in [Laravel Run Parallel Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker) to run our tests in parallel.

The only difference is that we use the `--database=test-mysql` flag instead of the `--env=testing` flag with the `php artisan migrate` command.
