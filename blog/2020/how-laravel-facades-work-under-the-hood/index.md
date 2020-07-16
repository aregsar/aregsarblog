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

