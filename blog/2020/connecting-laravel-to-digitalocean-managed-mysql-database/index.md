# Connecting Laravel To Digitalocean Managed MySql Database

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Connecting Laravel To Digitalocean Managed MySql Database](https://aregsar.com/blog/2020/connecting-laravel-to-digitalocean-managed-mysql-database)

## Changing MySql Connection password mode (for PHP 7.1 or earlier)

DigitalOcean Managed Databases using MySQL 8+ are automatically configured to use caching_sha2_password authentication by default. This is is not compatible with some versions of php. 

According to `https://www.digitalocean.com/docs/databases/mysql/resources/troubleshoot-connections/` you need to change this MySql configuration
 if your mysql client is PHP7.1 or earlier. So you should not need to do this for PHP7.4

According to the article if your applications are experiencing authentication issues, you can use the Password Encryption option in the control panel to set a user’s password encryption settings to MySQL 5.x encryption settings.

Another option is two create a MySql user with the old mysql_native_password configuration or alter an existing user to use mysql_native_password configuration.

## Using mysql-client cli

mysql-client is the longtime MySql client

from `https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage`

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

```bash
# install mysql-shell cli
sudo apt install mysql-shell
# connect to database and alter USER for php app that will connect
# to managed database
# use the --sql flag do indicate the language we will use
mysqlsh -u phpuser -p -h host -P port --sql
# for PHP to connect we need ti use mysql_native_password plugin
# phpuser is the user laravel\php-fpm runs under
# when creating a new user via cli
mysql> CREATE USER phpuser IDENTIFIED WITH mysql_native_password BY 'password';
```

If user and database already existed then we could connect to the database and alter the user password plugin.

```bash
mysqlsh -u phpuser -p -h host -P port -D database --sql
# altering an existing user
mysql> ALTER USER phpuser IDENTIFIED WITH mysql_native_password BY 'mysqlpassword';
exit
```

# Laravel Managed MySql connection settings

from `https://laracasts.com/discuss/channels/laravel/digital-ocean-managed-databases?page=0`

In .env add:

MYSQL_ATTR_SSL_CA="/full/path/to/ca-certifcate.crt" 
SSL_MODE=required

In config/database.php under mysql connection add:

'ssl_mode' => env('SSL_MODE'),

Then run:

php artisan config:cache

https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-encrypted-connections.html
mysql -h <hostname> -u <username>  -p –ssl-mode=DISABLED
mysql> status

https://stackoverflow.com/questions/53061182/mysql-connection-over-ssl-with-laravel

https://www.digitalocean.com/community/questions/ssl-client-key-certificate-for-managed-mysql-database



==============
https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster