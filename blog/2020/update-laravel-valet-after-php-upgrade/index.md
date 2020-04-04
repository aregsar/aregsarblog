# Update Laravel Valet After PHP Upgrade

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

Source the bash profile to make sure path to latest PHP version is set

```bash
source ~/.profile
```

Next update composer

```bash
composer global update
```

Now update valet by installing it again

```bash
valet install
```

Finally tell valet to user the latest upgraded php

```bash
valet use php
```
