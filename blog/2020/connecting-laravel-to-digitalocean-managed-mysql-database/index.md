# Connecting Laravel To Digitalocean Managed MySql Database

April 9, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Connecting Laravel To Digitalocean Managed MySql Database](https://aregsar.com/blog/2020/connecting-laravel-to-digitalocean-managed-mysql-database)


# Laravel Managed MySql connection settings


In my .env add:

MYSQL_ATTR_SSL_CA="/full/path/to/ca-certifcate.crt" 
SSL_MODE=required


In config/database.php under mysql connection add:

'ssl_mode' => env('SSL_MODE'),

Then run:

php artisan config:cache


## Changing MySql Connection password mode

Install instructions MySQL Secure Shell to connect using the mysqlsh cli:

https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install-osx-quick.html

DigitalOcean Managed Databases using MySQL 8+ are automatically configured to use caching_sha2_password authentication by default

Connect using mysqlsh then run the MySql command:
ALTER USER use_your_user IDENTIFIED WITH mysql_native_password BY 'use_your_password';

> Note: according to `https://www.digitalocean.com/docs/databases/mysql/resources/troubleshoot-connections/` you only need to run this command if your mysql client is PHP7.1 or earlier.
So you should not need to do this for PHP7.4

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage


