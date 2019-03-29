# How to setup a custom domain for your github pages blog in five minutes

Mar 17, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

[Part 6 - Setup third party services for your github pages blog](https://aregsar.com/blog/2019/how-to-setup-third-party-services-for-your-github-pages-blog)

[Part 7 - Github pages tech stack: Jekyll, Liquid, Markdown](https://aregsar.com/blog/2019/github-pages-tech-stack-jekyll-markdown-liquid)

Welcome to part 4 of this series on setting up a blog with Github pages.

In this post in the series I will show you how I published this blog under the aregsar.com domain name by registering the custom domain name and then setting up the proper DNS records to point the custom domain name to the location that Github pages hosts the blog.

This process required four easy steps that took approximately 5 minutes to complete.

## Step 1 - Registered a custom domain name

To setup a custom domain for this blog, I registered the custom domain name `aregsar.com` with my domain registrar dnsimple.com.

## Step 2 - Added custom domain name to my Github pages repository

I entered the custom domain `aregsar.com` into the custom domain name text box, in the Github pages section of my repository settings page, then clicked save.

After doing so, the following message displayed at the top of the Github pages section section.

`Your site is ready to be published at http://aregsar.com/.`

At this point Github pages adds a file named `CNAME` to the root of my remote repository. This file contains a single line which is the domain name `aregsar.com` that I added in the custom domain text box.

As always I did a `git pull` to get the `CNAME` file that was added to the remote repository.

Since I had not configured DNS records for the custom domain yet, when the site is re-published because of the setting change, the message changed to:

`Domain's DNS record could not be retrieved.`

You have to refresh the repository settings page to see the changed message.

## Step 3 - Pointing custom domain to Github pages

To Allow the DNS records to be retrieved we need to add them and map them to the Github pages published URL for our blog.

> Caution: Before configuring your custom domain with your DNS provider, add your custom domain to your GitHub Pages site as detailed in step 2. Configuring your custom domain with your DNS provider without adding your custom domain to GitHub could result in someone being able to host a site on one of your subdomains. For more info see: [https://help.github.com/en/articles/quick-start-setting-up-a-custom-domain](https://help.github.com/en/articles/quick-start-setting-up-a-custom-domain)

Using my domain regsitrars domain name administration panel, I added the neccessary DNS records to point my custom domain `aregsar.com` to the location where Github hosts my GitHub pages blog.

> Since my Github username is `aregsar`, the location that Github pages hosts my blog is `aregsar.github.io`. For your own site it will publish to `<your-github-username>.github.io`.

First I added the `ALIAS` record for the root domain `aregsar.com`:

`ALIAS aregsar.com -> aregsar.github.io`

Next I added a `CNAME` record for the `www` sub domain.

`CNAME www.aregsar.com -> aregsar.github.io`

As you can see I added an ALIAS record for root domain `argesar.com` mapped to `aregsar.github.io` which is where github pages hosts my GitHub pages site.

I also added a CNAME record for the subdomain `www.aregsar.com` also mapped to `aregsar.github.io`.

This way I can navigate to the blog using `www.aregsar.com` and Github pages redirects to the domain root `aregsar.com`.

Github pages recomends adding a `www` sub domain and will automatically redirect it to the domain root. See [here](https://help.github.com/en/articles/about-supported-custom-domains#www-subdomains) for more details.

> Note: your registrar may not support ALIAS records. In that case follow the instructions at [https://help.github.com/articles/setting-up-an-apex-domain/](https://help.github.com/articles/setting-up-an-apex-domain/) to add four `A` records instead that point to the IP address of the Github pages servers. The CNAME record should still map to `<your-github-username>.github.io` in this scenario.

Refreshing the repository settings page you can see that the publishing message at the top of Github pages section eventually changes to:

`Your site is published at http://aregsar.com/`

Note the URL is not secured since we have not selected the https setting yet. We will do that in the next step.

## Step 3 - Enforcing HTTPS

I removed and then re-added the custom domain `aregsar.com` to the custom domain text box in the Github pages section of the repository settings page to trigger the process of enabling HTTPS.

> Note: you must click the save button after you clear the text box and then add the custom domain name back in and click the save button again.

At this point the Enforce HTTPS checkbox was automatically checked for me. If that is not the case for you, click the Enforce HTTPS checkbox if it is not checked to publish the blog using an Https URL.

After checking the Https checkbox the, published URL endpoint message at the top of the github pages section changed to the message:

`Your site is ready to be published at https://aregsar.com/`

A few moments later when the site was re-published because of the settings change, the message changed to:

`Your site is published at https://aregsar.com/`

This can be seen after refreshing the repository settings page.

After this final step I was able to navigate to `aregsar.com` or `www.aregsar.com` and see my blog home page.

## Conclusion

In this process I showed you the process of setting up a custom domain name for this blog. You can do the same using your own custom domain name for your own Github pages blog.

Check out the next post in the series where I show you how I set up the file structure of this blog.

Thanks for reading.

[Top](#how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)
