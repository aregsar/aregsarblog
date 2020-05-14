# How To Queue Laravel User Verification Email

May 14, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[How To Queue Laravel User Verification Email](https://aregsar.com/blog/2020/how-to-queue-laravel-user-verification-email)

## Sending User Verification Email

I have instructions here that sets up a mail server in a docker container and tests it by configuring the Laravel user verification email here:

[Laravel App With Mail Server In Docker](https://aregsar.com/blog/2020/laravel-app-with-mail-server-in-docker)

In this article I will show you how to change that configuration to queue the user verification email.

## Enabling Email verification

I will first quickly repeat the steps required to setup a verification email.

make the User class implement `MustVerifyEmail`

`class User extends Authenticatable implements MustVerifyEmail`

make the Auth::routes method take a `['verify' => true]` input argument
`Auth::routes(['verify' => true]);`

protect any routes that use the `auth` middleware that we want to check the authenticated user has verified their email with the `verified` middleware by adding it to the controller constructor or to the route. 

Below is an example of adding it to a controller constructor. 

```php
public function __construct()
{
    $this->middleware('auth','verified');
}
```

## Queuing the verification email

By default Laravel triggers a notification when a user completed the registration step to send a verification email when the `Auth::routes` method has this feature enabled via the  `['verify' => true]` parameter.

This notification directly sends the email as part of the web request.

In order to queue the delivery of this email we can take one of two approaches:

1 We can queue the notification
2 We can dispatch a queued job that sends the notification


## Queuing the notification

Create a new notification that extends the verify email notification:

```bash
artisan make:notification Auth/QueuedVerifyEmail
```

Make this class extend ` Illuminate\Auth\Notifications\VerifyEmail`
and implement `Illuminate\Contracts\Queue\ShouldQueue`

Also add the `Illuminate\Bus\Queueable` trait to the body of the class.

That is all we need to do to make a new notification based on the frameworks notification that is queue-able.

The last this we need to do is to add the `sendEmailVerificationNotification()` method to the User class which will override the default method that the user class gets from the trait to substitute our queued notification instead of the original notification.

```php
namespace App\Auth\Notifications;

use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Bus\Queueable;

class QueuedVerifyEmail extends VerifyEmail implements ShouldQueue
{
    use Queueable;
}


public function sendEmailVerificationNotification()
{
    $this->notify(new \App\Notifications\Auth\QueuedVerifyEmail);
}
```

## Queueing a job that sends the notification



