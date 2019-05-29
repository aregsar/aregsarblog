# Installing Laravel using Composer

May 20, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Introduction

In this post I will detail how we can use Composer, the PHP package manager, to create a new Laravel project.

I will further show how we can run composer to install PHP packages for our Laravel project.

This post assumes that you have PHP and Composer installed on your local system.

> Note: An alternate Laravel installation CLI tool is available that you can install as a global Composer package. We will avoid using this tool and just use  the `composer create-project` command to create Laravel projects. This eliminates an additional dependency on the Laravel installer tool that we don't need to install and keep updated.

For completeness sake, I will also mention running the Node package manager, `Yarn` to install NPM modules for our project.

## Installing required PHP extensions for Laravel projects on macOS

Before we create a new laravel project we need to make sure certain PHP extensions are installed. These are required by Laravel.

These extensions are already installed by the latest PHP installation.

> Note: The Laravel documentation requires the OpenSSL PHP extension as a requirement. However OpenSSL does not exist in the PHP extensions package repository any longer. In the latest PHP installation, OpenSSL is selected by default. This can be seen by running `php -i | grep "SSL Version"` which shows `SSL Version => OpenSSL/1.0.2r`.

You can check the installed extension by running `php -m`.

## Installing additional PHP extensions

I often install the following extensions by running a separate `pecl install` command for each individual extension:

```bash
pecl install --force xdebug
pecl install --force memcached
pecl install --force redis
```

> Note: It is recommended to install any required extensions individually instead of including a list of space character delimited extensions on a single line. This is because PECL gets confused when adding the extension setting to the php.ini file, if a zend extension such as xdebug is included in the list

## Using Composer to create a new Laravel project

We can run the following composer command to download and create a new Laravel project:

`composer create-project --prefer-dist laravel/laravel:5.8 myapp`

Here we specified `myapp` as the name of our project and `5.8` as the version of Laravel project that we want to create.

> Note: If you are using PHP 7.3 and you get an JIT compiler setting error:
`preg_match(): JIT compilation failed: no more memory`
when you run the `composer create-project` command, you have to update the JIT compiler setting in your PHP configuration file as detailed in my blog post [Installing Composer PHP Package Manager](https://aregsar.com/blog/2019/installing-composer-php-package-manager) under the section `Fixing Composer JIT compiler error for PHP v7.3`.

When running the command we did not specify a specific version patch such as `5.8.17`, so while the latest version of the Laravel framework `5.8.17` will be installed but the zero patch version `5.8.0` of the Laravel project files will be installed .

To get a specific patch version of the project files, just specify the full version like so:

`composer create-project --prefer-dist laravel/laravel:5.8.17 myapp`

> Note: This will still install the latest Laravel framework patch for the minor version of Laravel, regardless of the patch number we specfied. So there is a distinction between the version of the __framework__ files and the version of the __project_ files. The framework files are located in `vendor/laravel/` directory and the project files are all the files in the project directory and child directories except the `vendor` directory.

If we omit the Laravel version entirely, the latest version of the __framework__, including the latest patch, will be installed as usual. In addition the latest version of the __project__ files will be installed.

So running the following will install version `5.8.17` of the project files, if that is the latest version and within that project folder it will install version `5.8.19` of the framework files in the vendor directory, if that is the latest version of the framework:

`composer create-project --prefer-dist laravel/laravel myapp`

> Note: If a specific version or patch number of the __project__ files does not exist in the composer repository, then the install will fail. Even if that patch is a valid version for the __framework__ files.

After installation is complete, run `cd myapp` to enter the project directory and view the project files and directories.

## Installing Composer packages

Since we are using Composer to create the project, Composer will also install the Laravel packages for us which will create the `vendor` directory where the packages are installed.

So running `composer install` from the project directory should indicate that there are no new packages to install.

## Checking the installation using the Artisan CLI

We can run the Laravel `Artisan` CLI command from within the project directory to verify the version of the Laravel __framework__ files in the vendor directory:

`php artisan`

This should list the installed Laravel version and the list of available Artisan commands.

> Note: To display the installed Laravel __framework__ verion, the php artisan command looks in file `vendor/laravel/framework/src/Illuminate/Foundation/Application.php` to inspect the `const VERSION = '5.8.17';` property specified in the `Application` class. This version number may be out of sync with version of the actual Laravel __project__ files that composer installs.

## Checking the installed version of the project files

To recap, the Laravel __framework__ version installed via the composer install command is always the latest patch version of the framework for the minor version of the installed framework.

This is distinct from the version of the installed project files. The only way to know the version of the project files is to check the output of the composer install command and note the actual version of the project files that it is installing.

So for instance after running `composer create-project --prefer-dist laravel/laravel:5.8 myapp` I can see that composer is creating version `5.8.0` of the __project__ files. But at the same time it is installing version `5.8.19` of the __framework__ files in the vendor directory.

If I try to install `laravel/laravel:5.8.19`, it fails because the version `5.8.19` of the project files does not exist, since the latest version of the project files is `5.8.17`. But trying to install `laravel/laravel:5.8.17` succeeds in installing the version `5.8.17` of the project files and yet it installs version `5.8.19` of the framework files in the vendor directory. The installation output shows that version `5.8.17` of the project files are being installed, but if I run `php artisan` it shows version `5.8.19` because that command is reading the version number of the installed framework files from `vendor/laravel/framework/src/Illuminate/Foundation/Application.php`.

## Installing NPM modules

If we have NodeJS installed on our system, we can also run `yarn` from the project directory to install NPM modules in our project. If we don't have NodeJS installed, another way to install the NPM packages is to run `yarn` from the project directory using the official Node Docker container:

`docker run --rm -v $(pwd):/app -w /app node yarn`

Either way running the command will create the `node_modules` directory where the NPM modules will be installed.

Running `yarn` will also create the `yarn.lock` dependency file in the project directory.

## Running the application

To run our application, we can run a PHP server using the following Artisan command:

`php artisan serve --port=8080`

We should now be able to navigate our web browser to `localhost:8080` to see the Laravel application home page.

In another post I will detail how we can install and configure Laravel `Valet` to serve all our Laravel apps without having to explicitly run a web server.

## Conclusion

In this post I detailed how we can create a new Laravel project with composer, using any version of the Laravel framework.

Thanks for reading.