# Installing PHP Extensions on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## PHP Extensions

PHP extensions are installed using pecl.

You can search for extensions

```bash
pecl search pdf
```

Install extensions

```bash
pecl install redis
pecl install xdebug
```

Upgrade an extension

```bash
pecl uninstall redis
pecl install redis
```

> If having issues installing an extension, add the `--force` option to the `pecl install` command

```bash
pecl install redis --force
```

You can run pecl list to see which extensions are installed:

```bash
pecl list
```

check if php lists it the extension

```bash
php -i | grep redis
#or
php -r "var_dump(extension_loaded('redis'));"
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
