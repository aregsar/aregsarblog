# Laravel Feature Tests With MySQL In Docker

May 21, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Feature tests using a production database

In the blog post [My Laravel Local Docker Services Configuration](https://aregsar.com/blog/2020/my-laravel-local-docker-services-configuration) I described how I use MySQL running in docker for local development.

In this article I will show you how to use a MySQL test database server running in a docker container to run your Laravel phpunit tests against.

We will set up a separate MySQL server instance used for testing that will run in its own separate docker container . This way our test database will be completely separate from our development database to avoid impacting the development database when running tests.

To setup a new MySQL docker container we can duplicate the existing MySQL service in the docker compose file and just give it a different port mapping.

Here is the docker compose file with the two MySQL services:

```yml
version: "3.1"
services:
    # this is the application database server used when running the app locally
    myapp-mysql:
      image: mysql:8.0
      container_name: app-mysql
      # persist data to data/mysql directory to save data accross container runs
      volumes:
        - ./data/mysql:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # upon container first run a database with the name of MYSQL_DATABASE setting will be created
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD=${DB_PASSWORD}"
      ports:
        # connect to the application server through port 8001 on localhost
        - "8001:3306"

    # this is the test database server used to run features tests against
    myapp-mysql-test:
      image: mysql:8.0
      container_name: app-mysql-test
      # no volume mapping required since we dont need the data to persist after container is shut down
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # upon container first run a database with the name of MYSQL_DATABASE setting will be created
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        # connect to the test server through port 8011 on localhost
        - "8011:3306"
```

And here is the related environment variable settings from the `.env` file:

```ini
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
```

Note the `DB_PORT` is currently set to the value of the development database port mapping.

When we run our phpunit tests on our local host we will connect to the port mapped to the `myapp-mysql-test` test database.

## Connecting to the test database

If we look in the `config/database.php` file we can see that at the moment we only have one MySQL connection named `mysql`. This connection is the default database connection that the application uses and the database migrations run against.

The default database connection is specified in the `config/database.php` file as shows below:

```php
'default' => env('DB_CONNECTION', 'mysql')
```

During execution of feature tests we need to apply migrations to the test database then run the tests against the test database by overriding the `DB_PORT` setting.

During execution of feature tests when the RefreshDatabase or RunsMigrations traits are used to run database migrations the `default` connection is used.
Also when we run php the `artisan migrate command` the `default` connection is used.

Similarly during our unit tests, we test application code that uses Laravels `Eloquent` ORM which uses the `default` connection.

The `default` connection is set to use the `mysql` connection which is configured to use the `DB_PORT` setting value.

After the `DB_PORT` setting is overridden, the `mysql` default database connection will connect to the test database thereby running the migrations and tests against the test database.

There are two approaches we can use run tests and migrations against the test database. Both approaches override the `DB_PORT` setting to switch the port value from `8001` to `8011` to connect to the test database server.

## Approach 1 - Overriding DB_PORT setting in the phpunit.xml file

The `phpunit.xml` file has a php section where you can specify any environment settings you want to override before running tests.

Below is the out of the box overrides defined in `phpunit.xml` after a new Laravel project installation:

```xml
<php>
    <server name="APP_ENV" value="testing"/>
    <server name="BCRYPT_ROUNDS" value="4"/>
    <server name="CACHE_DRIVER" value="array"/>
    <server name="DB_CONNECTION" value="sqlite"/>
    <server name="MAIL_MAILER" value="array"/>
    <server name="QUEUE_CONNECTION" value="sync"/>
    <server name="SESSION_DRIVER" value="array"/>
</php>
```

So by adding a `<server name="DB_PORT" value="8011"/>` element we can override the `DB_PORT` setting specified in the `.env` file.

The overridden setting will then be used when running the migrations and unit tests.

## Approach 2 - Overriding DB_PORT setting in a .env.testing file

If we specify a `.env.testing` file in the root directory of our project, `phpunit` will use that file instead of the standard `.env` file to set the environment variable values when running tests.

So we can override the port value by copying the `.env` file to a `.env.testing` file and change the `DB_PORT` value to 8011 in the `.env.testing` file:

```ini
DB_PORT=8011
```

> Note: We can change other settings in the `.env.testing` file as well if we need to. For example we can change the cache driver to `array` and session driver to `array` to test agains in memory cache and session stores.

It is important to note that the overrides section in `phpunit.xml` will now override the settings in the `.env.testing` file instead of the `.env` file. So we need to remove those overrides from `phpunit.xml` since we do not want them to be overridden.

## Choosing the approach to override the DB_PORT setting

Normally in the unit test classes we use the Laravel `RefreshDatabase` trait to migrate the database as part of the unit tests.

When doing parallel testing, this approach has issues. In the [Laravel Run Parallel Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker) blog post I describe how to overcome those issues and why we need to use the `.env.testing` file.

The approach of using the `.env.testing` file is useful when we need to run tests in parallel. This is because we can not use the `RefreshDatabase` trait when running tests in parallel and therefore need to run the database migrations outside the unit tests by using the `artisan migrate` command.  

For the `artisan migrate` command to be able to run against the test database we need to override the `DB_PORT` setting that the migrate command uses. We do so by passing the separate `.env.testing` file name, that overrides the `DB_PORT` setting value, as an argument to the migrate command.

If you do not need to run tests in parallel, then the approach of using the `phpunit.xml` file to override the `DB_PORT` value is simpler as you would not have to keep the `.env.testing` file in sync with other changes to the `.env` file.

## Setting up phpunit to use `RefreshDatabase` trait

If we are not running tests in parallel, we need to use the `RefreshDatabase` trait in our phpunit classes so that the test database is refreshed after each test.

In order to not repeat ourselves in every test class we can extend the base `Tests\TestCase` phpunit class and add the trait there. Then our test classes can extend the derived `TestCase` class to take advantage of the `RefreshDatabase` trait.

```php
namespace Tests\Feature;

use Tests\TestCase as BaseTestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

abstract class TestCase extends BaseTestCase
{
    use RefreshDatabase;
}
```
