# Laravel Feature Tests With MySQL In Docker

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Laravel Feature Tests With MySQL In Docker](https://aregsar.com/blog/2020/laravel-feature-tests-with-mysql-in-docker)

## Feature tests using a real database

In this article I will show you how to use a real MySQL database server running in docker container to run your Laravel phpunit tests against.

In the blog post [my laravel local docker services configuration](https://aregsar.com/blog/2020/my-laravel-local-docker-services-configuration) I described how I use MySQL running in docker for local development. 

For testing we will set up a separate MySQL server instance running in a separate testing docker container. This way our test database will be completely separate from our development database just like how we would run separate database servers for our test, staging and production environments.

To setup a new MySQL docker container we can duplicate the existing MySQL service in the docker compose file and just give it a different port mapping. So from our local host we can connect to the port mapped to the test database to run our feature tests.

> It is assumed that the Laravel application and tests are running on local host.

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


Below are the environment variable settings in the .env file:

```ini 
DB_DATABASE=myapp
DB_USERNAME=myapp
DB_PASSWORD=myapp
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
```

Note that we have just one MySQL connection named `mysql` configured in `config/database.php` file.
This connection is the default database connection that the application, database migrations and phpunit tests all use. The default database connection is specified in the `config/database.php` file as shows below:

```php
'default' => env('DB_CONNECTION', 'mysql'),
```

## Connecting to the test database

During execution of feature tests we need to apply migrations to the test database then run the tests against the database.

There are two approaches we can use to do so.

Both approaches we override the DB_PORT=8001 setting to switch the port value from 8001 of the application database server to 8011 of the test database server.
After we override the setting, the `mysql` default database connection will connect to the test database thereby running the migrations and tests against the test database.

## overriding DB_PORT setting in the phpunit.xml file

The phpunit.xml file has php section where you can specify environment settings you want to override before running tests. 

## overriding DB_PORT setting in .env.testing file

If we specify a .env.testing file in the root of our project, phpunit will use that file instead of the standard .env file to set the environment values for running tests. 

So we can override the port value by copying the .env file to a .env.testing file and change the port to 8011 in the .env.testing file.

> Note: We can change other settings in the .env.testing file as well if we need to. For example we can change the cache driver to array and session driver to array for example.

## Choosing the approach to override the DB_PORT setting

The approach of using the .env.testing file is useful when we need to run tests in parallel. This is because we need to run the database migrations outside the unit tests and we need to specify the .env.testing file to be used to run the database migrations agains the test server.

This will be subject of my next blog post.

If you do not need to run tests in parallel, then the approach of using the phpunit.xml file is simpler as you do not need to create a new .env.testing file and have to keep the values of the un-changed settings in sync as you make changes to the standard .env file.







