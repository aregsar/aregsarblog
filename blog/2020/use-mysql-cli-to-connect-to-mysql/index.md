# Use MySQL CLI To Connect To MySQL

April 30, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[Use MySQL CLI To Connect To MySQL](https://aregsar.com/blog/2020/use-mysql-cli-to-connect-to-mysql)


brew install mysql

echo 'export PATH=/usr/local/mysql/bin:$PATH' >> ~/.bash_profile

=======================

#must have no space between -p and PASSWORD

mysql -h 127.0.0.1 -P 3306 -u myname -pmypassword mydb
mysql -h localhost -P 3306 -u myname -pmypassword mydb
mysql -h localhost -P 3306 -u myname -pmypassword
mysql -h localhost -P 3306 -u root -p

https://scalegrid.io/blog/configuring-and-managing-ssl-on-your-mysql-server/
By default, MySQL server always installs and enables SSL configuration. 
Clients can choose to connect with or without SSL as the server allows both types of connections. 
mysql -h hostnameorip -u root -p –ssl-mode=ENABLED
mysql -h hostnameorip -u root -p –ssl-mode=DISABLED

=====================

> status;
> show databases;
> use mydb;


# is PRIVILEGES needed
# is IDENTIFIED BY 'password' needed. If not added then no login password will be required
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON *.* TO 'username'@'localhost';
GRANT SELECT ON *.* TO 'username'@'localhost';



CREATE DATABASE dbname;
use dbname;
DROP DATABASE dbname;
