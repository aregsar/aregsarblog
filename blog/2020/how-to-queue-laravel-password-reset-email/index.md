# How To Queue Laravel Password Reset Email

May 14, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this article I will show you how to change the out of the box Laravel configuration to queue password reset emails.

For queuing User Verification Emails see:

[How To Queue Laravel User Verification Email](https://aregsar.com/blog/2020/how-to-queue-laravel-user-verification-email)

## Queuing the Password Reset email

The Laravel authentication framework sends a non queued password reset notification as part of the password reset flow.
The password reset notification sends the email by calling the `sendPasswordResetNotification()` method on the `App\User` class.

The `sendPasswordResetNotification` method is inherited by the `App\User` class from the `Illuminate\Foundation\Auth\User` class (aliased as `Authenticatable`) which the `App\User` class extends:

```php
namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

//we are extending the Illuminate\Foundation\Auth\User class that is aliased to Authenticatable above
class User extends Authenticatable
{
    use Notifiable;
}
```

The `Illuminate\Foundation\Auth\User` class implements the `Illuminate\Auth\CanResetPassword` trait that in turn contains the default implementation of the `sendPasswordResetNotification` method:

```php
namespace Illuminate\Foundation\Auth;

class User extends Model implements
    AuthenticatableContract,
    AuthorizableContract,
    CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword, MustVerifyEmail;
}
```

Here is the default `sendPasswordResetNotification` implementation in the `CanResetPassword` trait:

```php
namespace Illuminate\Auth\Passwords;

use Illuminate\Auth\Notifications\ResetPassword as ResetPasswordNotification;

trait CanResetPassword
{
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
}
```

Because `sendPasswordResetNotification` method is included as a trait in the extended `Illuminate\Foundation\Auth\User` class, we can override its default implementation by simply implementing a `sendPasswordResetNotification` method in the `App\User` class with our overridden implementation that will queue the delivery of reset notification email.

We can take one of two approaches for the implementation within the overridden `sendPasswordResetNotification` method:

Approach 1 - We can queue the notification

Approach 2 - We can dispatch a queued job that sends the notification

These approaches are shown in the next two sections.

## Approach 1 - Queuing the notification approach

With this approach we will extend the `Illuminate\Auth\Notifications\ResetPassword` notification that the `Illuminate\Auth\CanResetPassword` trait sends, into a queue-able notification. Then we will queue this extended notification from the overridden `sendPasswordResetNotification` method implementation that we will add to the `App\User` class.

So first let's create, in the `\App\Notifications\Auth` directory, a new `QueuedResetPassword` notification that extends the `Illuminate\Auth\Notifications\ResetPassword` notification:

```bash
artisan make:notification Auth/QueuedResetPassword
```

This command creates the class `\App\Notifications\Auth\QueuedResetPassword`.

Next we make this class extend the `Illuminate\Auth\Notifications\ResetPassword` class
and implement the `Illuminate\Contracts\Queue\ShouldQueue` contract.

Finally we add the `Illuminate\Bus\Queueable` trait to the body of the class.

That is all we need to do to make a new `QueuedResetPassword` notification that is queue-able.

Below is the our final `QueuedResetPassword` notification class:

```php
namespace App\Notifications\Auth;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;
use Illuminate\Auth\Notifications\ResetPassword;

class QueuedResetPassword extends ResetPassword implements ShouldQueue
{
    use Queueable;
}
```

Now we need to add the `sendPasswordResetNotification()` method to the `App\User` class which will override the default method that it gets from the `CanResetPassword` trait of its parent class. Then we can notify the user using the `QueuedResetPassword` notification to substitute our new queued notification instead of the original notification.

```php
class User extends Authenticatable
{
    use Notifiable;

    //Overrideen sendPasswordResetNotification implementation
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new \App\Notifications\Auth\QueuedResetPassword($token));
    }
}
```

Optionally we can add a constructor to the `QueuedResetPassword` class to queue the notifications to a different queue and/or different connection from the default connection and queue.

```php
class QueuedResetPassword extends ResetPassword implements ShouldQueue
{
    use Queueable;

    public function __construct()
    {
        //uncomment to override the queue
        //$this->queue = 'reset';

        //uncomment to override the connection
        //$this->connection = 'reset';
    }
}
```

## Approach 2 - Queueing a job that sends the notification approach

In this approach we create a queued job that we dispatch in the overridden `sendPasswordResetNotification` method of the `App\User` class.

When the job is processed, it will send the original `Illuminate\Auth\Notifications\ResetPassword` notification that was being sent directly from the overridden `sendPasswordResetNotification` method.

So first we need to create a job that will be queued.

```bash
artisan make:job QueuedPasswordResetJob
```

Next in the `handle()` method of the job we need to copy a slightly modified implementation from the original `sendPasswordResetNotification()` method as shown below:

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Auth\Notifications\ResetPassword;
use App\User;

class QueuedPasswordResetJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $user;
    protected $token;

    public function __construct(User $user, $token)
    {
        //the user property passed to the constructor through the job dispatch method
        $this->user = $user;
        $this->token = $token;
    }

    public function handle()
    {
        //This queued job sends
        //Illuminate\Auth\Notifications\ResetPassword notification
        //to the user by triggering the notification
        $this->user->notify(new ResetPassword($this->token));
    }
}
```

Finally we need to override the default `sendPasswordResetNotification()` method implementation by adding a `sendPasswordResetNotification()` method directly to the `App\User` class where we will dispatch the `QueuedPasswordResetJob` to the queue.

```php
class User extends Authenticatable
{
    public function sendPasswordResetNotification($token)
    {
        //dispactches the job to the queue passing it this User object
        QueuedPasswordResetJob::dispatch($this,$token);
    }
}
```

Looking back at the `QueuedPasswordResetJob` class we can see that the implementation we added to its  `handle()` method is slightly different than the implementation of the original `sendPasswordResetNotification` method.

The `handle()` method of the job class calls `$this->user->notify` where the original `sendPasswordResetNotification` method in the `CanResetPassword` trait calls `$this->notify`.

This is because in the original implementation, the `$this` pointer refers to the `App\User` class that includes the trait. But in the Job class the `$this` pointer refers to the job class so we need to reference the `user` property of the job class then call the `notify` method on the user.

## Customizing the redirect location after reseting password

By default the user will be automatically logged in and redirected to the `/home` URL after reseting their password, to override this we can define a `redirectTo` property on the `Auth\ResetPasswordController` like so:

```php
protected $redirectTo = '/mynewhome';
```

This will then override and redirect them to the `/mynewhome` URL instead.
