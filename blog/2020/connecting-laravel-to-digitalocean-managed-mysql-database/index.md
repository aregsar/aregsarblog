# Connecting Laravel To Digitalocean Managed MySql Database

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Connecting Laravel To Digitalocean Managed MySql Database](https://aregsar.com/blog/2020/connecting-laravel-to-digitalocean-managed-mysql-database)

## Changing MySql Connection password mode (for PHP 7.1 or earlier)

DigitalOcean Managed Databases using MySQL 8+ are automatically configured to use caching_sha2_password authentication by default. This is is not compatible with some versions of php.

According to `https://www.digitalocean.com/docs/databases/mysql/resources/troubleshoot-connections/` you need to change this MySql configuration
 if your mysql client is PHP7.1 or earlier. So you should not need to do this for PHP7.4.

According to the article if your applications are experiencing authentication issues, you can use the Password Encryption option in the control panel to set a user’s password encryption settings to MySQL 5.x encryption settings.

Instead of using the control panel, another option is two create a MySql user with the old mysql_native_password configuration or alter an existing user to use mysql_native_password configuration.

Here is the procedure to create a new User mysql_native_password:

```bash
mysql -u phpuser -p -h host -P port
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'user_password';
mysql> GRANT ALL ON myappdatabase.* TO 'phpuser'@'%';
mysql> exit
```

Here us the procedure to alter an existing User to use mysql_native_password:

```bash
mysql -u phpuser -p -h host -P port
mysql> ALTER USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'user_password';
exit
```

Here is and example of creating a user with the default caching_sha2_password configuration:

```bash
mysql -u phpuser -p -h host -P port
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED BY 'user_password';
mysql> GRANT ALL ON myappdatabase.* TO 'phpuser'@'%';
mysql> exit
```

> Note: the `%` means user can connect from any host

More info on MySql user accounts and access:

`https://linuxize.com/post/how-to-create-mysql-user-accounts-and-grant-privileges/`

## Using mysql-client cli

mysql-client is the longtime MySql client

from `https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage`

Here is an example of installing mysql-client cli, connecting to MySql, creating a new database and user and granting all permissions on the database to the new User. 

```bash
# install mysql-client
sudo apt install mysql-client cli
mysql -u phpuser -p -h host -P port
# default status should be -ssl-mode=ENABLED
mysql> status
mysql> CREATE DATABASE myappdatabase;
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'mysqlpassword';
mysql> GRANT ALL ON myappdatabase.* TO 'travellist-user'@'%';
mysql> exit
```



## Using mysql-shell cli

mysql-shell is the superset of mysql-client and supports interacting with MySql in SQL, Python and Javascript

from `https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04`

Here is an example of installing mysql-shell cli, connecting to MySql in SQL language mode, creating a new database and user and granting all permissions on the database to the new User.

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
mysql> CREATE USER phpuser IDENTIFIED WITH mysql_native_password BY 'password';
mysql> GRANT ALL ON myappdatabase.* TO phpuser;
exit
```

## Connecting to MySql with SSL mode

By default the MySql clients connect with MySQL in SSL mode because the Managed MySQL is configured with SSL enabled setup.

In order to disable that we can turn off SSL mode when connecting:

```bash
mysql -h <hostname> -u <username>  -p –ssl-mode=DISABLED
mysql> status
```

You can see checking the status that SSL is turned off.

The following links provide more details:

`https://dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html#using-encrypted-connections-client-side-configuration`

`https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-encrypted-connections.html`

`https://www.digitalocean.com/community/questions/ssl-client-key-certificate-for-managed-mysql-database`

`https://dev.mysql.com/doc/refman/8.0/en/windows-and-ssh.html`

# Laravel Managed MySql SSL connection settings

In order to enable Laravel to use SSL mode when connecting to MySQl we can configure it as shown.

In `.env` file add:

```ini
SSL_MODE=required
MYSQL_ATTR_SSL_CA="/full/path/to/ca-certifcate.crt"
```

In `config/database.php` under mysql connection add:

```php
'ssl_mode' => env('SSL_MODE'),
```

Then run:

```bash
php artisan config:cache
```

> Note when running in production we don't need to configure ssl mode to connect to the Managed MySql instance.

`https://laracasts.com/discuss/channels/laravel/digital-ocean-managed-databases?page=0`

https://stackoverflow.com/questions/53061182/mysql-connection-over-ssl-with-laravel

==============
https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster