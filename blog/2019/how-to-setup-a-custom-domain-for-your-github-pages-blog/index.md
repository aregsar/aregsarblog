# how-to-setup-a-custom-domain-for-your-github-pages-blog

## Pointing a custom domain name to your github pages site

To setup a custom domain for my blog, I registered my custom domain name with my domain registrar dnsimple.com.

Then using the domain name administration panel of my registrar I added the neccessary records to point the custom domain to the location where Github hosts my GitHub pages site.

To set that up I added the following two records:

`ALIAS   aregsar.com -> aregsar.github.io`

`CNAME   www.aregsar.com -> aregsar.github.io`

As you can see I added an ALIAS record for root domain `argesar.com` mapped to `aregsar.github.io` which is where github pages hosts my GitHub pages site.

I also added a CNAME record for the subdomain `www.aregsar.com` also mapped to `aregsar.github.io`.

Github pages recomends adding a www subdomain and will redirect it to the root domain.

See the following link for more info:

https://help.github.com/articles/using-a-custom-domain-with-github-pages/

> Note: your registrar may not support ALIAS records. In that case follow the instructions at `https://help.github.com/articles/setting-up-an-apex-domain/` to add four `A` records instead that point to the IP address of the Github pages servers. The CNAME record should still map to `<your-github-username>.github.io` in this case.

## Connecting the custom domain to our Github pages repository

> Note that you have to register and point the domain, as described in the previous section, before being able to successfully complete the instuctions is this section.

Next I entered the custom domain `aregsar.com` into the custom domain name text box in the github pages section and clicked save.

> Note: wait for DNS record propagation to take effect if you see any github pages error messages after saving the custom root domain name. Any error should resolve automatically in a few minutes.

At this point Github pages adds a file named `CNAME` to the root of your repository that contains a single line with the domain name we just added in the Github settings custom domain section.

For this blog it contains the line `aregsar.com`

## Enable the Enforce HTTPS checkbox

Finally, In the Github pages section, I clicked the checkbox to enable https

> Note: wait for DNS record propagation to take effect if github pages does not allow you to check the HTTPS checkbox and displays error messages complaining that it can't enable https. Any error should resolve automatically in a few minutes.

After checking the Https checkbox the published URL endpoint for the blog will change as indicated by the message at the top of the github pages section, which for me is:

`Your site is ready to be published at https://aregsar.com/`

> Note: if you see any github pages error messages, after checking the HTTPS box, wait for github pages to generate SSL certificates and publish at the https endpoint. Any error should resolve automatically in a few minutes.