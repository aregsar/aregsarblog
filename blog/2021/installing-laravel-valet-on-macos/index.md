# Installing Laravel Valet on MacOS

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

## Valet

```bash
composer global update
```

You can use valet use to switch between PHP versions:

```bash
valet use php@8.0
```

make sure to remove valet.sock file if it was not automatically removed:

```bash
cd ~/.config/valet
rm valet.sock
```

```bash
valet restart
```

> Must also restart valet anytime you install a PHP extension.
