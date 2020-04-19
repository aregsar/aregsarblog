# Connecting Laravel To Digitalocean Managed MySql Database

April 20, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Connecting Laravel To Digitalocean Managed MySql Database](https://aregsar.com/blog/2020/connecting-laravel-to-digitalocean-managed-mysql-database)

https://www.digitalocean.com/community/questions/how-to-setup-laravel-with-digitalocean-managed-redis-cluster

------------------

https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-encrypted-connections.html
mysql -h <hostname> -u <username>  -p –ssl-mode=DISABLED
mysql> status

https://stackoverflow.com/questions/53061182/mysql-connection-over-ssl-with-laravel

https://www.digitalocean.com/community/questions/ssl-client-key-certificate-for-managed-mysql-database
-------------------------

from https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-laravel-6-application-using-managed-databases-and-object-storage
# install mysql-client
sudo apt install mysql-client cli
mysql -u phpuser -p -h host -P port
# default status should be -ssl-mode=ENABLED
mysql> status
mysql> CREATE DATABASE myappdatabase;
mysql> CREATE USER 'phpuser'@'%' IDENTIFIED WITH mysql_native_password BY 'phpuserpassword';
mysql> GRANT ALL ON myappdatabase.* TO 'travellist-user'@'%';
mysql> exit

from https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04
# install mysql-shell cli
sudo apt install mysql-shell
# connect to database and alter USER for php laravel app that will connect
# to managed database
mysqlsh -u phpuser -p -h host -P port --sql
mysqlsh -u phpuser -p -h host -P port -D database --sql
# for PHP to connect we need ti use mysql_native_password plugin
# phpuser is the user laravel\php-fpm runs under
# when creating a new user via cli
CREATE USER phpuser IDENTIFIED WITH mysql_native_password BY 'password';
# altering an existing user
ALTER USER phpuser IDENTIFIED WITH mysql_native_password BY 'password';
exit






# Laravel Managed MySql connection settings

In .env add:

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


