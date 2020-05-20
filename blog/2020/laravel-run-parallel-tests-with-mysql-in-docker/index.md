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

In Laravel tests we use the `RefreshDatabase` trait to truncate the test database tables and apply the migrations before running tests. When connecting to in memory databases this happens before each test. However when running agains a real database with transactional capabilities, the trait only applies the migrations before running the first test and uses transactions to roll back any changes that were made during each test.



