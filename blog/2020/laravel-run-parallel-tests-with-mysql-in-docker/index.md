# laravel run parallel tests with mysql in docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[laravel run parallel tests with mysql in docker](https://aregsar.com/blog/2020/laravel-run-parallel-tests-with-mysql-in-docker)

## Parallel Feature tests using a real database

In this article I will show you how to run phpunit tests that use a MySQL test database server in parallel while avoiding issues with running database migrations and using the standard `RefreshDatabase` trait.

In the blog post [Laravel Feature Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-feature-tests-with-mysql-in-docker)
I described how I use MySQL running in docker for running feature tests.

Here is the docker compose file from that blog post that has a MySQL service for app development and a separate MySQL service for testing:

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
      # tmpfs setting speeds up tests by using in memory persistance during tests
      tmpfs: /var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # upon container first run a database with the name of MYSQL_DATABASE setting will be created
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD=${DB_PASSWORD}"
      ports:
        # connect to the test server through port 8011 on localhost
        - "8011:3306"
```

Below are the environment variable settings in the .env file from the previous blog post:

```ini 
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
```

> It is assumed that the Laravel application and tests are running on local host.

## The RefreshDatabase trait and parallel tests

In Laravel tests we use the `RefreshDatabase` trait to truncate the test database tables and apply the migrations before running tests. When connecting to in memory databases this happens before each test. However when running agains a real database with transactional capabilities, the trait only applies the migrations one time before running the first test and uses transactions to roll back any changes that were made during each test.

When using a real database, the way the `RefreshDatabase` trait achieves this under the hood is by first, applying the `RunsMigrations` trait which is responsible for running the database migrations only once before any tests are run. Second, by applying the `UsesTransactions` trait which starts a database transaction before running each test and rolls back the transaction after the test. By rolling back the transaction, the database state goes back to the state after applying the migrations and we don't have to truncate the tables and run the migration again which reduces performance.

When tests are run serially then using the `RefreshDatabase` trait works. However when test are run in parallel one or more tests will try to run the code associated with the `RunsMigrations` trait that `RefreshDatabase` uses at the same time. This will cause migrations being applied simultaneously by multiple tests, even though migrations only should run once at the start of testing.

The solution to this problem is to run the migrations just once manually before running phpunit tests and using the `UsesTransactions` trait instead of the  `RefreshDatabase` trait in our tests so the tests do not run migrations. Since `RefreshDatabase` runs `UsesTransactions` after applying migrations, we are still doing exactly the same thing.

Now, since we have to run the database migrations outside the tests, we can not use the phpunit.xml environment variable overrides to allow the database migrations to be applied to the test database.

Therefore we must use the .env.testing file to override the DB_PORT environment variable so that we can pass the file name to the `artisan db:migrate` command so the migrations will use the overriden DB_PORT value from the .env.testing file.

Simultaneously, the phpunit testing framework will use the .env.testing file by default so that both the migrations and the tests will use the same test database.

## Setting up the .env.testing file

Copy the .env file to .env.testing file in the root of the project.

In the .env.testing file change the DB_PORT setting value from 8001 to 8011 to connect to the test database server port running in docker running on localhost.

## Setting up phpunit to use `RefreshDatabase`

So that we dont have to type `use DatabaseTransactions` in all our test classes, we can extend the base `BaseTestCase` class and add the trait there.

Then our test classes can extend the derived TestCase class to take advantage of the DatabaseTransactions trait.

```php
namespace Tests\Feature;

use Tests\TestCase as BaseTestCase;
use Illuminate\Foundation\Testing\DatabaseTransactions;

abstract class TestCase extends BaseTestCase
{
    use DatabaseTransactions;
}
```

## Running migrations with Artisan using .env.testing

To allow the php artisan migrate command to use the .env.testing file we must pass it as a option flag to the command:

```bash
php artisan migrate --env=testing
```

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
xxx on the command line.