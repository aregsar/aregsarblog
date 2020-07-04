# Connecting Laravel To Digitalocean Managed MySql Database

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

This post is meant as a quick list reference of links and commands for connecting to the DigitalOcean managed MySQL database.

## Changing MySql Connection password mode (for PHP 7.1 or earlier)

DigitalOcean Managed Databases using MySQL 8+ are automatically configured to use `caching_sha2_password` authentication by default. This is not compatible with some versions of PHP.

According to [troubleshoot-digitalocean-MySQL-connections](https://www.digitalocean.com/docs/databases/mysql/resources/troubleshoot-connections) If your mysql client is PHP 7.1 or earlier the default authentication configuration needs to be changed.

If your applications are experiencing authentication issues with PHP 7.1 or earlier, you can use the Password Encryption option in the MySQL control panel to set the user password encryption settings to MySQL 5.x encryption.

Instead of using the control panel, another option is two create a MySql user with the `mysql_native_password` configuration or alter an existing user to use `mysql_native_password` configuration.

Below is the procedure to create a new User `mysql_native_password`:

```bash
# connect to mysql instance
mysql -u phpuser -p -h host -P port
# create a user with mysql_native_password authentication
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'user_password';
# grant privileges to the new user
mysql> GRANT ALL PRIVILEGES ON myappdatabase.* TO 'phpuser'@'%';
mysql> exit
```

Belwo is the procedure to alter an existing User to use `mysql_native_password`:

```bash
# connect to mysql instance
mysql -u phpuser -p -h host -P port
# alter user to use mysql_native_password authentication
mysql> ALTER USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'user_password';
exit
```

Below is an example of creating a user with the default `caching_sha2_password` configuration:

```bash
# connect to mysql instance
mysql -u phpuser -p -h host -P port
# create a user with the default  authentication (caching_sha2_password)
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED BY 'user_password';
# grant privileges to the new user
mysql> GRANT ALL PRIVILEGES ON myappdatabase.* TO 'phpuser'@'%';
mysql> exit
```

> Note: the `%` in 'phpuser'@'%' means the user can connect from any host and `myappdatabase.*` means that we are granting privileges to all the tables in the myappdatabase database.

More info on MySql user accounts and access can be found here:

[how-to-create-mysql-user-accounts-and-grant-privileges](https://linuxize.com/post/how-to-create-mysql-user-accounts-and-grant-privileges)

## Using mysql-client cli

`mysql-client` is the standard MySql client.

Here is an example of installing the `mysql-client` CLI, connecting to a MySql instance, creating a new database and user and granting all permissions on the database to the new User.

```bash
# install mysql-client
sudo apt install mysql-client cli
mysql -u phpuser -p -h host -P port
# default status should be -ssl-mode=ENABLED
mysql> status
mysql> CREATE DATABASE myappdatabase;
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'mysqlpassword';
mysql> GRANT ALL PRIVILEGES ON myappdatabase.* TO 'phpuser'@'%';
mysql> exit
```

Instructions for using the mysql-client are listed at [how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage)

## Using mysql-shell cli

`mysql-shell` commands are the superset of `mysql-client` commands and support interacting with MySql in SQL, Python and Javascript.

Here is an example of installing `mysql-shell` CLI, connecting to MySql in SQL language mode, creating a new database and user and granting all permissions on the database to the new User.

```bash
# install mysql-shell cli
sudo apt install mysql-shell
# connect to database and alter USER for php app that will connect
# to managed database
# use the --sql flag do indicate the language we will use
mysqlsh -u phpuser -p -h host -P port --sql
mysql> status
mysql> CREATE DATABASE myappdatabase;
# for PHP to connect we need ti use mysql_native_password plugin
# phpuser is the user laravel\php-fpm runs under
# when creating a new user via cli
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'mysqlpassword';
mysql> GRANT ALL PRIVILEGES ON myappdatabase.* TO 'phpuser'@'%';
exit
```

Instructions for using the mysql-shell are listed at 
from [how-to-connect-to-managed-database-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04)

## Connecting to MySql with SSL mode

By default the MySql clients connect with MySQL in SSL mode because the Managed MySQL is configured with SSL enabled setup:

```bash
# connect
mysql -u phpuser -p -h host
# check the status
mysql> status
```

In order to disable that we can turn off SSL mode when connecting:

```bash
# connect
mysql -u phpuser -p -h host –ssl-mode=disabled
# check the status
mysql> status
```

> Note: trying to connect with –ssl-mode=disabled may be rejected by the managed MySQL server

The following links provide more details:

[ssl-client-key-certificate-for-managed-mysql-database](https://www.digitalocean.com/community/questions/ssl-client-key-certificate-for-managed-mysql-database)

[how-to-configure-ssl-tls-for-mysql-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-configure-ssl-tls-for-mysql-on-ubuntu-18-04)

[using-encrypted-connections-client-side-configuration](https://dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html#using-encrypted-connections-client-side-configuration)

[mysql-shell-encrypted-connections](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-encrypted-connections.html)

[windows-and-ssh](https://dev.mysql.com/doc/refman/8.0/en/windows-and-ssh.html)

## Laravel Managed MySql SSL connection settings

> Note: This setup is for running Laravel locally. When running in production we don't need to configure ssl mode to connect to the Managed MySql instance

In order to enable Laravel to use SSL mode when connecting to MySQl we need to perdorm the following:

First we add the following we can settings to the  `.env` file:

```ini
SSL_MODE=required
MYSQL_ATTR_SSL_CA="/full/path/to/ca-certifcate.crt"
```

> Note: `ca-certifcate.crt` can be downloaded from the Managed database configuration. see [ssl-client-key-certificate-for-managed-mysql-database](https://www.digitalocean.com/community/questions/ssl-client-key-certificate-for-managed-mysql-database)

Then we add the 'ssl_mode' configuration to the `config/database.php` file, within the `mysql` configuration block:

```php
'ssl_mode' => env('SSL_MODE'),
```

To cache the new configuration run:

```bash
php artisan config:cache
```

More info about the Laravel MySql SSL settings:

From [digital-ocean-managed-databases](https://laracasts.com/discuss/channels/laravel/digital-ocean-managed-databases)

[mysql-connection-over-ssl-with-laravel](https://stackoverflow.com/questions/53061182/mysql-connection-over-ssl-with-laravel)

## Restricting access to managed MySQL database

While SSL secures the data transmitted to and from the MySQL database, it does not restrict who can connect to the database.

The Managed MySQL database allows restricting network access based on client IP Address and VPC (Virtual Private Cloud) settings that can be configured via the DigitalOcean console or API.
