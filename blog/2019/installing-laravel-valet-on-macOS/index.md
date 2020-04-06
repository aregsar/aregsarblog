# Installing Laravel Valet on macOS

June 7, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Introduction

In this post I will detail the installation and running of Laravel Valet on macOS using the PHP Composer package manager.

> Note: Laravel Valet has been ported to `Ubuntu Linux`. Check out these links for installing Valet on `Linux` or `Windows Subsystem for Linux` on Windows 10: `https://cpriego.github.io/valet-linux/` and `https://github.com/cpriego/valet-linux`. You may be able to install Valet directly on the Windows 10 OS as well by using these resources `https://blog.hashvel.com/posts/how-to-install-laravel-valet-on-windows-os/` and `https://github.com/cretueusebiu/valet-windows`. I have not attempted to install Valet on Linux or Windows so this post only applies to macOS.

Laravel Valet is a package that consists of a daemon process that runs in the background and serves up Laravel sites through an Nginx proxy server.

When we install Valet, it uses the Homebrew package manager for macOS to automatically install an Nginx web server, if it is not already installed.

During the installation, Valet also uses Homebrew to install a DNS server called Dnsmasq.

Dnsmasq acts a local DNS server intercepting and routing URIs with`.test` domain to our local Nginx server.

The URI's that Valet serves will be of the form `<project-name>.test` where the project name in the URL specifies which Laravel project will be served.

Valet maps the project names to the directory name of the project to be served, by using the `Valet park` command in the parent directory of the project.

> Note: we can configure a different domain name instead of `.test` for Valet to use. You can check out the documentation for Laravel Valet for further info. `https://laravel.com/docs/5.8/valet`.

## Uninstalling Valet

To get a clean install, in case you have an old installation of Valet, you can remove Valet and its dependencies by running the following

```bash
valet uninstall
composer global remove laravel/valet
sudo brew services stop nginx
sudo brew services stop dnsmasq
brew uninstall dnsmasq
brew uninstall nginx
sudo rm -rf ~/.config/valet
rm /usr/local/bin/valet
```

Some of the steps may not be necessary based on your installation but will not do any harm.

## Installing Valet via Composer

To install Valet we can run:

`composer global require laravel/valet`

the global `composer.json` file is located at:

`~/.composer/composer.json`

So requiring valet globally installs the valet executable file at:

`~/.composer/vendor/laravel/valet/valet`

The installation also adds the symlinks:

`~/.composer/vendor/bin/valet` -> `~/.composer/vendor/laravel/valet/valet`

So we have to make sure `~/.composer/vendor/bin` is added to the path env variable to run the `valet` command.

The installation also creates the configuration directory for Valet at:

`~/.config/valet/`

Once Valet is installed, we can check its version by running:

`valet --version`

## Configuring Valet with Nginx ad Dnsmaq

After Composer installs Valet, we need to configure Valet. We can do so by running:

`valet install`

This command will use Homebrew to download and install Nginx and Dnsmasq servers and configure Valet to use them.
Installing `Nginx` and `Dnsmasq` will add the `nginx` and `dnsmasq` services on your system and start them up.

## Configuring Valet to serve web projects

To add our projects to be served by Valet, go to the directory where our Laravel projects reside and run:

`valet park`

This will serve any project using the URI `<project-name>.test` in the web browser.

For example, let's say you had a Laravel project directory named `movies` and this directory was created inside the `~/dev` directory. In this case you would navigate to `~/dev` directory and run the `valet park` command.

After running the command, you should be able to navigate your browser to `movies.test` and see your Laravel `movies` project homepage being served.

Also if we check the file `~/.config/valet/config.json` shown below we will see that `~/dev` is added to the `paths` property. Note that the `tld` (top level domain) property is set to `test` which is why we append the `.test` domain to the project name.

```json
{
    "tld": "test",
    "paths": [
        "/Users/aregsarkissian/dev"
    ]
}
```

## Troubleshooting Valet install

If after running `valet install` and `valet park`, the sites are not being resolved, check if the `nginx` and `dnsmsq` services were actually installed and are running.

You can verify if they were installed and are running by running this command:

`brew list`

If either service is not listed or is stopped you will need to install and run it manually.

When I uninstalled Valet and re-installed it on my system, for some reason the `valet install` command did not re-install the Nginx and Dnsmasq packages.

So I had use Homebrew to manually install and run them so Valet would work.

Here are the steps I took

```bash
brew install dnsmasq
brew install nginx
sudo brew services start dnsmasq
sudo brew services start nginx
```

## Troubleshooting other issues

One issue I ran into with running the valet command was that it required `sudo` privilege

To not require `sudo` execute:

```bash
valet trust
```

For some reason `valet park` was not working because app was not served using .test tld.

To fix you can try:

```bash
valet start
```

This restarts all the `nginx` and `dnsmasq` processes

If it is still not working try re-installing valet

If that does not work try un-installing valet then doing a fresh install

> Note the valet uninstall removes the current php installation so PHP and possibly its extensions may need to be re-installed via homebrew and pecl again. Also seem like the valet uninstall effected my current nvm node setting where node and npm were not found. This was most likely due to the node path setting being modified during the valet uninstall. To resolve that I used the `nvm use <nodeversion>` command to reset the node path

## Conclusion

In this post I described how to install Laravel Valet on macOS to be able to serve all your Laravel projects without having to manually launch a web server.

Thanks for reading.
