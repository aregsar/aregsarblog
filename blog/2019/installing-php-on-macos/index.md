# Installing PHP on macOS

May 13, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Introduction

In this post I will detail the installation steps to install PHP on a Mac using the Homebrew package manager.

## Installing Homebrew

Before we install PHP we need to install the Homebrew package manager, here after referenced simply as brew.

The installations for installing brew are located at [https://brew.sh/](https://brew.sh/).

Gathered from the docs, you can run the following bash statement to install brew:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

This bash script downloads and runs a Ruby language script using the default Ruby installation on your Mac.

## Updating Homebrew

If you already have brew installed, make sure to update it before installing PHP by running:

`brew update`

> Note: Be careful to not run __brew upgrade__ instead, which updates all the packages that brew has installed on your system.

Run `brew doctor` afterwards to make sure brew does not see any issues with current installed packages.

## Upgrading to the latest PHP Homebrew installation

If you already have a PHP version installed by brew, you can run `brew upgrade php` to upgrade the latest version of PHP installed on your machine to the latest released version of PHP.

> Note: We did not specify the php version to upgrade which defaults to upgrade the latest installed version on our machine.

> Note: Even if you are on the latest minor release of PHP, running `brew upgrade php` will ensure that all the latest patches are applied.

## Upgrading a specific PHP version Homebrew installation

If you have multiple PHP versions installed by brew, you can upgrade the older versions with the latest patches by running `brew upgrate php@<version>`.

When installing older versions of software, brew uses the version number to name the installation directories so we have to use the version number in the command to tell brew which version we want to upgrade.

So for instance if you have the latest PHP version 7.3 installed and you also have PHP version 7.2 installed, you can run `brew upgrate php` to upgrade the PHP version 7.3 and run `brew upgrade php@7.2` to upgrade the older version 7.2.

## Removing all brew installed versions of PHP

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

Now install the latest PHP version using the command:

`brew install php`

After running this command latest version of PHP (7.3 as of this time) will be installed.

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

On my machine I see two results the first result is for the latest PHP v7.3 and the other is obviously PHP v7.2:

```bash
php #latest version 7.3
php@7.2
```

## The installation directories

When we run the `brew install php` command, the latest version of PHP is installed with files placed under the base path of:

`/usr/local/Cellar/php/7.3.5`

> Note: If the latest version of PHP is v7.3, then running `brew install php` or `brew install php@7.3` will result in the same installation directory. That is, the base directory would still be `/usr/local/Cellar/php/7.3.5/`. So both commands are effectively the same.
However if the latest version is v7.3 and we install an older version of PHP, say for example `brew install php@7.2`, then the base installation directory will be `/usr/local/Cellar/php@7.2/7.2.18/` which as you can see includes the `@7.2` version and the PHP 7.2 release version number in the path.

Brew installs the latest PHP binaries under the bin sub directory:

`/usr/local/Cellar/php/7.3.5/bin`

So the `php` CLI is located at `/usr/local/Cellar/php/7.3.5/bin/php` and the `pecl` CLI for installing PHP extensions is located at `/usr/local/Cellar/php/7.3.5/bin/pecl`

Similarly brew installs the latest system binaries for PHP at:

`/usr/local/Cellar/php/7.3.5/sbin`

Therefore the `php-fpm` server is installed at `/usr/local/Cellar/php/7.3.5/sbin/php-fpm`

Brew also adds symlinks that point to the installed binary files in the `/usr/local/bin/` and `/usr/local/sbin` directories.

For instance it adds the symlink `/usr/local/bin/php` that points to the `/usr/local/Cellar/php/7.3.5/bin/php` binary.

## Adding the binary file symlinks to the PATH variable

To be able to run the PHP and PECL CLI commands globally we need to add the `/usr/local/bin` and `/usr/local/sbin` symlink directories to our PATH environment variable.

For example, to be able to globally run the php, pecl and php-fpm commands, my PATH variable starts with:

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin`

## Symlinks to the installation directory

As mentioned before, when installing the latest version of PHP using the `brew install php` command the brew installation will create symlinks that reference the PHP binaries under the `/usr/local/Cellar/php/7.3.5/bin/` installation directory.

Here is a full list of the symlinks:

```bash
/usr/local/sbin/php-fpm -> /usr/local/Cellar/php/7.3.5/sbin/php-fpm

/usr/local/bin/pecl -> /usr/local/Cellar/php/7.3.5/bin/pecl
/usr/local/bin/pear -> /usr/local/Cellar/php/7.3.5/bin/pear
/usr/local/bin/peardev -> /usr/local/Cellar/php/7.3.5/bin/peardev
/usr/local/bin/phar -> /usr/local/Cellar/php/7.3.5/bin/phar
/usr/local/bin/phar.phar -> /usr/local/Cellar/php/7.3.5/bin/phar.phar
/usr/local/bin/php -> /usr/local/Cellar/php/7.3.5/bin/php
/usr/local/bin/php-cgi -> /usr/local/Cellar/php/7.3.5/bin/php-cgi
/usr/local/bin/php-config -> /usr/local/Cellar/php/7.3.5/bin/php-config
/usr/local/bin/hpdbg -> /usr/local/Cellar/php/7.3.5/bin/phpdbg
/usr/local/bin/phpize -> /usr/local/Cellar/php/7.3.5/bin/phpize
```

Brew will also create the following symlinks to the base installation directory:

`/usr/local/opt/php -> usr/local/Cellar/php/7.3.5`
`/usr/local/opt/php@7.3 -> usr/local/Cellar/php/7.3.5`

> Note that both `/usr/locl/opt/php` and `/usr/locl/opt/php@7.3` reference the same installation directory. The symlinks in the `/usr/local/opt/` directory will not be relevant to this article but in any case I mention them here for your information.

## PHP ini and configuration files

The brew installation puts the configuration files for installed PHP versions in the directory:

`/usr/local/etc/php/<version>/`

So the brew installation for the latest PHP version on my machine put all configuration files for the PHP CLI and PHP_FPM in:

`/usr/local/etc/php/7.3/`

So for instance the php.ini file is located at:

`/usr/local/etc/php/7.3/php.ini`

and the php-fpm.conf file at:

`/usr/local/etc/php/7.3/php-fpm.conf`

To show the location of the php.ini file you can run the following command:

`pear config-show | grep php.ini`

Each installed version of PHP will have its own configuration file version directory under `/usr/local/etc/php/`. So if we had installed PHP v7.2, then the php.ini file for that version would be at `/usr/local/etc/php/7.2/php.ini`.

## Installing PHP extensions using PECL

The PECL CLI installed by brew can be used to install specific PHP extensions that we may require for our projects.

We can install PHP extensions for the current active version of PHP, by running the command the following command:

`pecl install <extension>`

For instance we can install the xdebug extension by typing:

`pecl install xdebug`

You can also upgrade the installed version of an extension by running:

`pecl upgrade <extension-name>`

For instance `pecl upgrade xdebug` will upgrade the xdebug extension to the latest version.

We can list the PECL installed PHP extensions by running:

`php -m`

> Note: The `pecl list` command can also be used to list the installed extensions. However there is an issue with this command that I will detail later which means that the only reliable way to see if an installed extension is recognized by PHP is to run `php -m`.

The directory where the PHP extensions are installed depends on the current active PHP version. Each installed PHP version will add a PHP release date directory to the base extension directory `/usr/local/lib/php/pecl/` under which the extensions will be added.

Each installed PHP version has its own `php.ini` configuration file within which the PHP extensions directory for that version is specified. The setting in the `php.ini` file that specifies the extensions installation directory is:

`extension_dir = "/usr/local/lib/php/pecl/20180731"`

So after we run `pecl install xdebug` we can see verify that xdebug is installed at:

`/usr/local/lib/php/pecl/20180731/xdebug.so`

The path above is for PHP v7.3. If we have PHP v7.2 installed and we activate that version of PHP and run the `pecl install xdebug` command, then the extension would be installed at:

`/usr/local/lib/php/pecl/20170718/xdebug.so`

As you can see xdebug was installed under the release date directory for PHP v7.2.

You can run the following command to show the value of PHP extension directory setting from the `php.ini` file for the active PHP version, without having to open the php.ini file for that version:

`pecl config-get ext_dir`

To print out all PECL configuration settings run the following:

`pecl config-show`

> Note: It may also appear that the PHP 7.3 extensions are also installed in `/usr/local/Cellar/php/7.3.5/pecl/20180731/`, but that is because `/usr/local/Cellar/php/7.3.5/pecl/` is a symlink to `/usr/local/lib/php/pecl/`.

## How PHP detects PECL installed extensions

In order for PHP to detect PECL installed extensions, each installed extension must be declared in the php.ini file. The `pecl install` command will add the extension to the php.ini file for us.

If we look inside the php.ini file we will see that the xdebug setting is actually added as a special type of extension that is a zend extension:

`zend_extension="xdebug.so"`

Normally PHP extensions are installed as standard PHP extensions and will be referenced in the php.ini file as:

`extension="<php-extension>.so"`

So if we run `pecl install memcached` for example, the extension file will be installed at `/usr/local/lib/php/pecl/20180731/memcached.so` and the `extension="memcached.so"` setting will be added to the top of the php.ini file.

## How to resolve missing extension issue encountered with PECL

Sometimes even after a clean install of PHP, the `pecl list` command reports that an extension is installed when `php -m` does not detect the extension. In fact after checking the proper extensions directory for the active PHP version, the extension file is actually not there. Also looking in the php.ini file for the active version, the extension setting is not listed there either.

This means that `pecl list` is actually in error and probably has a cache somewhere that is not cleared. Attempting to actually install the extension using `pecl install` in this scenario fails and reports that the extension is already installed.

There are two ways to get around this issue. The easiest way is just to force pecl to install the extension using the `--force` flag. The other way is to first run `pecl uninstall` to clear the cache and then run `pecl install` to install the extension.

I ran into this issue when I had installed xdebug and memcached for the latest php version and then after doing a fresh install of PHP, `pecl list` would report both extensions as already existing where in fact the files were not there and php.ini settings were missing which resulted PHP not detecting them.

I force installed the extensions using `pecl install --force xdebug` and `pecl install --force memcached` which added the extension files and the php.ini settings and then PHP detected the extensions again.

The other way I could have resolved this issue, for example, would have been to run `pecl uninstall xdebug` followed by `pecl install xdebug` to re-install the extension.

> Note: It is good practice to run `php -m` after running `pecl install` to make sure the extension is detected by PHP. If PHP does not detect the extension, check the extension directory to make sure the extension file exists and check the `php.ini` file to make sure the setting for the extension is added to the top of the file.

## Running the php-fpm service using brew

We can check to see all the versions of php-fpm installed on our machine by running:

`brew services list | grep php`

On my machine I see:

```bash
php       stopped
php@7.2   stopped
```  

The first line is the php-fpm service for the latest PHP version.

We can use brew to configure the latest version of php-fpm to auto-start by running the brew command:

`sudo brew services start php`

Now when we run `brew services list | grep php` again and we will see:

```bash
php       started root           /Library/LaunchDaemons/homebrew.mxcl.php.plist
php@7.2   stopped
```

brew will add the file `/Library/LaunchDaemons/homebrew.mxcl.php.plist` when we run the service.

We can stop the service by running `sudo brew services stop php`

Running `brew services list | grep php` again we see the service is stopped and the file `/Library/LaunchDaemons/homebrew.mxcl.php.plist` is removed:

```bash
php       stopped
php@7.2   stopped
```  

The LaunchDaemons files are created under the home directory.

So on my machine, the actual full path of the LaunchDaemons files created for each version of the php-fpm service is:

`/Users/aregsarkissian/Library/LaunchAgents/homebrew.mxcl.php.plist`
`/Users/aregsarkissian/Library/LaunchAgents/homebrew.mxcl.php@7.2.plist`

> Note: We can start or stop either version of php-fpm regardless of which installed version of PHP is configured as our current active version.
So for example if we have both the latest version v7.3 and v7.2 installed and we have configured version 7.3 to be our current active version, we can still run `sudo brew services start php@7.2` to run the 7.2 installed version.
However it recommended to run the php-fpm version that matches your current active version. You can determine the current active PHP version by running `php -v`.

## Installing older versions of PHP

The installation steps for installing older versions of PHP is the same as installing the latest version of PHP except you need specify the PHP version number using the `@` symbol in the `brew install` command.

Assuming we have already installed the latest version 7.3 with `brew install php` we can also install the older version 7.2 as well by running:

`brew install php@7.2`

After running the command, brew will install the php files under:

`/usr/local/Cellar/php@7.2/7.2.18/`

as apposed to `/usr/local/Cellar/php/7.3.5/` for the latest 7.3 version.

The version 7.2  php, pecl and php-fpm binaries will then be installed at:

`/usr/local/Cellar/php@7.2/7.2.18/bin/php`
`/usr/local/Cellar/php@7.2/7.2.18/bin/pecl`
`/usr/local/Cellar/php@7.2/7.2.18/sbin/php-fpm`

and the v7.2 configuration files will be installed in:

`/usr/local/etc/php/7.2/`

where the `/usr/local/etc/php/7.2/php.ini` file will contain the PECL installation directory setting for php 7.2 extensions:

`extension_dir = "/usr/local/lib/php/pecl/20170718"`

> Note: the extensions directory uses a directory named with the PHP release date, instead of using the PHP version number, where the pecl installed extensions will be stored.

If we run the PHP v7.2 pecl command `/usr/local/Cellar/php@7.2/7.2.18/bin/pecl install xdebug` for instance, the xdebug.so extension file will be installed in the `/usr/local/lib/php/pecl/20170718` directory.

> Note: Instead of referencing the full `pecl` command path for a specific PHP version as shown in the example above, we would normally switch the active PHP version we are using as described in the next section and then just use the global `pecl` command to install extensions for the active version.

## Switching between latest and older installed PHP versions

In order to run older version of PHP when we execute the global `php` and `pecl` commands we need to use brew to switch the active PHP version to the older version.

Lets assume we installed the the latest 7.3 using `brew install php`. This would mean the current active version is version 7.3. Then we installed the older 7.2 version by running `brew install php@7.2`. The current active version would still be v7.3.
Now we can switch to the older version 7.2 by running the following commands:

```bash
sudo brew services stop php
brew unlink php && brew link --force php@7.2
sudo brew services start php@7.2
```

The `brew unlink php` command unlinks the latest version of php (v7.3) and `brew link --force php@7.2` links the older version 7.2 instead.

> Note: we need to use the --force flag when linking older versions of PHP

Before switching the version we also stop any running php-fpm instance for the latest version and afterwards we start the php-fpm service for the version we switched to.

If we look in `/usr/local/bin/` and  `/usr/local/sbin/` before executing the `brew unlink php && brew link --force php@7.2` command, we see the following php binary file symlinks pointing to the latest version of the brew PHP installation directories:

```bash
/usr/local/sbin/php-fpm -> /usr/local/Cellar/php/7.3.5/sbin/php-fpm

/usr/local/bin/pecl -> /usr/local/Cellar/php/7.3.5/bin/pecl
/usr/local/bin/pear -> /usr/local/Cellar/php/7.3.5/bin/pear
/usr/local/bin/peardev -> /usr/local/Cellar/php/7.3.5/bin/peardev
/usr/local/bin/phar -> /usr/local/Cellar/php/7.3.5/bin/phar
/usr/local/bin/phar.phar -> /usr/local/Cellar/php/7.3.5/bin/phar.phar
/usr/local/bin/php -> /usr/local/Cellar/php/7.3.5/bin/php
/usr/local/bin/php-cgi -> /usr/local/Cellar/php/7.3.5/bin/php-cgi
/usr/local/bin/php-config -> /usr/local/Cellar/php/7.3.5/bin/php-config
/usr/local/bin/hpdbg -> /usr/local/Cellar/php/7.3.5/bin/phpdbg
/usr/local/bin/phpize -> /usr/local/Cellar/php/7.3.5/bin/phpize
```

But after we run the `brew unlink php && brew link --force php@7.2` command we see the symlinks point to the older 7.2 version installation directories:

```bash
/usr/local/sbin/php-fpm -> /usr/local/Cellar/php@7.2/7.2.18/sbin/php-fpm

/usr/local/bin/pecl -> /usr/local/Cellar/php@7.2/7.2.18/bin/pecl
/usr/local/bin/pear -> /usr/local/Cellar/php@7.2/7.2.18/bin/pear
/usr/local/bin/peardev -> /usr/local/Cellar/php@7.2/7.2.18/bin/peardev
/usr/local/bin/phar -> /usr/local/Cellar/php@7.2/7.2.18/bin/phar
/usr/local/bin/phar.phar -> /usr/local/Cellar/php@7.2/7.2.18/bin/phar.phar
/usr/local/bin/php -> /usr/local/Cellar/php@7.2/7.2.18/bin/php
/usr/local/bin/php-cgi -> /usr/local/Cellar/php@7.2/7.2.18/bin/php-cgi
/usr/local/bin/php-config -> /usr/local/Cellar/php@7.2/7.2.18/bin/php-config
/usr/local/bin/phpdbg -> /usr/local/Cellar/php@7.2/7.2.18/bin/phpdbg
/usr/local/bin/phpize -> /usr/local/Cellar/php@7.2/7.2.18/bin/phpize
```

Remember that we added `/usr/local/bin/` and  `/usr/local/sbin/` to our PATH variable so when we run the `php` or `pecl` commands we will execute the command that is symlinked to the proper version of the binary file we want to run.

And since we are running the symlinked version of `pecl`, the php extensions that we install (or uninstall) will installed in the proper PHP version's extension file directory.

Running `php -v` we can see that the older version 7.2 is reported.

To switch back to using the latest version we need to reverse the process by executing:

```bash
sudo brew services stop php@7.2
brew unlink php@7.2 && brew link php
sudo brew services start php
```

Running `php -v` we can see that the latest version is reported again.

> Note: PHP extensions installed using `pecl install` command are installed for the version of the PHP that is currently activated. So you must install PHP extensions you need for each installed version of PHP separately after you switch to that version of PHP.

>Also note that the version of an extension that you need to install for older version of PHP, might not be the latest version of the extension because the latest version of the extension may be incompatible with the older PHP version. In that case you need to install the specific version of the extension you need, by specifying the extension version when you run `pecl install`. For instance instead of `pecl install xdebug` which will install the latest version of the extension, you might need to run `pecl install xdebug-2.7.1`.

## Resolving PHP 7.3 Composer package manager error

PHP uses a package manager named `Composer` that is a separate install from PHP itself.

If you have the Composer PHP package manager installed, you may need to deactivate the PHP v7.3 JIT compiler setting so that the `composer global update` command does not fail.

The JIT setting is in the `/usr/local/etc/php/7.3/php.ini` file.

You can run the following to see the setting:

```bash
cd /usr/local/etc/php/7.3
cat php.ini | grep 'pcre.jit'
```

Run the following command to uncomment the setting and change its value:

`sed -i '' 's/;pcre.jit=1/pcre.jit=0/g' /usr/local/etc/php/7.3/php.ini`

## Recap of installation directories for latest version and version 7.2

Here is a recap of the directories for the latest PHP version 7.3 and the older PHP version 7.2 for quick reference:

After running `brew install php` to install the latest version 7.3 we have the following installation directories:

PHP binaries:

`/usr/local/Cellar/php/7.3.5/bin/php`
`/usr/local/Cellar/php/7.3.5/bin/pecl`
`/usr/local/Cellar/php/7.3.5/sbin/php-fpm`

PHP configuration files:

`/usr/local/etc/php/7.3/php.ini`
`/usr/local/etc/php/7.3/php-fpm.conf`

PHP extensions location:

`/usr/local/lib/php/pecl/20180731/`

Opcache extension location:

`/usr/local/Cellar/php/7.3.5/lib/php/20180731/opcache.so`

In addition symlinks to binaries are added only when installing the latest version:

`/usr/local/bin/php`
`/usr/local/bin/pecl`
`/usr/local/sbin/php-fpm`

After running `brew install php@7.2` we have the following installation directories:

PHP binaries:

`/usr/local/Cellar/php@7.2/7.2.18/bin/php`
`/usr/local/Cellar/php@7.2/7.2.18/bin/pecl`
`/usr/local/Cellar/php@7.2/7.2.18/sbin/php-fpm`

PHP configuration files:

`/usr/local/etc/php/7.2/php.ini`
`/usr/local/etc/php/7.2/php-fpm.conf`

PHP extensions location:

`/usr/local/lib/php/pecl/20170718/`

Opcache extension location:

`/usr/local/Cellar/php@7.2/7.2.18/lib/php/20170718/opcache.so`

> Note: Installing the older versions of PHP does not create new symlinks to the installed binaries. We need to unlink the active latest version and link to the older version for brew to remove the existing symlinks and create new symlinks to the old versions installed binaries.

After we install the latest PHP version or after we install and switch the active version to an older version of PHP, we can run `pecl install` to add the specific PHP extensions that we need for the active version. And we can add the `--force` flag to the `pecl install` command in the case where pecl mistakingly thinks that the extension is already installed.

> Note: directories `/usr/local/Cellar/php/7.3.5/pecl` and `/usr/local/Cellar/php@7.2/7.2.18/pecl` created by the brew for the latest and 7.2 versions are both symlinks to `/usr/local/lib/php/pecl` which is the common base path for the PHP extensions for each installed version of PHP.

Also as an FYI there is an alternate installation location for PHP binaries for both versions that is not used:

`/usr/local/opt/php/bin/php`
`/usr/local/opt/php/bin/pecl`
`/usr/local/opt/php/sbin/php-fpm`
`/usr/local/opt/php@7.3/bin/php`
`/usr/local/opt/php@7.3/bin/pecl`
`/usr/local/opt/php@7.3/sbin/php-fpm`
`/usr/local/opt/php@7.2/bin/php`
`/usr/local/opt/php@7.2/bin/pecl`
`/usr/local/opt/php@7.2/sbin/php-fpm`

## Conclusion

In this article I listed the steps to take to install the Homebrew package manager for the macOS.

Next I showed how to upgrade an existing PHP installation using brew.

Then I showed how we can use brew to install the latest version of PHP from scratch followed by instructions on how to also install older versions of PHP side by side using brew.

I also explained how to use the PECL extension manager CLI for managing PHP extensions for each installed version of PHP.

Finally I showed how we can switch the active PHP version between multiple installed versions of PHP.

In the process I detailed where the PHP configuration files are installed and the configuration settings required for the PHP extensions. I also detailed the installation directories for the PHP binaries for each installed version.

The following resources were used to produce this article:

[https://medium.com/@romaninsh/install-php-7-2-xdebug-on-macos-high-sierra-with-homebrew-july-2018-d7968fe7e8b8](https://medium.com/@romaninsh/install-php-7-2-xdebug-on-macos-high-sierra-with-homebrew-july-2018-d7968fe7e8b8)

[https://vyspiansky.github.io/2018/11/08/set-up-php-7.2-on-macos-mojave-with-homebrew/](https://vyspiansky.github.io/2018/11/08/set-up-php-7.2-on-macos-mojave-with-homebrew/)

[bhttps://threenine.co.uk/setting-php7-development-mac-osx/](https://threenine.co.uk/setting-php7-development-mac-osx/)

[https://stitcher.io/blog/php-73-upgrade-mac](https://stitcher.io/blog/php-73-upgrade-mac)

[https://tommcfarlin.com/running-multiple-versions-of-php-with-homebrew/](https://tommcfarlin.com/running-multiple-versions-of-php-with-homebrew/)

[https://grrr.tech/posts/installing-homebrew-php-extensions-with-pecl/)](https://grrr.tech/posts/installing-homebrew-php-extensions-with-pecl/)

[https://discourse.brew.sh/t/pecl-with-multiple-php-versions/1977/2](https://discourse.brew.sh/t/pecl-with-multiple-php-versions/1977/2)

Thanks for reading.