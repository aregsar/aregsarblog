# Upgrade Php Version MacOS

April 3, 2020 by [Areg Sarkissian](https://aregsar.com/about)

Use Homebrew MacOS package manager to upgrade the default php version to the latest php version.

First update homebrew to latest version

```bash
brew update
```

Then upgrade PHP to the latest version

```bash
brew upgrade php
```

Make sure to source your bash profile or relaunch the terminal for the update to the $PATH variable to take effect.

Sourcing the bash profile

```bash
source ~/.profile
```

> Note: The reason we need to source the bash profile is to ensure the $PATH variable has the `/usr/local/bin` path before the `/usr/bin` path. This is because the symlink to the current php version is in `/usr/local/bin` which needs to override the default macOS installed PHP version symlink in `/usr/bin`

Now we can check the current PHP version

```bash
php -v
```

## Switching current PHP version

We can switch between multiple installed PHP versions globally

Assuming we have both php@7.4 and php@7.3 installed and assuming the current version of php is set to php@7.4, we can switch to php@7.3 like so:

```bash
brew unlink php && brew link --force --overwrite php@7.3
```

> Note: this unlinks the `php` symlink located at `usr/local/bin/php` that is currently pointing to the `php@7.4` version located at `/usr/local/Cellar/php/7.4.4/bin` to point to the `php@7.3` version located at `/usr/local/Cellar/php/7.3.5/bin`

To switch back simply change the version number in the command:

```bash
brew unlink php && brew link --force --overwrite php@7.4
```

I added the following function to my bash profile to easily switch to a specific PHP version.

```bash
phpver() {
    brew unlink php && brew link --force --overwrite php@$1
}
```

Now I can simply type `phpver 7.4` to set my php version to php@7.4

## Install PECL libraries

Follow instructions here to add php extension libraries:

[installing-homebrew-php-extensions-with-pecl](https://grrr.tech/posts/installing-homebrew-php-extensions-with-pecl)
