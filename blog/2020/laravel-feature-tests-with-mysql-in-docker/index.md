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

During execution of feature tests we need to apply migrations to the test database then run the tests against the test database.

There are two approaches we can use to do so.

With both approaches we override the DB_PORT setting to switch the port value from 8001 to 8011 to connect to the test database server.

After we override the setting, the `mysql` default database connection will connect to the test database thereby running the migrations and tests against the test database.

## Approach 1 - Overriding DB_PORT setting in the phpunit.xml file

The phpunit.xml file has php section where you can specify any environment settings you want to override before running tests.

Below is the out of the box overrides defined in phpunit.xml after a new laravel project installation:

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

So by adding a `<server name="DB_PORT" value="8011"/>` element we can override the `DB_PORT` setting specified in the .env file.

Database 
The overridden setting will be used when running the migrations 

## Approach 2 - Overriding DB_PORT setting in .env.testing file

If we specify a .env.testing file in the root of our project, phpunit will use that file instead of the standard .env file to set the environment values for running tests.

So we can override the port value by copying the .env file to a .env.testing file and change the port to 8011 in the .env.testing file:

```ini
DB_PORT=8011
```

> Note: We can change other settings in the .env.testing file as well if we need to. For example we can change the cache driver to array and session driver to array for example.

It important to note that the overrides section in phpunit.xml will now override the settings in the .env.testing file. So we need to remove those overrides from phpunit.xml if we do not need those overrides.

It is probably a good idea to just override any settings we need to override for testing in the .env.testing file, to have all our overrides in one file.

## Choosing the approach to override the DB_PORT setting

The approach of using the .env.testing file is useful when we need to run tests in parallel. This is because we need to run the database migrations outside the unit tests and we need to specify the .env.testing file to be used to run the database migrations agains the test server.

This will be subject of my next blog post.

If you do not need to run tests in parallel, then the approach of using the phpunit.xml file is simpler as you do not need to create a new .env.testing file and have to keep the values of the un-changed settings in sync as you make changes to the standard .env file.

In Laravel tests we use the `RefreshDatabase` trait to truncate the test database tables and apply the migrations before running tests. When connecting to in memory databases this happens before each test. However when running agains a real database with transactional capabilities, the trait only applies the migrations before running the first test and uses transactions to roll back any changes that were made during each test.








