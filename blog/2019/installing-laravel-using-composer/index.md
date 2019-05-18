# installing laravel using composer

[installing laravel using composer](https://aregsar.com/blog/2019/installing-laravel-using-composer)

## Introduction

In this post I will detail how we can use the Composer, the PHP package manager, to create a new Laravel project using any version of Laravel.

I will further show how we can run composer to install the packages for our project and even run yarn to install Node modules for our project.

This post assumes that you have installed PHP and also installed Composer on your local system.

> Note: An alternate Laravel project creation tool exists that you can install as a global CLI tool using Composer. We will avoid using this tool and just use Composer directly to create Laravel projects using the composer create-project command. This eliminates another global dependency that we need to worry about and keep updated.

## Using Composer to create a new Laravel project

We ca run the following composer command to download and create a new Laravel project:

`composer create-project --prefer-dist laravel/laravel:5.8 myapp`

Here we specified the version of Laravel we want as `5.8` and named our project `myapp`.

Since we are using Composer to create the project, Composer will also install the Laravel packages for us an create the `vendor` directory where it installs the packages.

We can run `cd myapp` to see all the project files and directories.

Once in the project directory we can run `yarn` and `npm install`, if we have NodeJS installed on our system, to install NPM modules in our project. If we don't have NodeJS installed a quick way to install the NPM packages is to run the Node Docker container:

`docker run --rm -v $(pwd):/app -w /app node yarn`

Either way running the command will create the `node_modules` directory
and install the NPM modules there. It will also create the `yarn.lock` dependency file.

## Checking the installation

We can try running the Laravel Artisan CLI command to verify the installation:

`php artisan`

This should list the installed Laravel version and the list of available Artisan commands.

We can also run `composer install` just to make sure we don't see any errors related to PHP 7.3.

Running the command will not install any packages since they are already installed.

> Note: Composer and PHP 7.3 Compiler JIT issue

Finally we can run the Laravel built in server using the Artisan command:

`php artisan serve --port=8080`

Now we should be able to navigate our web browser to `localhost:8080`
to see the Laravel application home page.

In another post I will detail how we can install and configure the Laravel Valet to serve all our Laravel apps without having to explicitly run a web server.

## Conclusion

In this post I detailed how we can create a new Laravel project with any version of the Laravel framework using Composer.

Thanks for reading.