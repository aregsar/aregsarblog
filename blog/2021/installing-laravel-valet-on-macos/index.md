# Installing Laravel Valet on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This post is part of the [Get Started with Production Ready Laravel](https://aregsar.com/blog/2021/get-started-with-production-ready-laravel) series of posts.

## Prerequisites

Install Homebrew

Install PHP (should install php-fpm)

Install Composer

Add `/usr/local/bin to your` system path

Valet will install `dnsmasq` and `nginx` using homebrew

## Installing and configuring valet

To install Valet run the following commands:

```bash
composer global require laravel/valet

valet --version

valet install

valet trust

valet use php

valet start
```

## Updating and re-intstalling Valet

```bash
valet stop

#run this only if you have issues with valet installing
#see notes below about possible side effects of this command
#valet uninstall --force

composer global update

valet --version

valet install

valet trust

valet use php

valet start
```

The `valet start` command also starts the dnsmaq and nginx services

> Note. It seems that the `valet uninstall` command removes the current php installation so PHP and possibly its extensions may need to be re-installed via homebrew and pecl again. Also seems like the valet uninstall effected my current nvm node setting where node and npm were not found. This was most likely due to the node path setting being modified during the valet uninstall. To resolve that I used the nvm use <nodeversion> command to reset the node path.

## Resetting valet to use a new php version

When we swich php versions, the php command point to the new php version.
Just to be safe we may have to inform valet about this by doing the following:

```bash
valet stop

rm ~/.config/valet/valet.sock

#php now point to a different version of php
valet use php

valet start
```

Also repeat the same steps anytime you install a new PHP extension.

Check that the `~/.config/valet/valet.sock` file exists.

If you run into issues. Try running:

```bash
valet install

valet use php

valet restart
```

