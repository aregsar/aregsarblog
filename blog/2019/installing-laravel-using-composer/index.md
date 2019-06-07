# Installing Laravel using Composer

May 20, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Updated on May 29 by Areg Sarkissian

## Introduction

In this post I will detail how we can use Composer, the PHP package manager, to create a new Laravel project.

I will further show how we can run composer to install PHP packages for our Laravel project.

This post assumes that you have PHP and Composer installed on your local system.

> Note: An alternate Laravel installation CLI tool is available that you can install as a global Composer package. We will avoid using this tool and just use  the `composer create-project` command to create Laravel projects. This eliminates an additional dependency on the Laravel installer tool that we don't need to install and keep updated.

For completeness sake, I will also mention running the Node package manager, `Yarn` to install NPM modules for our project.

## Installing required PHP extensions for Laravel projects on macOS

Before we create a new laravel project we need to make sure certain PHP extensions are installed.

These are extensions required by Laravel:

+ BCMath PHP Extension
+ Ctype PHP Extension
+ JSON PHP Extension
+ Mbstring PHP Extension
+ PDO PHP Extension
+ Tokenizer PHP Extension
+ XML PHP Extension

I you install the latest PHP version using Homebrew for macOS, the extensions will be included by the PHP installation. So you will not need to explicitly install them. For Linux installation you will need to install all or some of these extensions individually using the package manager for your Linux distribution.

> Note: The Laravel documentation requires the OpenSSL PHP extension as a requirement. However OpenSSL does not exist in the PHP extensions package repository any longer. In the latest PHP installation, OpenSSL is selected by default. This can be seen by running `php -i | grep "SSL Version"` which shows `SSL Version => OpenSSL/1.0.2r`.

You can verify installed extensions by running `php -m`.

## Installing additional PHP extensions

I often install the following extensions by running a separate `pecl install` command for each individual extension:

```bash
pecl install --force xdebug
pecl install --force memcached
pecl install --force redis
```

> Note: It is recommended to install any required extensions individually instead of including a list of space character delimited extensions on a single line. This is because PECL gets confused when adding the extension setting to the php.ini file, if a zend extension such as xdebug is included in the list

## Using Composer to create a new Laravel project

We can run the following Composer command to download and create a new Laravel project:

`composer create-project --prefer-dist laravel/laravel:5.8.* myapp`

Here we specified `myapp` as the name of our project and `5.8.*` as the version of Laravel project skeleton that we want to create. Composer will create a project directory by the name of `myapp` and install the project skeleton files in the directory. If we leave out the project name, the project name will default to `laravel`.

> Note: If you are using PHP 7.3 and you get an JIT compiler setting error:
`preg_match(): JIT compilation failed: no more memory`
when you run the `composer create-project` command, you have to update the JIT compiler setting in your PHP configuration file as detailed in my blog post [Installing Composer PHP Package Manager](https://aregsar.com/blog/2019/installing-composer-php-package-manager) under the section `Fixing Composer JIT compiler error for PHP v7.3`.

When we execute the command the first line of the response will indicate the version of the Laravel project skeleton that is being installed:

`Installing laravel/laravel (v5.8.17)`

Once the project directory is created we can enter the project by running `cd myapp`.

To find out the version of the Laravel framework that is installed, within the project directory we can run:

`php artisan --version`

The output of this command for my laravel project was `5.8.19`.

As you can see the version of the installed Laravel framework `5.8.19`is different from the version of the Laravel project skeleton `5.8.17`.

To understand why, you need to understand that there are two distinct codebases that get installed, from two distinct repositories, when we create a new Laravel project.

First there is the Laravel project skeleton repository:

`https://github.com/laravel/laravel`

This repo is where the template files for your initial project structure come from.

Then there is the Laravel framework repository:

`https://github.com/laravel/framework`

This is where the files for the Laravel framework itself reside.

When we create a new Laravel project with `composer create-project` command, first Composer downloads a tagged version from the project skeleton repository into a project directory that it creates then Composer runs `composer install` from within the project directory to install the Laravel framework from the Laravel framework repo into the `vendor` subdirectory of the project directory.
The version of the Laravel framework that it will install into the vendor directiory is determined by the version of the Laravel framework specified in the `composer.json` file in the project skelleton release that was downloaded.

The project skeleton repository and the Laravel framework repositories have independent tagged releases. As of this writing the latest tagged releases for each repository is shown below:

`https://github.com/laravel/laravel/releases/tag/v5.8.17`

`https://github.com/laravel/framework/releases/tag/v5.8.19`

As you can see, the latest version for each is different and matches the versions that I saw for each when I installed the project skeleton from `https://github.com/laravel/laravel/releases/tag/v5.8.17` using `composer create-project`.

> Note: the `*` patch version used in the project skeleton version `5.8.*` of the create-project command indicates that composer should download the latest tagged release, which in this case is `5.8.17`.

If we look at the `composer.json` file of the project skeleton that was installed, we will see the following composer require section:

```bash
"require": {
        "php": "^7.1.3",
        "fideloper/proxy": "^4.0",
        "laravel/framework": "5.8.*",
        "laravel/tinker": "^1.0"
    },

```

As you can see `laravel/framework": "5.8.*` is specified. So when composer runs `composer install` during running the `composer create-project` installation, it will install the latest tagged release of the Laravel framework from `https://github.com/laravel/framework/releases/tag/v5.8.19`.

When we run `php artisan --version` from the project directory, the installed Laravel framework version is retrieved from the `vendor\laravel\framework\src\Illuminate\Foundation\Application.php` file. Inside this file the version is set to an `Application` class constant as show below:

 `const VERSION = '5.8.17';`

 This is where the artisan command retrieves the installed version of the Laravel framework.

> Note: Unfortunately the actual version of the project skeleton that was installed is not stored anywhere in the project. However since Composer prints out the version that it downloads in the results of the `composer create-project` command, we can copy it into the read.me file for future reference.

> Note: Always remember that the version we specify in the `composer create-project` command is the version of the project skeleton. The version of the Laravel framework that gets installed within the project is specified in the `composer.json` file of the project skeleton.

If we want a specific version of the project skeleton we can specify it explicitly in the command.

`composer create-project --prefer-dist laravel/laravel:5.8.17 myapp`

This will download version `5.8.17` of the project skeleton if exist as a tagged release in the project skeleton repo `https://github.com/laravel/laravel`.

> Note: If a tagged release does not exist for the specific version specified, then create-project will fail.

If we omit the patch version then the zero patch release will be downloaded:

`composer create-project --prefer-dist laravel/laravel:5.8 myapp`

This will download version `5.8.0` of the project skeleton.

If we omit the project skeleton version entirely, the latest tagged version of the project skeleton will be installed:

`composer create-project --prefer-dist laravel/laravel myapp`

This will download version `5.8.17` of the project skeleton.

In any of these cases the Laravel framework version that is installed will solely depend on the version specified in the `composer.json` file of the project skeleton.

## Installing Composer packages

As mentioned before composer create-project will also install the Laravel framework packages for us which will create the `vendor` directory where the packages are installed.

So running `composer install` from the project directory, after the installation is complete, should indicate that there are no new packages to install.

If we want to install additional packages using composer we can use the composer require command.

For instance `composer require predis/predis` will install the latest Redis package that Laravel needs if we want to use Redis as a cache or queue in our laravel projects.

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
