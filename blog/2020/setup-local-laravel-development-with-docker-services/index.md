# Setup Local Laravel Development With Docker Services

April 23, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I will show how I run Docker services that support my locally installed Laravel applications.

I run the following services in individual Docker containers to support local Laravel application development:

+ MySQL database
+ Redis used for session, cache and queued job storage
+ MailHog Mail server to capture emails and view in dashboard
+ Elastic Search

> Note: Except for MailHog, these are the same services I use in production to maintain dev/prod parity

All the containers are run via Docker compose.

