# MySQL Configure Laravel

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

## Steps to configure Laravel to use MySQL

In this post we will configure Laravel to use a MySQL server running in a local docker container.

To start, go to the root of your Laravel project then follow steps below:

### Step 1 - Setup a MySQL sever docker service

Lets first setup a MySQL server using a Docker compose service, for local development that our application can connect to:

Create a docker-compose.yml file containing the mysql docker service.

```bash
echo docker-compose.yml << EOL
version: "3.1"
services:
    mysql:
    image: mysql:8.0
    container_name: mysql
    volumes:
        - ./data/mysql:/var/lib/mysql
    environment:
        - MYSQL_ROOT_PASSWORD="${DB_PASSWORD}"
        # upon container first run a database with the name of MYSQL_DATABASE setting will be created
        - MYSQL_DATABASE="${DB_DATABASE}"
        - MYSQL_USER="${DB_USERNAME}"
        - MYSQL_PASSWORD="${DB_PASSWORD}"
    ports:
        - "${DB_PORT}:3306"

EOL
```

If you already have a `docker-compose.yml` file in the project root then just add the mysql service portion of the above yml code under the services section.

We also need to create a directory in the Laravel project root to persist the mysql data.

If a directory named `data` does not exist in the root of the project, run the following:

```bash
# make sure git ignores the data directory
echo '/data' >> .gitignore
mkdir data
#mkdir -p data/mysql
```

The ./data/mysql volume will be created within the Laravel project directory that contains the docket-compose.yml file when we run the docker-compose up -d command.

### Step 2 - Set the connection settings for the docker MySQL service

Open the project `.env` file and set the connection settings for the docker mysql service.

Substitute your project name for `<Project_name>`.

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=8001
DB_USERNAME=root
DB_PASSWORD=123456
DB_DATABASE=<Project_name>
```

The docker-compose.yml file we setup before will use the settings from the .env file.

Connect to the mysql server using a database management tool and create a database named the same as the DB_DATABASE setting value.

## The MySQL default connection configuration

The default database driver configuration is already configured to use mysql as a fallback and uses the `DB_CONNECTION` that was set to mysql in the previous section.

Here is the default connection declared at the root level in then `config/database.php` file

```php
'default' => env('DB_CONNECTION', 'mysql'),
```

The default connection setting selects the `mysql` connection which is declared in the connections array at the root level in the `config/database.php` file shown below:

```php
    'connections' => [

        'mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
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
    ],
```

We dont need any additional configuration to connect to the default mysql configuration.

## Adding additional MySQL connections

We can add additional database connections in the `connections` array.

These connections can be used as non default connections by explicitly pass in the name of a non default connection name to the Laravel Eloquent and Query Builder APIs.

Below I show a new connection I added named mysql2:

```php
    'connections' => [

        'mysql2' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE_2', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
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
    ],
```

The only change I made to this connection is I am using a DB_DATABASE_2 .env file setting that can be set to a different database.

This will make this connection use the same mysql server that is running as a docker service but use a different database.

If you want to use the connection to connect to a separate server, then you would need to add another mysql docker service mapped to a different port on your host machine. Then add a separate DB_PORT_2 parameter to connect to the this mapped port.

## Testing the connection

```bash
php artisan tinker
```

```php
//use the default connection
DB::connection()->getPdo();
//use the connection explicitly
DB::connection("mysql")->getPdo();
```

## Connecting to MySQL using non default connections

Instead of using the default database connection, you can explicitly pass in the name of a non default connection to the Laravel Schema, Eloquent and Query Builder APIs.

Below I have listed some examples of using the connection from the previous step to connect to the database that are from [laravel-multiple-database-connections](https://fideloper.com/laravel-multiple-database-connections) article by

```php
//test the non default connection
DB::connection("mysql2")->getPdo();

//use with schemabuilder
$users = DB::connection('mysql2')->select(...);

//use in migrations
Schema::connection('mysql2')->create('some_table', function($table)
{
    $table->increments('id'):
});

//set and use on a eloquent model
$someModel = new SomeModel;
$someModel->setConnection('mysql2');
$something = $someModel->find(1);

//configure a eloquent model to override the default connection
class SomeModel extends Eloquent
{
    protected $connection = 'mysql2';
}
```
