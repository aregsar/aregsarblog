# Get Started with Production Ready Laravel

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post will show you how to start a Laravel project, develop it in a production scale ready manner in a series of posts.

These posts will show you how to setup your local environment for starting a new Laravel project.

Other posts in the series will then show you how to change the out of the box Laravel configuration to be production ready using Redis that uses docker Redis server image for development.

It will also show how to configure a MySQL database and use a docker MySQL service for local development.

It will show how to configure email and setup a local email server and email management dashboard in docker for development.

In future posts in the series we will cover provisioning infrastructure and deploying your application to a production environment that starts out small but has all the components be able to scale.

Note: The posts describe setting up a local PHP installation and Laravel Valet to serve your project in development. The remaining services used by your application to run locally, will be running in a docker environment.

If you do not want to or unable to install PHP, Composer, Valet and the Laravel CLI on your system, you can use Laravel sail instead. Laravel sail sets up everything as dcoker services so you can develop your code on docker. The one downside is that the filesystem access may be less perfomant.

## Environment setup

April 1, 2021

[Installing PHP on MacOS](https://aregsar.com/blog/2021/installing-php-on-macos)

April 1, 2021

[Installing PHP Extensions on MacOS](https://aregsar.com/blog/2021/installing-php-extensions-on-macos)

April 1, 2021

[Installing Composer on MacOS](https://aregsar.com/blog/2021/installing-composer-on-macos)

April 1, 2021

[Installing Laravel CLI on MacOS](https://aregsar.com/blog/2021/installing-laravel-cli-on-macos)

April 1, 2021

[Installing Laravel Valet on MacOS](https://aregsar.com/blog/2021/installing-laravel-valet-on-macos)

## Laravel services configuration

April 1, 2021

[Redis Configure Laravel](https://aregsar.com/blog/2021/redis-configure-laravel)

April 1, 2021

[Redis Configure Laravel Session](https://aregsar.com/blog/2021/redis-configure-laravel-session)

April 1, 2021

[Redis Configure Laravel Cache](https://aregsar.com/blog/2021/redis-configure-laravel-cache)

April 1, 2021

[MySQL Configure Laravel](https://aregsar.com/blog/2021/mysql-configure-laravel)

April 1, 2021

[Configure Laravel Local Mail Server](https://aregsar.com/blog/2021/configure-laravel-local-mail-server)

April 1, 2021

[Redis Configure Laravel Queue](https://aregsar.com/blog/2021/redis-configure-laravel-queue)
