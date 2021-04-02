# Installing PHP Extensions on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

PHP extensions are installed using the PECL cli which is installed when we install PHP.

We have to install extensions once for each version of PHP we have installed.

This means that we also have to install extensions again every time we upgrade the installed PHP version to the latest version.

## Preparing to install extensions

```bash
# update the pecl repo
pecl channel-update pecl.php.net
# clear cached extensions
pecl clear-cache
```

## Searching for PHP Extensions

We can search the PECL repository for available PHP extensions using fuzzy search:

For example we can search for the redis extension:

```bash
pecl search redis
```

The result is shown below:

```bash
redis   5.3.4 (stable)  5.3.1 PHP extension for interfacing with Redis
```

## Installing extensions

In all examples I will use the php redis extension which is a very popular extension.

We can Install extensions:

```bash
pecl install redis
```

In some scenarios we may need to use the --force flag. See the troubleshooting section:

```bash
pecl install --force redis
```

## Upgrading the extension

We can upgrade an extension by uninstalling and reinstalling the extension when a new version is out:

```bash
pecl uninstall redis
pecl install redis
```

Alternatively we can use the upgrade command:

```bash
pecl upgrade redis
```

## Displaying installed extensions

You can run `pecl list` to see which extensions are installed:

```bash
pecl list
```

The output may look something like this if we have the xdebug and redis extensions installed:

```bash
Package Version State
redis   5.3.1   stable
xdebug  2.9.6   stable
```

We can check for a specific extension:

```bash
pecl list | grep redis
```

Now we will only see the redis extenssiom:

```bash
redis   5.3.1   stable
```

## Checking if PHP recognizes extensions

Even though PECL might list an extension as installed, php might not recognize it if it was not properly installed.

To make sure the extension is properly installed we need to also check if php can see the extension.

We can list all php extensions:

```bash
php -m
```

Or we can check if php lists it a specific extension such as redis

```bash
php -m | grep redis
```

Another way is to use raw php code to check for the extension:

```bash
php -r "var_dump(extension_loaded('redis'));"
```

The following is yet another way to check the extension is installed while also displaying its configuration settings:

```bash
php -i | grep redis
```

## Troubleshooting extensions

The following are sreps we can take to troubleshoot any extensions that are not listed by php or pecl after installation.

First as mentioned above try using the force flag with the install command and run the command again:

```bash
pecl install --force redis
```

After doing so check again for the extension with both pecl and php commands mentioned in previous sections.

If the extension is still not recognized, make sure the extensions are added in the correct php.ini file.

You can run php --ini to see which configuration files are loaded:

```bash
php --ini
```

The response is below:

```bash
Configuration File (php.ini) Path: /usr/local/etc/php/8.0
Loaded Configuration File:         /usr/local/etc/php/8.0/php.ini
Scan for additional .ini files in: /usr/local/etc/php/8.0/conf.d
Additional .ini files parsed:      /usr/local/etc/php/8.0/conf.d/ext-opcache.ini
```

Open the php.ini file where you should see the installed extensions at the top of the file.

Below I show the redis and xdebug extension files in the php.ini file.

```ini
extension="redis.so"
zend_extension="xdebug.so"
```

Some extensions like xdebug are listed as zend extensions but most are standard extension.

Sometimes the extensions are added into their own separate configuration file which is then included from the php.ini file. These additional files will usually be under the `conf.d` directory.

Manually add them if they are not anywhere in the php.ini file or in separate files in the conf.d directory.

Also make sure the installation directory is configured properly in the php.ini file for the php version the extension is added to by following instructions in the next section.

## The pecl installation directory

The directory where the php extensions are installed:

```bash
pecl config-get ext_dir
```

The result for php 8.0 installation is:

```bash
/usr/local/lib/php/pecl/20200930
```

The general form of the installation directory for each version of php is:

```bash
/usr/local/lib/php/pecl/<php_api_version>
```

The location is also specified by the extension_dir setting in the php.ini configuration file for each version of php.

Make sure this setting exists and is uncommented if you run into issues having extensions recognized by php.

Below is extension_dir setting in `/usr/local/etc/php/8.0/php.ini` configuration file for php 8.0.

```ini
extension_dir = /usr/local/lib/php/pecl/20200930
```

If we cd into the extensions directory we can see the two redis and xdebug extension files:

```bash
ls -l /usr/local/lib/php/pecl/20200930
```

The result:

```bash
redis.so
xdebug.so
```
