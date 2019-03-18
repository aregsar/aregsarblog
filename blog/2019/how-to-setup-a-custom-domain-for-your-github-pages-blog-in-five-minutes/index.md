# How to setup a custom domain for your github pages blog in five minutes

Mar 18, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

In this post in the series I will show you how I published this blog under the aregsar.com domain name by registering the custom domain name and then setting up the proper DNS records to point the custom domain name to the location that Github pages hosts the blog.

This process required four easy steps that took approximately 5 minutes to complete, most of which was waiting for the DNS records to propegate and Github pages to issue SSL certificates.

## Step 1 - Registered a custom domain name

To setup a custom domain for my blog, I registered the custom domain name `aregsar.com` with my domain registrar dnsimple.com.

## Step 2 - Pointed the custom domain name to my github pages site

Using my domain regsitrars domain name administration panel, I added the neccessary DNS records to point my custom domain aregsar.com to the location where Github hosts my GitHub pages blog.

First I added the `ALIAS` record for the root domain `aregsar.com`:

`ALIAS aregsar.com -> aregsar.github.io`

Next I added a `CNAME` record for the `www` sub domain. 

`CNAME www.aregsar.com -> aregsar.github.io`

As you can see I added an ALIAS record for root domain `argesar.com` mapped to `aregsar.github.io` which is where github pages hosts my GitHub pages site.

I also added a CNAME record for the subdomain `www.aregsar.com` also mapped to `aregsar.github.io`.

This way I can navgate to the blog using `www.aregsar.com` which gets redirected to the root domain `aregsar.com`.

Github pages recomends adding a `www` sub domain and will automatically redirect it to the root domain.

See the following link for more info:

[https://help.github.com/articles/using-a-custom-domain-with-github-pages/](https://help.github.com/articles/using-a-custom-domain-with-github-pages/)

> Note: your registrar may not support ALIAS records. In that case follow the instructions at [https://help.github.com/articles/setting-up-an-apex-domain/](https://help.github.com/articles/setting-up-an-apex-domain/) to add four `A` records instead that point to the IP address of the Github pages servers. The CNAME record should still map to `<your-github-username>.github.io` in this case.

## Step 3 - Connected the custom domain to my Github pages repository

I entered the custom domain `aregsar.com` into the custom domain name text box, in the github pages section of my repository settings page, then clicked save.

> Note: wait for DNS record propagation to take effect if you see any github pages error messages after saving the custom root domain name. Any error should resolve automatically in a few minutes.

At this point Github pages adds a file named `CNAME` to the root of my remote repository. This file contains a single line which is the domain name `aregsar.com` that I added in the Github pages settings custom domain section.

As always I did a `git pull` to get the `CNAME` file that was added to the remote repository.

## Step 4 - Enable the Enforce HTTPS checkbox

Finally, In the Github pages section, I clicked the checkbox to enable https.

> Note: wait for DNS record propagation to take effect if github pages does not allow you to check the HTTPS checkbox and displays error messages complaining that it can't enable https. Any error should resolve automatically in a few minutes.

After checking the Https checkbox the published URL endpoint for the blog will change as indicated by the message at the top of the github pages section, which for me is:

`Your site is ready to be published at https://aregsar.com/`

> Note: if you see any github pages error messages, after checking the HTTPS box, wait for github pages to generate SSL certificates and publish at the https endpoint. Any error should resolve automatically in a few minutes.

## Conclusion

In this process I showed you the easy process of setting up a custom domain name for this blog.

Check out the next post in the series where I show you how I set up the file structure of this blog that determined the URL structure of the blog.

Thanks for reading.

[Top](#how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)
