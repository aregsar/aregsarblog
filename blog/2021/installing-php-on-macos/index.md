# Installing PHP on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

```bash
brew update
```

## Normal upgrade

brew upgrade php

## Upgrade with shivammathur/homebrew-php

```bash
brew tap shivammathur/php
brew install shivammathur/php/php@8.0
```

```bash
brew link --overwrite --force php@8.0

php -v
```

```bash
php -r "phpinfo();"
```

```bash
php --ini
```

The response is below:

```bash
Configuration File (php.ini) Path: /usr/local/etc/php/7.4</hljs>
Loaded Configuration File: /usr/local/etc/php/7.4/php.ini
Scan for additional .ini files in: /usr/local/etc/php/7.4/conf.d
Additional .ini files parsed: /usr/local/etc/php/7.4/conf.d/ext-opcache.ini,
/usr/local/etc/php/7.4/conf.d/php-memory-limits.ini
```
