# Laravel App With Mail Server In Docker

May 11, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this blog post I will show you how to setup a development mail server running in a docker container to capture emails sent by your Laravel application and view those emails in a local web dashboard.

## Creating the docker-compose file

First we will create a new `docker-compose.yml` file in the Laravel project root directory:

> Note: skip to the next section if you already have a `docker-compose.yml` file in the root directory

```bash
touch docker-compose.yml
echo 'version: "3.1"' >> docker-compose.yml
echo 'services:' >> docker-compose.yml
```

## Adding the mysql service to docker-compose

Next we will add the following `MailHog` mail server container service to the `docker-compose.yml` file:

```yaml
    mailhog:
      image: mailhog/mailhog:latest
      container_name: myapp-mailhog
      ports:
        - "8003:1025"
        - "8100:8025"
```

You will note that there are two port mappings. The first port 1025 that is mapped to the localhost port 8003 on the local host is the mail server IMAP port that the application sends emails to.

The second port 8025 that is mapped to the local host port 8100 is a web dashboard http port that we can navigate to in our web browser, to see and manage the emails that were received by the mail server.

> Note: As configured, the emails will not be persisted across docker container restarts.

You can view docker configurations for alternative to `Mailhog`, such as `MailCatcher` or `PaperCut` at the end of the article.

## Adding the environment variables for the mail service

As a last step we will update the environment variables and configuration for our Laravel project to send emails to our local `Mailhog` server running in docker.

First we will update mail server settings in the `.env` file of our Laravel project as shown below:

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

Note that we are using port 8003 where the mail server port is mapped to.

Next we will update the `config/mail.php` file configuration as shown below:

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

By default the registration is not configured to send an email verification link before activating the user account. We will configure it to send an email verification email to test our email configuration. 

Because of our email settings in the `.env` file, the email will be delivered to the `MailHog` server running in the docker container. We will be able to view the email by navigating to the the `MailHog` server dashboard URL.

Before we begin we must add the appropriate UI scaffolding to our project with the `--auth` flag to add the authentication controllers and routes.

Instructions to do so can be found at [Create Laravel Project With Tailwind UI and Auth](https://aregsar.com/blog/2020/create-laravel-project-with-tailwind-ui-and-auth/)

Now we are ready and we can setup email verification to send emails by following the steps below:

### Step 1: Implement the MustVerifyEmail contract

Implement the MustVerifyEmail contract in the User model by opening the `App/User.php` file and adding `implements MustVerifyEmail` to the `class User extends Authenticatable` class declaration.

The modified version of the class declaration should look like:

```php
class User extends Authenticatable implements MustVerifyEmail
```

> Note the file already includes the `Illuminate\Contracts\Auth\MustVerifyEmail` namespace.

### Step 2: Add email verification routes

Add email verification routes by opening the `app/routes/web.php` file and adding the `['verify' => true]` parameter to the `Auth::routes();` method.

The modified version of the auth routes method should look like:

```php
Auth::routes(['verify' => true]);
```

## Step 3: Add the email verification middleware

> Note: This step is required to block routes to users with un-verified email. However it is not needed for testing our email configuration. So if you are just experimenting with configuring your email in the project, you can skip this section.

In order to make sure routes that need validated email will block access to the route if a user has not validated their email we need to add the `verified` middleware to those routes to protect them.

So in this final step we will add the middleware to the home controller
which will check any requests to routs mapped the home controller and will redirect users that have not confirmed their email to a page indicating that the user is not verified with an option to resend a verification email.

Below we have added the `verified` middleware in HomeController constructor:

```php
public function __construct()
{
    $this->middleware('auth','verified');
}
```

Alternatively we could have added the `verified` middleware to routes in our application routes file.

## Testing the user verification email

To test the user verification email we first have to startup our redis and mail server containers by typing the following docker command:

```bash
docker-compose up -d
```

Next we have to run our Laravel project and register new user with any arbitrary email address.

This should trigger a verification email that will be sent directly as part of the registration flow.
By default verification emails are not queued.

Now we can navigate our browser to `mailhog` dashboard at `http://localhost:8100` where you should be able to view and open the user verification email.

Click the verification link, which should verify the user registration through the application showing the user as logged in.

## Using MailCatcher or PaperCut instead of MailHog server

You can swap out `MailHog` for `MailCatcher` using the `docker-compose` configuration below:

```yaml
    mailcatcher:
        image: schickling/mailcatcher:latest
        container_name: myapp-mailcatcher
        ports:
            - 8003:1025
            - 8100:1080
```

Or you can swap out `MailHog` for `PaperCut` using the `docker-compose` configuration below:

```yaml
    papercut:
      image: jijiechen/papercut:latest
      container_name: myapp-papercut
      ports:
        - "8003:25"
        - "8100:37408"
```

## Using the Mailtrap hosted service instead of the docker mail server

If you prefer to use the [Mailtrap](https://mailtrap.io) hosted service instead of running a docker mail server, you can change the following environment variables in the `.env` file as shown below:

```ini
MAIL_DRIVER=smtp  
MAIL_HOST=smtp.mailtrap.io  
MAIL_PORT=2525  
MAIL_USERNAME=<YOUR_MAILTRAP_USERNAME>  
MAIL_PASSWORD=<YOUR_MAILTRAP_PASSWORD>  
MAIL_ENCRYPTION=tls
```

You will need to setup a [Mailtrap](https://mailtrap.io) account.
