# Laravel Feature Tests With MySQL In Docker

May 21, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Feature tests using a production database

In the blog post [My Laravel Local Docker Services Configuration](https://aregsar.com/blog/2020/my-laravel-local-docker-services-configuration) I described how I use MySQL running in docker for local development.

In this article I will show you how to use a MySQL test database server running in a docker container to run your Laravel phpunit tests against.

We will set up a separate MySQL server instance running in a separate docker container used for testing. This way our test database will be completely separate from our development database so that we can setup the test database with test data.

To setup a new MySQL docker container we can duplicate the existing MySQL service in the docker compose file and just give it a different port mapping. So when running our phpunit tests on our local host we can connect to the port mapped to the test database.

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

And here are the related environment variable settings in the `.env` file:

```ini
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
```

Now in the in `config/database.php` file we have just one MySQL connection named `mysql`.
This connection is the default database connection that the application and database migrations use. 

The default database connection is specified in the `config/database.php` file as shows below:

```php
'default' => env('DB_CONNECTION', 'mysql'),
```

## Connecting to the test database

During execution of feature tests we need to apply migrations to the test database then run the tests against the test database.

There are two approaches we can use to do so.

With both approaches we override the `DB_PORT` setting to switch the port value from 8001 to 8011 to connect to the test database server.

After we override the setting, the `mysql` default database connection will connect to the test database thereby running the migrations and tests against the test database.

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

> Note: We can change other settings in the `.env.testing` file as well if we need to. For example we can change the cache driver to `array` and session driver to `array` for example.

It is important to note that the overrides section in `phpunit.xml` will now override the settings in the `.env.testing` file instead of the `.env` file. So we need to remove those overrides from `phpunit.xml` since we do not want them to be overriden.

## Choosing the approach to override the DB_PORT setting

The approach of using the `.env.testing` file is useful when we need to run tests in parallel. This is because we need to run the database migrations outside the unit tests and we need to specify the `.env.testing` file to be used to run the database migrations on the test database server.

In the unit test classes we use the Laravel `RefreshDatabase` trait to migrate the database as part of the unit tests.

When doing parallel testing, this approach has issues. In the [Laravel Run Parallel Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker) blog post I describe how to overcome those issues and why we need to use the `.env.testing` file.

If you do not need to run tests in parallel, then the approach of using the `phpunit.xml` file overrides instead of using a separate `.env.testing` file is simpler as you do not have to keep the `.env.testing` file in sync with the changes to the `.env` file.

## Setting up phpunit to use `RefreshDatabase` trait

We need to use the `RefreshDatabase` trait in our phpunit classes so that the test database is refreshed after each test.

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
