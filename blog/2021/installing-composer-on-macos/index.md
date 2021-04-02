# Installing Composer on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## installing composer

The following steps are described at [Composer](https://getcomposer.org/download)

1-Download the setup script

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
composer require vendor/package
```

This will add the package to the composer.json and install the package.

If there is no existing composer.json file, it will be created.

To add a package as a development dependency we can use the --dev option:

```bash
composer require vendor/package --dev
```

To add a package globally we can use the global subcommand:

```bash
composer global require vendor/package
```

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

Removes installed packages that are no more required in composer.json

Checks the availability of the latest versions of your required packages

Installs the latest versions of your packages by downloading the latest version of their files into the vendor directory

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

if it does not exist, runs the composer update command to create it as mentioned in previous section.

Reads the composer.lock file

Installs the versions of the packages specified in the composer.lock file

## Composer install Options for production builds

```bash
composer install --no-interaction --optimize

```

## dumping the autoload configuration

Whenever we update the PSR-4 autoloader configuration in composer.json we need to make composer aware or the changes
by dumping the autoloader configuration. This command updates the autoloader files under the vendor directory.

```bash
composer dump-autoload
```

To dump an optimized version of the autoloader for production we can use the -o or --optimize option

```bash
composer dump-autoload -o
```

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

## Creating the project

```bash
mkdir myproject && cd myproject
mkdir app
composer require xxx/guzzlehttp:^7.2
#add the psr 4 autoload section "autoload": {"psr-4": {"App\\": "app/"}} to the
#composer.json file
composer install
#or
#composer dump-autoload
echo "<?php\nrequire_once(__DIR__ . '/vendor/autoload.php');" > index.php
#add files and directories under app directory

#run the code
php index.php
```

## Composer cheet sheet for PHP projects

curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

semantic versioning: avoids breaking updates
^2.6 == >=2.6.4 <3.0.0

semantic versioning: any patch level that might introduce a breaking change
~2.6 == >=2.6 <3.0

require is for loading packages

autoloading is for our PSR-4 classes and generates vendor/composer/autoload_psr4.php

classmap is for loading classes in directories that are in a
directory path outside the psr4 directory starting with the project root
scans classmap specified files and direvctories for php classes
classmap generates vendor/composer/autoload_classmap.php

files is for loading files with global php functions

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
```

## Composer commands

```bash
#update composer
composer self-update

#install dependancies
composer install

#prefer packagist
composer install --prefer-dist

#prefer cloning from vcs
composer install --prefer-source

#production install
composer install --no-dev

#update all installed dependancies (accourding to version settings in composer.json)
composer update

#update dependancy (accourding to version setting in composer.json)
composer update symfony/yaml

#install package
composer require symfony/yaml

#install as dev dependancy
composer require phpunit/phpunit:^8 --dev

#tree view of composer.lock file
composer show -t

#generates composer.json interactively
composer init

#generates composer.json and vendor directory if they dont exist
composer require acme/acmepackage

#call dump-autoload after adding the autoload section to composer.json
#to generate vendor/composer/autoload_psr4.php
#composer install --optimize-autoloader will also dump-autoload
composer dump-autoload -o

#both --optimize-autoloader and dump-autoload generate the classmap

#production options
composer install --no-dev --optimize-autoloader --no-interation --no-scripts --no-progress

# using -o or --optimize-autoloader with composer install will automatically call composer dump-autoload -o

#this installs packages and dump-autoloads

#it does so by setting "optimize-autoloader": true inside the config key of composer.json
composer install --optimize-autoloader --no-dev
composer install -o --no-dev

#this does not install anything. It just generates the classmap
composer dump-autoload -o
composer dump-autoload --optimize

#if project does not use dynamic classes than this optimizer better
#it does so by setting "classmap-authoritative": true inside the config key of composer.json
composer install --classmap-authoitative --no-dev
composer install -a --no-dev

#this does not install anything. It just generates the classmap
composer dump-autoload --classmap-authoritative
composer dump-autoload -a

============================================================

//usage in code

require_once **DIR** . '/../vendor/autoload.php';


#psr 4 directory structure and namespace mapping
app\User.php => namespace App;
app\Config\Storage.php => namespace App\Config;
app\Helpers\Time.php => namespace App\Helpers;
```

```php
//User.php

<?php
namespace App;
use App\Helpers\Time;
class User{}


#app\Helpers\Time.php
<?php
namespace App\Helpers;
class Time{}
```

```php
//app\Config\Storage.php
<?php
namespace App\Config;
use App\User;
use App\Helpers\Time;
class Storage{}


//index.php (root bootstrap file that autoloads all classes)
require_once __DIR__ . '/../vendor/autoload.php';
use App\User;
use App\Config\Storage;
$user = new User;
$db = new Storage;
```

## Execute PHP as script file

```bash
cat phpinfo.php <<EOF
#! /usr/bin/env php
<?php
//require_once __DIR__ . '/vendor/autoload.php';
phpinfo();
EOF
chmod +x phpinfo.php
#execute
./phpinfo.php
```
