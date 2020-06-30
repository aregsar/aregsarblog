# Use MySQL CLI To Connect To MySQL

May 5, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## install the mysql CLI

The standard command line interface to connect to MySQL server.

```bash
brew install mysql
echo 'export PATH=/usr/local/mysql/bin:$PATH' >> ~/.bash_profile
```

## install the mysqlsh CLI

An alternative command line interface to connect to MySQL server.

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-8.0.20-macos10.15-x86-64bit.tar.gz
tar xvf mysql-shell-8.0.20-macos10.15-x86-64bit.tar.gz
ln -s ~/mysql-shell-8.0.20-macos10.15-x86-64bit/bin/mysqlsh /usr/local/bin/
```

## Connect to a mysql server using mysql CLI

Below are commands to connect to MySQL server with various combinations of command line arguments:

```bash
#must have no space between -p and PASSWORD
mysql -h localhost -P 3306 -u myname -pmypassword -D mydb
mysql -h localhost -P 3306 -u myname -pmypassword mydb
mysql -h localhost -P 3306 -u myname -pmypassword
#defaults to no password no selected database
mysql -h localhost -P 3306 -u root -p
#defaults to root@localhost:3306 no password no selected database
mysql -u root
#defaults to $(whoami)@localhost:3306 no password no selected database
mysql
```

> Note: 127.0.0.1 can be substituted for localhost

### SSL connection

By default, MySQL server always installs and enables SSL configuration:

[configuring-and-managing-ssl-on-your-mysql-server](https://scalegrid.io/blog/configuring-and-managing-ssl-on-your-mysql-server)

Clients can choose to connect with or without SSL as the server allows both types of connections.

```bash
mysql -h hostnameorip -u root -p –ssl-mode=ENABLED
mysql -h hostnameorip -u root -p –ssl-mode=DISABLED
```

## Connect to a mysql server using mysqlsh CLI

```bash
mysqlsh --sql -h 127.0.0.1 -P 3306
mysqlsh --sql -h 127.0.0.1 -P 3306 -u root
# requires -D to use database
mysqlsh --sql -h 127.0.0.1 -P 3306 -u myname -pmypassword -D mydb
```

> Note: mysqlsh has some issues creating a user on my macOS installation. It is primarily used a client to connect to managed Digitalocean instance.

## Database information

Once connected to the MySQL server we can issue the following basic commands:

Show connection stats (this command only works with mysql CLI):

```bash
mysql> status
```

Show all databases:

```bash
mysql> show databases;
```

Show tables in a database:

```bash
mysql> use appdb;show tables;
```

Select a database

```bash
mysql> use appdb
```

Show tables in the selected database:

```bash
mysql> show tables;
```

## Create a User

Additional commands for creating a database user:

```bash
#create a user on any IP with no password
mysql> CREATE USER 'appuser'@'%';
#create a user on any IP with password mypassword
mysql> CREATE USER 'appuser'@'%' IDENTIFIED BY 'mypassword';
#create a user on localhost with password mypassword
mysql> CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'mypassword';
```

## Creating a database

Commands for creating a database and granting access permissions for a database to a user:

```bash
mysql> CREATE DATABASE appdb;
mysql> GRANT ALL ON appdb.* TO 'appuser'@'%';
mysql> GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
mysql> GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'localhost';
```

Dropping the Database:

```bash
mysql> DROP DATABASE appdb;
```

## Reference links

[how-to-connect-to-managed-database-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-connect-to-managed-database-ubuntu-18-04)

[how-to-connect-to-mysql](https://www.digitalocean.com/docs/databases/mysql/how-to/connect)

[mysql-shell-install](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install-starting.html)