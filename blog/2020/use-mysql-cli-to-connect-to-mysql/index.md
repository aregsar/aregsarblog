# Use MySQL CLI To Connect To MySQL

April 30, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Use MySQL CLI To Connect To MySQL](https://aregsar.com/blog/2020/use-mysql-cli-to-connect-to-mysql)

https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04

https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-encrypted-connections.html

## install the MySQL CLI

brew install mysql

echo 'export PATH=/usr/local/mysql/bin:$PATH' >> ~/.bash_profile

## Connect to a mysql server

#must have no space between -p and PASSWORD

mysql -h 127.0.0.1 -P 3306 -u myname -pmypassword mydb
mysql -h localhost -P 3306 -u myname -pmypassword mydb
mysql -h localhost -P 3306 -u myname -pmypassword
mysql -h localhost -P 3306 -u root -p
#root@localhost
mysql -u root
#homeuser@localhost
mysql 

https://scalegrid.io/blog/configuring-and-managing-ssl-on-your-mysql-server/
By default, MySQL server always installs and enables SSL configuration. 
Clients can choose to connect with or without SSL as the server allows both types of connections. 
mysql -h hostnameorip -u root -p –ssl-mode=ENABLED
mysql -h hostnameorip -u root -p –ssl-mode=DISABLED

=====================

## Database information

Show connection stats:

mysql> status;

Show all databases:

mysql> show databases;

Show tables in a database:

mysql> use appdb;show tables;

Select a database

mysql> use appdb;

Show tables in the selected database:

mysql> show tables;

## Create a User

mysql> CREATE USER 'appuser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
mysql> CREATE USER 'appuser'@'%' IDENTIFIED BY 'password';
mysql> CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'password';

## Create a database

mysql> CREATE DATABASE appdb;

## Grant access permissions for a database to a user

mysql> GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
mysql> GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'localhost';
mysql> GRANT ALL ON appdb.* TO 'appuser'@'%';
mysql> GRANT ALL ON *.* TO 'appuser'@'localhost';

####is IDENTIFIED BY 'password' needed. If not added then no login password will be required
####mysql> GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password';


## Droping a Database

mysql> DROP DATABASE appdb;

## ending the sesssion

mysql> exit;

or Ctrl+c
