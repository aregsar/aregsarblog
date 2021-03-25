# Installing PHP Extensions on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## PHP Extensions

PHP extensions are installed using the PECL cli which is installed when we installed PHP.
We have to install these extensions once for each version of PHP every time we update the installed PHP version.

We can search for specific extensions such as the redis extension shown below:

```bash
pecl search redis
```

The result looks like:

```bash
redis   5.3.4 (stable)  5.3.1 PHP extension for interfacing with Redis
```

We can Install extensions:

```bash
pecl install redis
pecl install xdebug
```

We can upgrade an extension

```bash
pecl uninstall redis
pecl install redis
```

> If having issues installing or updating an extension, add the `--force` option to the `pecl install` command

```bash
pecl install --force redis
```

You can run `pecl list` to see which extensions are installed:

```bash
pecl list
```

The reult looks like:

```bash
Package Version State
redis   5.3.1   stable
xdebug  2.9.6   stable
```

We can check if php lists it the extension

```bash
php -m
#or
php -r "var_dump(extension_loaded('redis'));"
```

check the extension configuration settings:

```bash
php -i | grep redis
```

If extensions aren't properly loaded, there are two easy fixes.

First, make sure the extensions are added in the correct ini file. You can run php --ini to know which file is loaded:

```bash
php --ini
```

The response is below:

```bash
Configuration File (php.ini) Path: /usr/local/etc/php/7.4</hljs>
Loaded Configuration File: /usr/local/etc/php/7.4/php.ini
Scan for additional .ini files in: /usr/local/etc/php/7.4/conf.d
Additional .ini files parsed: /usr/local/etc/php/7.4/conf.d/ext-opcache.ini,
/usr/local/etc/php/7.4/conf.d/php-memory-limits.ini
```

Open the php.ini file where you should see the installed extensions at the top of the file:

```ini
extension="redis.so"
zend_extension="xdebug.so"
```
