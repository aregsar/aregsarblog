# How To Queue Laravel User Verification Email

May 14, 2020 by [Areg Sarkissian](https://aregsar.com/about)

Also see:

[How To Queue Laravel Password Reset Email](https://aregsar.com/blog/2020/how-to-queue-laravel-password-reset-email)

## Sending User Verification Email

I have instructions here that sets up a mail server in a docker container and tests it by configuring the Laravel user verification email here:

[Laravel App With Mail Server In Docker](https://aregsar.com/blog/2020/laravel-app-with-mail-server-in-docker)

In this article I will show you how to change that configuration to queue the user verification email.

## Enabling Email verification

I will first quickly repeat the steps required to setup a verification email.

make the User class implement `Illuminate\Contracts\Auth\MustVerifyEmail`.
The `use Illuminate\Contracts\Auth\MustVerifyEmail;` statement is already included in the User.php file.
So all we have to do is add the `implements MustVerifyEmail` to the class declaration.

```php
class User extends Authenticatable implements MustVerifyEmail
```

Next make the `Auth::routes` method take a `['verify' => true]` input argument.

```php
Auth::routes(['verify' => true]);
```

Finally protect any routes that use the `auth` middleware that we want to check the authenticated user has verified their email with the `verified` middleware by adding it to the controller constructor or to the route.

Below is an example of adding it to a controller constructor:

```php
public function __construct()
{
    $this->middleware('auth','verified');
}
```

## Queuing the verification email

By default Laravel triggers a notification when a user completed the registration step to send a verification email when the `Auth::routes` method has this feature enabled via the  `['verify' => true]` parameter.

This notification directly sends the email as part of the web request by calling the sendEmailVerificationNotification() method on the User class. The sendEmailVerificationNotification method is included in the User class through the extended `Illuminate\Foundation\Auth\User` class:

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

The `Illuminate\Foundation\Auth\User` class implements the `MustVerifyEmail` trait that in turn contains the default implementation of the sendEmailVerificationNotification method

```php
class User extends Model implements
    AuthenticatableContract,
    AuthorizableContract,
    CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword, MustVerifyEmail;
}
```

Here is the default  sendEmailVerificationNotification implementation in the `MustVerifyEmail` trait:

```php
namespace Illuminate\Auth;

use Illuminate\Auth\Notifications\VerifyEmail;

trait MustVerifyEmail
{
    public function sendEmailVerificationNotification()
    {
        $this->notify(new VerifyEmail);
    }
}
```

Because sendEmailVerificationNotification method is included as a trait in the extended User class, we can override its default implementation by simply adding a sendEmailVerificationNotification method in the User class with our overridden implementation that will queue the delivery of this email.

Inside the overridden sendEmailVerificationNotification method we can take one of two approaches for the implementation within the sendEmailVerificationNotification method:

Approach 1-We can queue the notification
Approach 2-We can dispatch a queued job that sends the notification

These approaches are shown in the next two sections.

## Approach 1 - Queuing the notification approach

With this approach we will extend the `Illuminate\Auth\Notifications\VerifyEmail` notification, that the `MustVerifyEmail` trait sends, into a queue-able notification and queue this extended notification in the overridden `sendEmailVerificationNotification` method that we add to the User class.

So first we create a new notification that extends the VerifyEmail notification:

```bash
artisan make:notification Auth/QueuedVerifyEmail
```

This command creates the class `\App\Notifications\Auth\QueuedVerifyEmail`

Make this class extend ` Illuminate\Auth\Notifications\VerifyEmail`
and implement `Illuminate\Contracts\Queue\ShouldQueue`

Also add the `Illuminate\Bus\Queueable` trait to the body of the class.

That is all we need to do to make a new notification based on the frameworks notification that is queue-able.

```php
namespace App\Auth\Notifications;

use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;

class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;
}
```

The last thing we need to do is to add the `sendEmailVerificationNotification()` method to the User class which will override the default method that the `User` class gets from the trait to substitute our queued notification instead of the original notification.

```php
class User
{
    public function sendEmailVerificationNotification()
    {
        $this->notify(new \App\Notifications\Auth\QueuedVerifyEmail);
    }
}
```

Optionally add a constructor to the QueuedVerifyEmail class to queue verify the notifications to a different queue and/or different connection from the default connection and queue

```php
class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;

    public function __construct()
    {
        $this->queue = 'verify';
        //$this->connection = 'verify';
    }
}
```

## Approach 2 - Queueing a job that sends the notification approach

In this approach we create a queued job that we dispatch in the in the overridden sendEmailVerificationNotification of the User class. When the job is processed, it will send the original Illuminate\Auth\Notifications\VerifyEmail notification that was being sent directly from the sendEmailVerificationNotification method before.

So we first need to create a job that will be queued.

```bash
artisan make:job QueuedVerifyEmailJob
```

Next in the handle() method we need to copy the notification implementation that is in the original `sendEmailVerificationNotification()` method.

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Auth\Notifications\VerifyEmail;
use App\User;

class QueuedVerifyEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $user;

    public function __construct(User $user)
    {
        //the user property passed to the constructor through the job dispatch method
        $this->user = $user;
    }

    public function handle()
    {
        //This queued job sends
        //Illuminate\Auth\Notifications\VerifyEmail verification
        //to the user by triggering the notification
        $this->user->notify(new VerifyEmail);
    }
}
```

Finally we need to override the default `sendEmailVerificationNotification()` method implementation by adding a sendEmailVerificationNotification() method directly in User class.
In this method we dispatch the QueuedVerifyEmailJob to the queue.

```php
class User
{
    public function sendEmailVerificationNotification()
    {
        //dispactches the job to the queue passing it this User object
         QueuedVerifyEmailJob::dispatch($this);
    }
}
```

Note that the original implementation of sendEmailVerificationNotification was slightly different then the implementation in the handle method of the  QueuedVerifyEmailJob class.

In the `handle` method of the job class we call `$this->user->notify` where as the original sendEmailVerificationNotification method in the `MustVerifyEmail` trait calls `$this->notify`.
This is because in the `MustVerifyEmail` traits implementation of the sendEmailVerificationNotification method the $this pointer references the User class that includes the trait.
So in the job class we we reference the User and call notify on it.
