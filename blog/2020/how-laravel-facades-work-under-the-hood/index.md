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

As you can see it only contains the required overridden implementation of `getFacadeAccessor`.

So when for example the `Request::flush()` method is called on the Request facade there is no implementation for that method.

In this scenario, the static `__callStatic` method of its extended `Facade` class will be called.

The `Facade` class and the effect of the `__callStatic` method call are covered in the next section.

> Note: The comments at the top of the facade class will have a list of methods that you can call on that facade class. The comments also list the service class that will be resolved that those methods are defined in. As an example the listed `flush()` method is one of the methods that we can call on the Request facade and we can see that the Request facade resolves the `Illuminate\Http\Request` service class that defines the `flush()` method

## The Illuminate\Support\Facades\Facade base class

The base `Illuminate\Support\Facades\Facade` class that the `Request` facade extends is where the service class associated with the facade is resolved and its method invoked.

The class is shown below:

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

As we detailed in the previous section the `__callStatic` method is called when we call the `Request::flush()` method. When this method is called the action method name `flush` is passed to it as the first parameter. If the flush method had parameters, then those parameters would have been passed as the second parameter of the  `__callStatic` method. But in this case the second parameter will be null or empty.

The `__callStatic` method first calls the `static::getFacadeRoot()` method of the base facade class to resolve the instance of the actual `Request` service class that implements the `flush` method.

Then once it has the instance, it calls the `flush` method on the instance by calling `$instance->$method(...$args)` which uses the method parameter `$method` to call the `flush` method passing in any argument if they exist by passing in the `$args` parameter passed to `__callStatic`.

## How getFacadeRoot resolves the service class

When the `__callStatic` method calls the `static::getFacadeRoot()` the `Request` service class is resolved. The way that happens is described below:

The base Facade class has a default implementation of `protected static function getFacadeAccessor()` method that is overridden by every facade class that extends it.

The `Request` facade overrides this method that simply return the string `request`. This string is that same as the string used to register an alias for the Request service class with the application container.

The `getFacadeRoot()` method of the Facade class calls the `static::resolveFacadeInstance` method of the Facade class passing the method the result of the call to the `static::getFacadeAccessor` method of the facade class.

In other words, the `static::resolveFacadeInstance` method of the facade class is invoked and passed in as its argument the `request` string returned by the overridden `static::getFacadeAccessor` method.

So the `protected static function resolveFacadeInstance($name)` is called with the `$name` parameter value of `request`.

Inside the method, the `return static::$resolvedInstance[$name] = static::$app[$name];` line is executed that resolves the `Request` service class and return the instance.

That line does two things. First it calls `static::$app[$name]` to resolve the instance from the `static::$app` container using the `$name` string parameter which in this case is `request`.

Then it sets the resolved instance of the `Request` service class to the `static::$resolvedInstance[$name]` array so that the next time the facade tries to access the `Request` service class it can short cut return it using the previous `return static::$resolvedInstance[$name]` line if it exists in the array.

Finally the resolved instance in `static::$resolvedInstance[$name]` is returned.

> Note: sometimes we already have access to a resolved instance of the service class. This instance is passed as the $name parameter to resolveFacadeInstance so it is simply returned via the line return $name; at the top of the resolveFacadeInstance method

===============

```php
namespace Illuminate\Support\Facades;

abstract class Facade
{

    protected static $app;

    public static function setFacadeApplication($app)
    {
        static::$app = $app;
    }
}
```
