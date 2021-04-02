# Installing Composer on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## installing composer

The following steps are described at [Composer](https://getcomposer.org/download)

1-Download the setup script

curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

2-Optionally, verify the script hash

```bash
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

> Make sure to replace the hash code above with the latest code from [Composer](https://getcomposer.org/download) or from [Composer Public Keys](https://composer.github.io/pubkeys.html).

3-Run the setup script

```bash
php composer-setup.php --filename=composer --install-dir=/usr/local/bin
```

4-Remove the downloaded setup script

```bash
php -r "unlink('composer-setup.php');"
```

5-Verify installed version

```bash
composer --version
```

> The composer script install should have created a `~/.composer` directory and added a `COMPOSER_HOME` environment variable with that path to your system PATH in your bash profile.

## Installing composer the quick install way

This alternative installation method uses curl to install composer and does not verify the hash

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

## Upgrading composer

upgrade to new version

```bash
composer self-update
```

Verify the updated version

```bash
composer --version
```

## removing composer

```bash
rm /usr/local/bin/composer
rm -rf ~/.composer
```

## creating the initial composer.json file

```bash
composer init
```

This will interactively create a new composer.json file. If you don't create a composer.json file before requiring packages, then it when automatically be created for you when you require your first package.

## requiring packages

```bash
#the latest version
composer require vendor/package

#specifying a version
composer require vendor/package:<version>
```

This will add the package to the composer.json and install the package into the vendor directory. If there is no existing composer.json file, it will be created.

To add a package as a development dependency we can use the --dev option:

```bash
composer require vendor/package --dev
```

To add a package globally we can use the global subcommand:

```bash
composer global require vendor/package
```

Example installing phpunit including the version as a dev dependency

```bash
composer require phpunit/phpunit:^8 --dev
```

> Note `composer require` command updates the composer.json file and installs the package in the vendor directory (creating the vendor directory if it does not exist) but does not update or create the composer.lock file.

## updating packages

```bash
composer update
```

To update global packages we can use the global subcommand which uses the composer.json from our ~/.composer directory:

```bash
composer global update
```

These commands preform the following:

Reads composer.json

Removes installed packages that are no more required in composer.json (if the package was removed from composer.json or its version was changed )

Checks the availability of the latest versions of your required packages

Installs the latest versions of your packages, as restricted by the package version in the composer.json file, by downloading the package files into the vendor directory. (see section on package versioning scheme below in the Composer.json structure section).

Updates composer.lock to store the installed packages version by writing all of the packages and the exact versions of them that it downloaded to the composer.lock file, locking the project to those specific versions

## installing packages

To install or reinstall packages using the composer.json file, run the following command:

```bash
composer install
```

To install or reinstall packages globally using the global composer.json in the ~/.composer directory use the global subcommand:

```bash
composer global install
```

These command perform the following steps:

Checks if composer.lock file exists

if it does not exist, runs the `composer update` command to create generate the composer.lock file as mentioned in previous section.

Reads the composer.lock file

Installs the versions of the packages specified in the composer.lock file

## Composer install Options for production builds

```bash
composer install --no-dev --optimize-autoloader --no-interation --no-scripts --no-progress --prefer-dist
```

> note --prefer-dist tries to install from packagist.org and falls back to git repository. Replace it with --prefer-source to try to install from the git repository then fallbask to packagist.org

## Removing packages

Simply remove the package from the composer.json file and re-run `composer install`.

This will run `composer update` which will remove the package and its dependancies (if they are not required by another package) and will update the composer.lock file by removing the package and its dependancies from the file.

## displaying composer.lock file dependency tree

```bash
composer show -t
```

## dumping the autoload configuration in development

Whenever we update the PSR-4 autoloader configuration in composer.json file we need to make composer aware of the changes
by dumping the autoloader configuration. This command generates and\or updates the autoloader files under the vendor directory with the classmap.

```bash
composer dump-autoload
```

## dumping the autoload configuration in production

To dump an optimized version of the autoloader for production we can use the -o or --optimize option

```bash
#generate classmap file vendor/composer/autoload_psr4.php
composer dump-autoload --optimize
```

In production when we run `composer install` we can dump the autoload configuration at the same time using the -o or --optimize-autoloader option. No need to for a separate call to

```bash
# using =--optimize-autoloader with composer install will automatically call composer dump-autoload -o
#it does so by setting "optimize-autoloader": true inside the config key of composer.json
composer install --optimize-autoloader --no-dev --no-interation --no-scripts --no-progress --prefer-dist
```

> if project does not use dynamic classes than --classmap-authoritative optimizer option is better than --optimize-autoloader option

## PSR-4 autoloader configuration

The autoloader configuration in the composer.json file maps a class namespace starting from the root of the project where the composer.json file resides to a directory under the root.

```json
{
  "PSR-4": {
    "\\App": "/app",
    "\\Resources\\Views": "/resources/views"
  }
}
```

> Note the root class namespace in the mapping uses a `\\` escape sequence

The class namespacing convention then follows the directory hierarchy starting from the mapped directory on down.
For example the class `User` in the file app/user.php will map to the \App\User namespace and the class Database in the file app/configs/database.php file will map to \App\Config\Database namespace

## The Composer.json structure

require section is for loading packages

semantic versioning: avoids breaking updates
^2.6 == >=2.6.4 <3.0.0

semantic versioning: any patch level that might introduce a breaking change
~2.6 == >=2.6 <3.0

autoloading section is for our PSR-4 classes and generates vendor/composer/autoload_psr4.php

classmap is for loading classes in directories that are in a
directory path outside the psr4 directory starting with the project root
scans classmap specified files and direvctories for php classes
classmap generates vendor/composer/autoload_classmap.php

files is for loading files with global php functions

repositories is for instructing composer to look for a package to install from the repo location first

````bash

## The composer.json file

```json
{
  "classmap": ["database/Seeders", "database/Factories", "factories/"],
  "files": ["global/functions.php"],
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "App\\Tests": "test/"
    }
  },
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/Seldaek/monolog"
    }
  ],
  "require": {
    "php": ">=7.4",
    "symfony/yaml": "2.6.4",
    "monolog/monolog": "1.12.0"
  },
  "require-dev": {
    "phpunit/phpunit": "^8"
  }
}
````

## composer autoloader and PHP namespaces relationship

In the index.php bootstrap file:

```php
require_once __DIR__ . '/../vendor/autoload.php';
```

psr 4 directory structure and namespace mapping

app\User.php => namespace App;
app\Config\Storage.php => namespace App\Config;
app\Helpers\Time.php => namespace App\Helpers;

in app\User.php

```php
<?php
namespace App;
use App\Helpers\Time;
class User{}
```

in app\Helpers\Time.php

```php
<?php
namespace App\Helpers;
class Time{}
```
