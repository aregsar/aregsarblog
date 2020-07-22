# How Laravel Facades Work Under The Hood

July 21, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Laravel Facade classes

The Laravel Facade classes are static wrappers around the framework service class instances resolved out of the Laravel container.

Framework service classes are the classes that provide core framework features such as routing and caching. Facade classes give us a convenient way to access methods of these service classes using static class method semantics.

Even though the method calls on the Facade classes appear to be static methods, they are actually dynamic methods that resolve, from the container, an instance of a service class that actually implements the method. Then the method is called on the resolved instance of the service class.

## How facade service class resolution works

Every facade class must implement the `getFacadeAccessor` method. This method will return a key string that will be used to resolve, from the container, the service class instance associated with the facade.

This key string can be the fully namespace qualified name of service class that we want to resolve from the container, can be the fully namespace qualified name of the contract interface that the service class implements or can be an alias string, for the service class, called `Service Container Binding` key.

The important thing to note is that in order for the service to be able to be resolved, the framework must have registered the service that needs to be resolved using the key string that we return from the facade `getFacadeAccessor` method.

The list of fully namespace qualified class name and alias key mappings for Laravel Facade classes can be found in the Laravel Facades documentation at [https://laravel.com/docs/7.x/facades#facade-class-reference](https://laravel.com/docs/7.x/facades#facade-class-reference)

As an example I am showing below the App and Auth facade class mappings from the documentation:

| Facade | class                             | Service Container Binding
| ------ | --------------------------------- | -------------------------
| App    | Illuminate\Foundation\Application | app
| Auth   | Illuminate\Auth\AuthManager       | auth

Looking at the the `App` facade we can see that it resolves an instance of `Illuminate\Foundation\Application` service class. There is also a Service Container Binding alias called `app` registered with the container for that service class.

So if we look at the source code of the `App` facade class we can see that its `getFacadeAccessor` method returns the string `app` which will be used to resolve the `Illuminate\Foundation\Application` service class out of the container.

## The Request facade

As an example of how facades work under the hood, lets look at the `Illuminate\Support\Facades\Request` facade class shown below:

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

The comments at the top of the every facade class implemented by the framework will have a list of methods that you can call on that facade class. These are listed using the `@method` attribute. The comments also list the service class that will be resolved that those methods are defined in. This is listed using the `@see` attribute.

I am showing only one of the `@method` comments of the `Illuminate\Support\Facades\Request` facade which shows that `static void flush()` method is one of the methods that we can call on the Request facade. Also the `@see` attribute shows that the Request facade resolves the `Illuminate\Http\Request` service class that is the class that defines the actual `static void flush()` method that will be called when we call `Request::flush()`.

As you can see The Request facade only contains the required implementation of `getFacadeAccessor`.

Now Let's see what happens when the `Illuminate\Support\Facades\Request::flush()` method is called on the Request facade.

Since there is no implementation for that method and it is a static method call, the static `__callStatic` method of the base `Illuminate\Support\Facades\Facade` class will be called. This is where all the magic of the facade class happens and will be covered it the next section.

## The Facade base class

The base `Illuminate\Support\Facades\Facade` class that the `Illuminate\Support\Facades\Request` facade extends is where the service class associated with the facade is resolved and then its method invoked.

A partial implementation of the class is shown below:

```php
namespace Illuminate\Support\Facades;

use Closure;
use Mockery;
use Mockery\MockInterface;
use RuntimeException;

abstract class Facade
{
    protected static $app;

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

    public static function setFacadeApplication($app)
    {
        static::$app = $app;
    }
}
```

As I detailed in the previous section the `__callStatic` method is called when we call the `Request::flush()` method. When this method is called the action method name `flush` is passed to it as the first parameter. If the flush method had parameters, then those parameters would have been passed as the second parameter of the  `__callStatic` method. But in this case the second parameter will be null or empty.

The `__callStatic` method first calls the `static::getFacadeRoot()` method of the base facade class to resolve the instance of the actual `Illuminate\Http\Request` service class that implements the `flush` method.

Once the `Illuminate\Http\Request` instance is returned, the `__callStatic` method calls the `flush` method on that instance by calling `$instance->$method(...$args)` which uses the method parameter `$method` to call the `flush` method.

## How getFacadeRoot resolves the service class

As we saw in the previous section, when the `__callStatic` method calls the `static::getFacadeRoot()` of the base `Illuminate\Support\Facades\Facade` class, the `Illuminate\Http\Request` service class is resolved. The way that happens is described below:

The `static::getFacadeRoot()` method of the base Facade class calls the `static::resolveFacadeInstance` method of the base Facade class passing to the method the result of the call to the `static::getFacadeAccessor` method.

In other words `static::getFacadeRoot()` first calls `static::getFacadeAccessor`, then calls `static::resolveFacadeInstance` passing to it the resulting string returned by the `static::getFacadeAccessor` call.

The base `Illuminate\Support\Facades\Facade` class has a default implementation of `protected static function getFacadeAccessor()` method that is overridden by every facade class that extends it.

The `Illuminate\Support\Facades\Request` facade, that extends the base Facade class, overrides this method. The overridden implementation simply returns the string `'request'`.

So when the base Facade class `static::getFacadeRoot()` method calls `static::getFacadeAccessor`, it is actually calling the
`Illuminate\Support\Facades\Request` facade's overridden implementation of `static::getFacadeAccessor` which returns the string `'request'`.

This `'request'` string is the same string as the string used to register an alias for the `Illuminate\Http\Request` service class with the application container.

You can see this string listed in the Facade list mappings for the `Request` facade in the Laravel Facade documentation [https://laravel.com/docs/7.x/facades#facade-class-reference](https://laravel.com/docs/7.x/facades#facade-class-reference) that I previously mentioned.

So to reiterate, the `static::getFacadeRoot()` of the base Facade class calls the overridden `static::getFacadeAccessor` method of the `Illuminate\Support\Facades\Request` class that returns the string `'request'` that the `static::getFacadeRoot()` method then passes to the `static::resolveFacadeInstance` method of the base Facade class.

Inside the method, the `return static::$resolvedInstance[$name] = static::$app[$name];` line is executed. This line resolves the `Illuminate\Http\Request` service class from the `static::$app` container using the `'request'` string passed in as the `$name` parameter. It sets the resolved instance into the `static::$resolvedInstance` array using the same `$name` parameter then returns the resolved instance.

> Note: sometimes we already have access to a resolved instance of the service class. This instance is passed as the $name parameter to resolveFacadeInstance so it is simply returned via the line return $name; at the top of the resolveFacadeInstance method

As stated in the previous section, once the instance is returned, the `flush` method called on the facade is called on the instance.

> A note on the alias string returned by getFacadeAccessor. This is the string used to bind an alias for the Illuminate\Http\Request service class in the Laravel container. So that it can be resolved out of the container using that alias string. That is not to be confused with Facade alias strings that are short strings mapped to Facade classes so that a Facade class can be referenced using its alias string instead of requiring the fully  namespace qualified Facade class name.

