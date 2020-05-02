# connecting to mysql and redis from laravel

[connecting to mysql and redis from laravel](https://aregsar.com/blog/2020/connecting-to-mysql-and-redis-from-laravel)

```php
try {
                $db = DB::connection()->getPdo();
            }
            catch (PDOException $e) {
                self::fatal(
                    "An error occurred while connecting to the database. ".
                    "The error reported by the server was: ".$e->getMessage()
                );
            }
```