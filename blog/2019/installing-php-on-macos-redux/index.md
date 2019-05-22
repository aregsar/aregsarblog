# Installing PHP on macOS Redux

May 20, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Introduction

This post is an abbreviated form of my previous post on installing PHP on a Mac using the Homebrew package manager.

In that post I went into great detail about installation directories and installing multiple versions of PHP. This made that post a more cumbersome read for someone that simply wants to install the latest version of PHP.

So in this post I will detail the minimal steps necessary to install the latest version of PHP on your Mac using Homebrew package manager for the OSX.

For a more detailed understanding of the impact of the installation, check out [Installing PHP on macOS](https://aregsar.com/blog/2019/installing-php-on-macos). Parts of that post are repeated here.

## Installing Homebrew

Before we install PHP we need to install the Homebrew package manager, hereafter referenced simply as brew.

The installations for installing brew are located at [https://brew.sh/](https://brew.sh/).

Gathered from the docs, you can run the following bash statement to install brew:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

This bash script downloads and runs a Ruby language script using the default Ruby installation on your Mac.

## Updating Homebrew

If you already have brew installed make sure to update it, by running `brew update`, before installing PHP.

> Note: Be careful to not run __brew upgrade__ instead, which updates all the packages that brew has installed on your system to their latest version.

Run `brew doctor` afterwards to make sure brew does not see any issues with the current installed packages.

## Removing all installed versions of PHP

To do a fresh install, we first need to remove any existing brew PHP installations.

The following is the bash commands I use to clean out my PHP installation directories:

```bash
brew list | grep php | xargs brew uninstall --force
rm -rf /usr/local/lib/php
rm -rf /usr/local/etc/php
rm ~/Library/LaunchAgents/homebrew.mxcl.php*
sudo rm /Library/LaunchDaemons/homebrew.mxcl.php*
brew untap homebrew/php
brew cleanup -s
brew untap homebrew/homebrew-dupes
```

The first line in the above bash statements goes through all installed versions and removes them all.

> Note: some of the commands above may not be required, but running them all will ensure a clean uninstall. You can safely ignore any file or directory does not exist errors.

## Installing the latest PHP version

We can install the latest PHP version using the command:

`brew install php`

After running this command, the latest versions of PHP and PHP-FPM (7.3 as of this time) will be installed. Also PECL the PHP extension installation CLI will be installed.

You can run the following command to verify the current PHP version.

`php -v`

Run the following to see all information and configuration settings for the installed version:

`php -i`

Run the following to only list the installed PHP extensions:

`php -m`

Run the following command to see the symlink created that points to the installed PHP CLI binary:

`which php`

You can show all installed versions of PHP by running:

`brew list | grep php`

## Adding the binary file symlinks to the PATH variable

To be able to run commands globally we need to add the `/usr/local/bin` and `/usr/local/sbin` symlink directories to our PATH environment variable. 

For example, my PATH variable starts with:

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin`

These directories contain symlinks to the installed `php`, `pecl` and `php-fpm`  binaries.

## PHP ini and configuration files

The brew installation puts the configuration files for installed PHP versions in the directory:

`/usr/local/etc/php/<version>/`

So the brew installation for the latest PHP version on my machine put all configuration files for the PHP CLI and PHP-FPM in:

`/usr/local/etc/php/7.3/`

For instance the php.ini file is installed at:

`/usr/local/etc/php/7.3/php.ini`

and the php-fpm.conf file is installed at:

`/usr/local/etc/php/7.3/php-fpm.conf`

To show the location of the php.ini file you can run the following command:

`pear config-show | grep php.ini`

The above command runs the pear utility that was also installed when we installed PHP.

> Each installed version of PHP will have its own configuration file version directory under `/usr/local/etc/php/`.

## Installing PHP extensions using PECL

The PECL CLI installed by brew can be used to install specific PHP extensions that we may require for our projects.

Installing an extension involves adding an `<extension>.so` file to the extensions directory and,  depending on the extension type, adding either a `zend_extension="<zend-extension>.so"` or a `extension="<extension>.so"` setting to the `php.ini` configuration file.

The `PECL install` command that installs the PHP extensions will add the extension file and the setting for us.

We can install PHP extensions for the current active version of PHP, by running the following command (without the .so extension):

`pecl install --force <extension>`

For instance we can install the xdebug extension by typing:

`pecl install --force xdebug`

> Note: We need to use the --force flag to avoid an issue where PECL install fails to install a non existing extension because of a cached setting. This can happen if you had previously installed the extension. The other option, in this situation, would be to run `pecl uninstall` before running `pecl install` without the --force flag.

To install a specific version of an extension we can specify the extension version, with a dash separator, like the example below:

`pecl install --force xdebug-2.7.1`.

To uninstall an extension run:

`pecl uninstall <extension>`

For instance we can uninstall the xdebug extension by typing:

`pecl uninstall xdebug`

We can list the PECL installed PHP extensions by running:

`php -m`

> Note: Do not rely on `pecl list` to list the installed extensions.

Each installed PHP version will add a PHP release date directory to the base extension directory `/usr/local/lib/php/pecl/` under which the extensions will be added.

The `php.ini` configuration has a setting that specifies the PHP extensions installation directory.

Here is an example of the extensions directory setting from php.ini for my PHP 7.3 installation:

`extension_dir = "/usr/local/lib/php/pecl/20180731"`

So for example, after we run `pecl install xdebug` we can verify that the `xdebug` extension file is installed at:

`/usr/local/lib/php/pecl/20180731/xdebug.so`

You can run the following command to show the value of PHP extension directory setting from the `php.ini` file:

`pecl config-get ext_dir`

To print out all PECL configuration settings run the following:

`pecl config-show`

> The special Opcache extension is installed at `/usr/local/Cellar/php/7.3.5/lib/php/20180731/opcache.so` when PHP is installed.

## Running the php-fpm service using brew

We can check to see all the versions of php-fpm installed on our machine by running:

`brew services list | grep php`

We can use brew to configure the latest version of php-fpm to auto-start by running the brew command:

`sudo brew services start php`

Now when we run `brew services list | grep php` we should see:

```bash
php       started root           /Library/LaunchDaemons/homebrew.mxcl.php.plist
```

We can stop the service by running `sudo brew services stop php`

Running `brew services list | grep php` again we should see:

```bash
php       stopped
```  

## Resolving PHP 7.3 Composer package manager error

PHP uses a package manager named `Composer` that is a separate install from PHP itself.

If you have the Composer PHP package manager installed, you need to deactivate the PHP v7.3 JIT compiler setting `pcre.jit` so that the Composer commands do not fail with the JIT compiler setting error: `preg_match(): JIT compilation failed: no more memory`.

The JIT setting is in the `/usr/local/etc/php/7.3/php.ini` file.

You can run the following to see the commented out `;pcre.jit=1` setting:

```bash
cat /usr/local/etc/php/7.3/php.ini | grep 'pcre.jit'
```

Run the following command to uncomment the setting and change its value:

```bash
sed -i '' 's/;pcre.jit=1/pcre.jit=0/g' /usr/local/etc/php/7.3/php.ini
```

## Overview of installation directories

After running `brew install php` to install the latest version of PHP, we should have the following installation directories:

PHP binaries:

`/usr/local/Cellar/php/7.3.5/bin/php`

`/usr/local/Cellar/php/7.3.5/bin/pecl`

`/usr/local/Cellar/php/7.3.5/sbin/php-fpm`

Symlinks to the PHP binaries:

`/usr/local/bin/php`

`/usr/local/bin/pecl`

`/usr/local/sbin/php-fpm`

PHP configuration files:

`/usr/local/etc/php/7.3/php.ini`

`/usr/local/etc/php/7.3/php-fpm.conf`

PHP extension files location:

`/usr/local/lib/php/pecl/20180731/`

Opcache extension location:

`/usr/local/Cellar/php/7.3.5/lib/php/20180731/opcache.so`

## Conclusion

In this article I listed the steps to take to install the Homebrew package manager for the macOS followed by the steps to install PHP using Homebrew.

In addition I detailed how to install PHP extensions and how the php-fpm FastCGI process manager is activated.

Finally I detailed the installation locations for PHP binaries and configuration files.

Thanks for reading.
