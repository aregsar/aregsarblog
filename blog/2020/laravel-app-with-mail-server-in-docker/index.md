# Laravel App With Mail Server In Docker

May 11, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Creating the docker-compose file

Create a new docker-compose.yml file in the Laravel project root directory:

> Note: skip to the next section if you already have a docker-compose.yml file in the root directory

```bash
touch docker-compose.yml
echo 'version: "3.1"' >> docker-compose.yml
echo 'services:' >> docker-compose.yml
```

## Adding the mysql service to docker-compose

We can use MailHog with the configuration below:

```yaml
    mailhog:
      image: mailhog/mailhog:latest
      container_name: myapp-mailhog
      ports:
        - "8003:1025"
        - "8100:8025"
```

You will note that there are two port mappings. The mapping that is exposed on port 8003 on the local host is the mail server IMAP server port that the application dispatches emails to.
THe other port that is exposed on the local host port 8100 is a  email dashboard http port that we can navigate to, to see and manage the emails that were received by the mailserver.

> Note: as configured, the emails will not be persisted across container restarts.

You can see alternative configurations for MailCatcher or PaperCut at the end of the article.

## Adding the environment variables for the mail service

Update the env vars in .env file to:

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=8003
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"
```

The `config/mail.php` file is where these env vars are used:

```php
'mailers' => [
      'smtp' => [
          'transport' => 'smtp',
          'host' => env('MAIL_HOST', 'localhost'),
          'port' => env('MAIL_PORT', 8003),
          'encryption' => env('MAIL_ENCRYPTION', 'tls'),
          'username' => env('MAIL_USERNAME'),
          'password' => env('MAIL_PASSWORD'),
          'timeout' => null,
      ],
],
  'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'admin@myapp.com'),
        'name' => env('MAIL_FROM_NAME', 'Admin'),
    ],
```

## Sending new user verification emails to MailHog

Laravel comes with a built in user registration and authentication system.
By default the registration is not configured to send an email verification link before activating the user account.

We will configure it to send an email verification email. Because of our email settings in the .env file, the email will be delivered to the MailHog server running in the docker container. We will be able to view the email by navigating to the the MailHog server dashboard URL.

Before we begin we must add the appropriate UI scaffolding with the --auth flag to add the authentication controllers and routes.

Instructions to do so can be found at [Create Laravel Project With Tailwind UI and Auth](https://aregsar.com/blog/2020/create-laravel-project-with-tailwind-ui-and-auth/)

### Step 1: Implement the MustVerifyEmail contract

Implement the MustVerifyEmail contract in the User model by opening the App/User.php file and adding `implements MustVerifyEmail` to the
`class User extends Authenticatable` class declaration.

The modified version of the class declaration should look like:

`class User extends Authenticatable implements MustVerifyEmail`

### Step 2: Add email verification routes

use Illuminate\Contracts\Auth\MustVerifyEmail;

Add email verification routes by opening app/routes/web.php file and adding `['verify' => true]` parameter to the auth routes `Auth::routes();` method.

The modified version of the auth routes method should look like:

`Auth::routes(['verify' => true]);`

## Step 3: Add the email verification middleware

In order to make sure routes that need validated email will block access to the route if the authenticated user has not validated their email we need to add the `verified` middleware to those routes to protect them.

So in this final step we will add the middleware to the home controller
which will check any requests to routs mapped the home controller to see if the user has verified their email. 

If not the middleware will redirect to a page indicating that the user is not verified with an option to resent a verification email.

Add the `verified` middleware to the middleware in the HomeController constructor:

The modified version of the constructor should look like:

```php
public function __construct()
{
    $this->middleware('auth','verified');
}
```

> Note: In order for the `verified` middleware to take action, the auth middleware must authenticate the user so the user can be inspected to see if it is verified. So the `auth` middleware must run before the `verified` middleware. Obviously if the user is not authenticated then by default they are not verified either and they will not even get to the `verified` middleware because the `auth` middleware will redirect them to a login page.
Also if a rout is not covered by an `auth` middleware then it makes no sense to have a `verified` because anonymous users don`t have registered email to be verified.

## Testing the user verification email

Run `docker-compose up -d` then startup a your Laravel project and register a user.

This should trigger a verification email, so navigate to `localhost:8100` where you should be able to open the email.

Click the verification link, which should verify the user registration through the application showing the user as logged in.

## Using MailCatcher or PaperCut instead of MailHog server

You can swap out MailHog for MailCatcher using configuration below:

```yaml
    mailcatcher:
        image: schickling/mailcatcher:latest
        container_name: myapp-mailcatcher
        ports:
            - 8003:1025
            - 8100:1080
```

Or you can swap out MailHog for PaperCut using configuration below:

```yaml
    papercut:
      image: jijiechen/papercut:latest
      container_name: myapp-papercut
      ports:
        - "8003:25"
        - "8100:37408"
```

## Using the mailtrap service instead of MailHog

If you prefer to use the Mailtrap service free tier instead of running a docker mail server, you can change the following env vars in the .env file as set below:

```ini
MAIL_DRIVER=smtp  
MAIL_HOST=smtp.mailtrap.io  
MAIL_PORT=2525  
MAIL_USERNAME=<YOUR_MAILTRAP_USERNAME>  
MAIL_PASSWORD=<YOUR_MAILTRAP_PASSWORD>  
MAIL_ENCRYPTION=tls
```

You will need to setup a Mailtrap.io account.
