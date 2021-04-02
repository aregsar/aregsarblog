# Installing Composer on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## installing composer

The following steps are described at [Composer](https://getcomposer.org/download)

download the setup script

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

Optionally, verify the script hash (make sure the hash code is the latest code from: [Composer Public Keys](https://composer.github.io/pubkeys.html) and replace the hash code below.

```bash
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

run the setup script

```bash
php composer-setup.php --filename=composer --install-dir=/usr/local/bin
```

remove the setup script

```bash
php -r "unlink('composer-setup.php');"
```

check installed version

```bash
composer --version
```

## Upgrading composer

upgrade to new version

```bash
composer self-update
```

check installed version

```bash
composer --version
```

composer install should have created a `~/.composer` directory

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
