# Laravel Run Parallel Tests With MySQL In Docker

May 21, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show you how to run phpunit tests using a MySQL test database server while avoiding issues related to database refresh and migrations when running tests in parallel.

When running tests in parallel using the `RefreshDatabase` trait is problematic.

## Parallel Feature tests using a MySQl

In the blog post [Laravel Feature Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-feature-tests-with-mysql-in-docker)
I described how we can connect to a MySQL test database server running in docker when running phpunit feature tests.

For reference I will repeat the configuration from that post below.

Here is the docker compose file:

```yml
version: "3.1"
services:
    # this is the application database server used when running the app
    myapp-mysql:
      image: mysql:8.0
      container_name: app-mysql
      # persist data to data/mysql directory to save data accross container runs
      volumes:
        - ./data/mysql:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # a database with the name of the MYSQL_DATABASE setting will be created on the first run of this container
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
      ports:
        # connect to this database server through port 8001 on localhost
        - "8001:3306"

    # this is the test database server used when running phpunit tests
    myapp-mysql-test:
      image: mysql:8.0
      container_name: app-mysql-test
      # no volume mapping required since we dont need the data to persist after container is shut down
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # a database with the name of the MYSQL_DATABASE setting will be created on the first run of this container
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD=${DB_PASSWORD}"
      ports:
        # connect to this test database server through port 8011 on localhost
        - "8011:3306"
```

The file has a MySQL service for app development and a separate MySQL service for testing:

Below are the environment variable settings in the `.env` file from the aforementioned blog post:

```ini 
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
```

## The RefreshDatabase trait and parallel tests

In Laravel tests we use the `RefreshDatabase` trait to truncate the test database tables and apply database migrations before running tests.

When connecting to in memory databases this happens before each test. However when running against a database such as MySQL with transactional capabilities, the trait only applies database migrations one time before running the first test and uses transactions to roll back any changes made to the database between subsequent tests.

The way the `RefreshDatabase` trait achieves this under the hood is by first, applying the `RunsMigrations` trait which is responsible for running the database migrations only once before any tests are run. Second, by applying the `UsesTransactions` trait which starts a database transaction before running each test and rolls back the transaction after the test.

By rolling back the transaction, the database state goes back to the state right after applying the migrations and we don't have to truncate the tables and run the migration again before each test which reduces performance.

When tests are run serially using the `RefreshDatabase` trait works without issues. However when test are run in parallel one or more tests will try to run the code associated with the `RunsMigrations` trait at the same time. This will cause migrations being applied simultaneously by multiple tests, even though migrations only run once at the start of testing.

The solution to this problem is to run the migrations just once manually before running phpunit tests. Then we can use the `UsesTransactions` trait instead of the  `RefreshDatabase` trait in our tests so the tests do not run migrations but still rollback changes made during each test.

Since `RefreshDatabase` runs `UsesTransactions` after applying migrations, we are still doing exactly the same thing.

Since we won't be running the database migrations using the `RunsMigrations` trait anymore, therefore overriding the `DB_CONNECTION` environment setting to allow the database migrations to be applied to the test database by `RunsMigrations` trait will not work.

## Setting up the .env.testing file

In order to tell the artisan database migration command to run migrations against the test database server we must setup a new environment variable file for testing. The name of this file must be `.env.testing` so that by default phpunit will use it instead of the `.env` file. This way both the migration command and phpunit will be using the same test database settings.

In the .env.testing file we need to change the DB_PORT environment variable value to the mapped port number of the test database running in the docker container.

So first we need to copy the `.env` file to `.env.testing` file in the root directory of the Laravel project.

Then in the `.env.testing` file we can change the `DB_PORT` setting value from 8001 to 8011 to connect to the test database server port running in docker.

## Running migrations with Artisan using .env.testing

Now we need to pass the `.env.testing` file name to the `artisan db:migrate` command so that the database migrations will use the `DB_PORT` setting from the `.env.testing` file.

To allow the php artisan migrate command to use the `.env.testing` file we must pass it as a option flag to the command like so:

```bash
php artisan migrate --env=testing
```

## Setting up phpunit to automatically use DatabaseTransactions trait

For reasons mentioned above we will use the `DatabaseTransactions` trait instead of the `RefreshDatabase` in our unit tests.

So that we don't have to type `use DatabaseTransactions` in all our test classes, we can extend the base `BaseTestCase` class and add the trait there.

Then our test classes can extend the base `Tests\TestCase` class to take advantage of the `DatabaseTransactions` trait in all classes derived from that extended class.

```php
namespace Tests\Feature;

use Tests\TestCase as BaseTestCase;
use Illuminate\Foundation\Testing\DatabaseTransactions;

abstract class TestCase extends BaseTestCase
{
    use DatabaseTransactions;
}
```

So now all our classes will need to be derived from the `Tests\Feature\TestCase` class.

## Using Paratests to run phpunit tests in parallel

To install the parallel testing package run:

```bash
composer require --dev brianium/paratest
```

To execute tests using paratest run:

```bash
paratest --processes 4 --testsuite Feature --runner WrapperRunner
```

## Automating our test setup

In order to automate running the migration before running the tests in a single command we can create a composer script to run the commands in sequence:

```json
{
    "scripts": {
        "ft": [
            "php artisan migrate --env=testing",
            "./vendor/bin/paratest --processes 4 --runner WrapperRunner --testsuite Feature"
        ],
        "ut": [
            "./vendor/bin/paratest --processes 4 --runner WrapperRunner --testsuite Unit"
        ]
    }
}
```

> I have also added a setting for running unit tests with a short command.

Now all we have to do to run our feature tests in parallel is to type
`composer ft` on the command line.

This way you make sure that the migrations are run for feature tests before running the rests and make the command to run the tests is short and easy to type as well.

Alternatively you could add the following aliases in your bash profile:

```bash
ft="php artisan migrate --env=testing && ./vendor/bin/paratest --processes 4 --runner WrapperRunner --testsuite Feature"
```

```bash
ut=" ./vendor/bin/paratest --processes 4 --runner WrapperRunner --testsuite Unit"
```

Then simply type `ft` or `ut` while in the root directory of your project to run the feature or unit tests.
