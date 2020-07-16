# How Laravel Facades Work Under The Hood

July 17, 2020 by [Areg Sarkissian](https://aregsar.com/about)

July 16, 2020

[How Laravel Facades Work Under The Hood](https://aregsar.com/blog/2020/how-laravel-facades-work-under-the-hood)

## Laravel Facade classes

The Laravel Facade classed are static wrapper around the framework `service class` instances resolved out of the Laravel container.

Laravel services classes are registered with the Laravel container to be resolved and used by the application during request execution by calling the methods on the resolved instances.

While these services can be resolved directly out of the container a more convenient approach is to directly call the methods that we would call on the resolved instance, directly on the facade class as a static method call.

Under the hood the call to the static facade method will resolve the  service class instance that is registered with the facade and call the same method on the resolved instance of the service class.

## Facade service class registration

Every facade class must implement the `getFacadeAccessor` method. This method will register a key string that is used to resolve the service class instance associated with the facade, from the Laravel container.

This key string can be the fully namespace qualified name of service class or contract that we want to resolve from the container or can be an alias string that is registered with the container for that service class.

The important thing to note is that in order for the service to be able to be resolved, the framework must have registered the service that needs to be resolved using the key string that we specify in the facade `getFacadeAccessor` method.

## The Request facade

As an example lets look at the the `Illuminate\Support\Facades\Request` facade class shown below:

```php
namespace Illuminate\Support\Facades;

 /**
 * @method static void flush()
 * @see \Illuminate\Http\Request
 */
class Request extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor()
    {
        return 'request';
    }
}
```


## The Illuminate\Support\Facades\Facade base class

The base `Illuminate\Support\Facades\Facade` class that the `Request` facade extends is where the service class associated with the facade is resolved and its method invoked.

```php
namespace Illuminate\Support\Facades;

use Closure;
use Mockery;
use Mockery\MockInterface;
use RuntimeException;

abstract class Facade
{
    /**
     * The application instance being facaded.
     *
     * @var \Illuminate\Contracts\Foundation\Application
     */
    protected static $app;

    /**
     * The resolved object instances.
     *
     * @var array
     */
    protected static $resolvedInstance;

    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        return $instance->$method(...$args);
    }

    public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }


    protected static function getFacadeAccessor()
    {
        throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
    }

    protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        if (static::$app) {
            return static::$resolvedInstance[$name] = static::$app[$name];
        }
    }


}
```

The base Facade class has a default implementation of `protected static function getFacadeAccessor()` method that is overriden by every facade class that extends it.

The Request facade overrides this method that simply return the string `request`. This string is that same as the string used to register an alias for the Request service class with the application container.

The `getFacadeRoot()` method of the Facade class calls the `static::resolveFacadeInstance` method of the Facade class passing the method the result of the call to the `static::getFacadeAccessor` method of the facade class.

In other words, the `static::resolveFacadeInstance` method of the facade class is invoked and passed in as its argument the `request` string returned by the overridden `static::getFacadeAccessor` method.

So the `protected static function resolveFacadeInstance($name)` is called with the `$name` parameter value of `request`.

Inside the method, one of the methods that resolve the `Request` service class is called.