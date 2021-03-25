# Configure Laravel Local Mail Server

January 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)

This tutorial shows how to quickly setup a mailhog docker service for local email delivery testing in your Laravel project.

The only prerequisite is to have docker installed.

To start go to the root of your Laravel project.

Step 1 -Setup the mailhog sever docker service

```bash
echo docker-compose.yml << EOL
version: "3.1"
services:
    mailhog:
      image: mailhog/mailhog:latest
      container_name: app-mailhog
      ports:
        - "${MAIL_PORT}:1025"
        - "8025:8025"
EOL
```

> If you already have a docker-compose.yml file in the project root then just add the mailhog service portion of the above yml code under the services section.

Step 2- Open the project `.env` file and update the connection settings for the mailhog local mail service.

```ini
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=8003
```

> docker-compose.yml also uses settings from the .env file

Step 3 - Startup the docker service

```bash
docker-compose up
```

Step 4 - Navigate to the mail server dashboard

The Mailhog service exposes a dashboard for viewing and managing delivered emails at port 8025 as configured in the docker-compose port mapping.

```bash
open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome http://localhost:8025
```

Now you are ready to test sending emails from your app.

When done you can shutdown the docker service

```bash
docker-compose down
```

> You can substitute any other preferred docker mail service for mailhog. All you need is to change the docker image and change port mapping to whatever port that service exposes its dashboard on.