[See this for valet.sock issue](https://stitcher.io/blog/php-8-upgrade-mac)

## Uninstalling and removing valet

```bash
valet stop

valet uninstall --force

composer global remove laravel/valet

sudo rm -rf ~/.config/valet

rm /usr/local/bin/valet
```

Additionally can stop and uninstall the valet installed dnsmasq and nginx services

```bash
sudo brew services stop nginx
sudo brew services stop dnsmasq
brew uninstall dnsmasq
brew uninstall nginx
```

## Issues with composer when running composer update

composer global update is run anytime run composer global remove or composer global require
we can also directly run composer global update

In either case sometimes composer global update fails with version conflict warnings

If this happens when trying to install, update or remove valet you may have to delete your composer.lock file and the vendor directory in the composer home directory ~/.composer

Then manually remove the valet package from composer.json and run:

```bash
composer global require laravel/valet
```

If you are unable to install due to composer errors, you may need to delete the composer.json and reinstall all the
installed global packages including Laravel valet.

See the composer post for handling version conflict error details

## Starting, stoping and restarting valet

Starting stoping and restarting valet also starts stops and restarts the dnsmasq and nginx services

```bash
valet start

valet stop

valet restart
```

## Serving sites with valet

linking all projects within a directory

```bash
cd ~/myprojects

valet park
```

linking a single project

```bash
cd ~/myspecialprojects/myapp

valet link
```

Each project will be served locally using the project name and .test domain

`http://<project_name>.test`

The site will also respond to any subdomain

`http://<subdomain>.<project_name>.test`

we can show the linked sites

```bash
valet links
```

Also if we check the file `~/.config/valet/config.json` we will see that `~/myspecialprojects` directory is added to the paths property.

We can also the tld (top level domain) property is set to test which allows serving using .test domain. We can change this and run `restart valet` to serve using a different domain. (just make sure it does not conflict with a real domain since dnsmasq will redirect it to the local nginx server)

```bash
cat ~/.config/valet/config.json
```

```json
{
  "tld": "test",
  "paths": ["/Users/aregsarkissian/zdev/laravel"]
}
```

## Removing linked sites

We can remove individual sites within a parked directory by entering the project and using the unlink command:

```bash
cd ~/myprojects/myotherapp

valet unlink
```

To unlink all projects within a parked directory enter the directory and use the forget command

```bash
cd ~/myprojects

valet forget
```

## Valet configuration files

When we run the `valet install` command, valet creates the configuration directory `~/.config/valet/`

Important configuration files and directories

The `~/.config/valet/config.json` file lists the directories that hosts the Laravel projects.

Here is a list of other notable configuration locations:

```bash
~/.config/valet/config.json

~/.config/valet/Sites/

~/.config/valet/dnsmasq.d/

~/.config/valet/Nginx/

/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf

~/.config/valet/valet.sock
```

for a full list and descriptions see Laravel valet documentation

[https://laravel.com/docs/8.x/valet](https://laravel.com/docs/8.x/valet)

## Valet executable

requiring valet globally installs the valet executable file at:

```bash
~/.composer/vendor/laravel/valet/valet
```

The installation also adds the symlinks:

```bash
~/.composer/vendor/bin/valet -> ~/.composer/vendor/laravel/valet/valet
```

Usually we add `~/.composer/vendor/bin` to our system `$PATH` variable to be able globally run executables symlinks that composer sets up in `~/.composer/vendor/bin`.

However Valet installs a symlink in `/usr/local/bin` that points to the symlink in `~/.composer/vendor/bin`.

```bash
/usr/local/bin/valet -> /Users/aregsarkissian/.composer/vendor/laravel/valet/valet
```

So as long as we have `/usr/local/bin` in our system `$PATH` we will be able to at least run the valet command globally.

To see the location of the valet executable:

```bash
which valet
```

Which points to the composer symlink in my case since the `~/.composer/vendor/bin` in my `$PATH` proceeds the `/usr/local/bin` in `$PATH`.

```bash
/Users/aregsarkissian/.composer/vendor/bin/valet
```

## How valet works

the `valet install` command will use Homebrew to download and install Nginx and Dnsmasq servers and configure Valet to use them. Installing Nginx and Dnsmasq will add the nginx and dnsmasq services on your system and start them up.

run `brew list` to see if nginx and dnsmaq are installed

we can manually install and start them if for some reason valet is unable to install them

```bash
brew install dnsmasq
brew install nginx
sudo brew services start dnsmasq
sudo brew services start nginx
```

Valet is configured by default to route any `<project_name>.test` domain using dnsmasq DNS resolver to the local Nginx server.
Dnsmasq resolves .test domains to the localhost, which the Nginx server is configured to listen on, while allowing other domains to be resolved to their IP address on the web.
In its valet configuration file at `~/.config/valet/valet.conf`, Nginx uses the Laravel `project_name` in the domain to pass to `fastcgi_index /Users/aregsarkissian/.composer/vendor/laravel/valet/server.php` php file that uses the valet driver to serve the project by using php to execute the public/index.php file in the project, hence serving the project at that URL.

The snippet below from `~/.composer/vendor/laravel/valet/server.php` shows how the php process `requires` the `index.php` file which is referenced as the $frontControllerPath in the script to execute the code in `index.php`:

```php
$frontControllerPath = $valetDriver->frontControllerPath(
    $valetSitePath, $siteName, $uri
);
chdir(dirname($frontControllerPath));
require $frontControllerPath;
```

Valet configures the project paths including the project name and the `.test` domain in ~/.config/valet/config.json.

## The valet.conf file

Below is the content of the `~/.config/valet/valet.conf` file

```nginx

server {
    listen 127.0.0.1:80 default_server;
    root /;
    charset utf-8;
    client_max_body_size 128M;

    location /41c270e4-5535-4daa-b23e-c269744c2f45/ {
        internal;
        alias /;
        try_files $uri $uri/;
    }

    location / {
        rewrite ^ "/Users/aregsarkissian/.composer/vendor/laravel/valet/server.php" last;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log "/Users/aregsarkissian/.config/valet/Log/nginx-error.log";

    error_page 404 "/Users/aregsarkissian/.composer/vendor/laravel/valet/server.php";

    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass "unix:/Users/aregsarkissian/.config/valet/valet.sock";
        fastcgi_index "/Users/aregsarkissian/.composer/vendor/laravel/valet/server.php";
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME "/Users/aregsarkissian/.composer/vendor/laravel/valet/server.php";
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
