# Laravel Feature Tests Alternative MySQL Config

May 21, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In the previous two blog posts [Laravel Feature Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-feature-tests-with-mysql-in-docker) and [Laravel Run Parallel Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker) I showed you feature test configurations for running Laravel feature tests with a MySQL test database running in docker. The first post covered configurations for running tests normally the other for running tests in parallel.

In this post I will outline an alternative testing configuration that can be used for either running tests normally or in parallel without having to create a `.env.testing` file as was required when running tests in parallel.

> See either of the blog posts mentioned above to setup your docker-compose configuration file to run a MySQL test database server.

## Adding a test database connection to config/database.php

First we need to add our test database configuration to the `config/database.php`file by copying the default `mysql` connection configuration in `config/database.php` and renaming the copy to `test-mysql`. Then we need to make the `port` setting of the `test-mysql` connection to use a `TEST_DB_PORT` environment variable setting instead of the original `DB_PORT` setting.

The result of the change in the `config/database.php` file is shown here:

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
            # the changed port setting that uses TEST_DB_PORT instead of DB_PORT
            'port' => env('TEST_DB_PORT'),
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

Since we added the `TEST_DB_PORT` environment variable to the configuration file, we need to add it to the `.env` file. Then we need to set its value to the local host mapped port value of the test MySQL database server port specified the `docker-compose.yml` file.

So if we have the following `mysqltest` test database configuration in `docker-compose.yml`:

```yaml
    mysqltest:
      image: mysql:8.0
      container_name: radar-mysql-test
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        # port mapping
        - "8011:3306"
```

Then we need to add the following environment variable to the `.env` file:

```ini
TEST_DB_PORT=8011
```

## Overriding DB_CONNECTION in phpunit.xml

In order to run our tests against the test database we need to override the `DB_CONNECTION` setting in the `.env` file and set it to `test-mysql` so that the default database connection uses the `test-mysql` in the `config/database.php` file.

By overriding the `DB_CONNECTION` we are changing the default database connection used to run the migrations and tests to `test-mysql` and since the `test-mysql` connection uses the `TEST_DB_PORT` setting, the tests will connect to the test database running in docker.

In the `phpunit.xml` file we can override the `DB_CONNECTION` value in by adding the following setting inside the `php` element in the file:

```xml
<server name="DB_CONNECTION" value="test-mysql"/>
```

If we are not running our tests in parallel then this is the only setting we need to override to run our tests. We just need to make sure that we are also using the `RefreshDatabase trait` in our test classes.

If we are running our tests in parallel then we need to also configure our database migrations to run against the test database server as shown in the next section.

## Connecting database migrations to test database for parallel tests

As mentioned in the parallel testing blog post, we need to run migrations manually on the command line before running the tests instead of using the the `RefreshDatabase` trait.

When running the migrations manually we need to specify which database the migrations run against in the migration command.

We can specify the database that migrations should use by setting the `--database` option of the artisan migrate command to the value of the `DB_CONNECTION` setting in the `phpunit.xml` file as shown below:

```bash
php artisan migrate --database=test-mysql
```

As noted before, since the specified `test-mysql` connection uses the `TEST_DB_PORT` setting, the migrations will run against the test database running in docker.

Using this migration command we can follow the instructions in [Laravel Run Parallel Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker) to run our phpunit tests in parallel.

