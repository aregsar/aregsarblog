# how-to-setup-services-for-your-github-pages-blog

Separate this post into 3 posts

one for adding disqus commnets
one for adding mailchimp newsletter email
one for adding gumroad digital downloads





## Github pages technology

Before we begin I want to mention the components that Github pages uses so that you can have a better understanding of the content of this post.

Github pages uses standard Github repositories to host Github pages content. The URL structure of the content follows the repository structure. The repository content is written in markdown and converted to Html markup using Jekyll.

Jekyll is the name of the engine that Github pages uses to process the markdown pages in the repository to convert them to standard Html pages while applying CSS theme styling to the Html content.

Every time we push a change to our Github pages repository, Jekyll reprocesses and republishes the new markup after a brief delay.

Jekyll also uses a templating system called Liquid that contains Liquid tags to plug in html content from various sources. Jekyll uses Liquid tags  to plug in individual page content into layout templates and also to substitute text and metadata defined in settings files and other sources into the layout and pages.

Also GitHub Pages uses its own flavor of markdown called [Github Flavored Markdown](https://github.github.com/gfm/) or GFM. GFM is a flavor of Markdown that adds a few syntax extensions the allow for code blocks and syntax highlighting based on programming language.

You will not need to know GFM to understand this post.

You will need a Github account and a git client installed on your development machine.

## Setting up a custom domain for your blog

### Pointing a custom domain name to your Github pages site

To publish our pages at a custom domain name we need to add DNS records to point the custom domain to our Github pages server.

So once we register a custom domain at our favorite domain registrar, we need to go to the domain name administration panel of the registrar and create the neccessary records to point the custom domain to our GitHub pages site.

The process of pointing your custom domain to your Guthub pages repository URL is detailed in the following links

`https://help.github.com/articles/`

`using-a-custom-domain-with-github-pages/`

`https://help.github.com/articles/setting-up-an-apex-domain/`

`https://help.github.com/articles/setting-up-a-www-subdomain/`

This blog has the domain `https://aregsar.com` set as the Github Pages domain.

To setup a custom domain for my blog, I registered my custom domain name with my domain registrar dnsimple.com.

Then using the domain name administration panel of my registrar I added the neccessary records to point the custom domain to the location where Github hosts my GitHub pages site.

To set that up I added the following two records:

`ALIAS   aregsar.com -> aregsar.github.io`

`CNAME   www.aregsar.com -> aregsar.github.io`

As you can see I added an ALIAS record for root domain `argesar.com` mapped to `aregsar.github.io` which is where github pages hosts my GitHub pages site.

I also added a CNAME record for the subdomain `www.aregsar.com` also mapped to `aregsar.github.io`.

Github pages recomends adding a www subdomain and will redirect it to the root domain.

> Note: your registrar may not support ALIAS records. In that case follow the instructions at `https://help.github.com/articles/setting-up-an-apex-domain/` to add four `A` records instead that point to the IP address of the Github pages servers. The CNAME record should still map to `<your-github-username>.github.io` in this case.

### Connecting the custom domain to our Github pages repository

> Note that you have to register and point the domain, as described in the previous section, before being able to successfully complete the instuctions is this section.

We now need to go to the github pages section of the settings page for our repository and enter in the custom root domain name that we registered with our domain registrar.

For this site that would be the domain name aregsar.com.

So I entered the custom domain `aregsar.com` into the custom domain name text box in the github pages section and clicked save.

> Note: wait for DNS record propagation to take effect if you see any github pages error messages after saving the custom root domain name. Any error should resolve automatically in a few minutes.

At this point Github pages adds a file named `CNAME` to the root of your repository that contains a single line with the domain name we just added in the Github settings custom domain section.

For this blog it contains the line `aregsar.com`

### Enable the Enforce HTTPS checkbox

Finally, In the Github pages section, I clicked the checkbox to enable https

> Note: wait for DNS record propagation to take effect if github pages does not allow you to check the HTTPS checkbox and displays error messages complaining that it can't enable https. Any error should resolve automatically in a few minutes.

After checking the Https checkbox the published URL endpoint for the blog will change as indicated by the message at the top of the github pages section, which for me is:

`Your site is ready to be published at https://aregsar.com/`

> Note: if you see any github pages error messages, after checking the HTTPS box, wait for github pages to generate SSL certificates and publish at the https endpoint. Any error should resolve automatically in a few minutes.

### Overriding the default Jekyll layout

So after copying the default.html content,
I replaced the content in the `<body>` tags with:

```html
<header class="page-header"  
        role="banner">
      <h1 class="project-name">Areg Sarkissian</h1>
</header>
<main id="content" class="main-content" role="main">
      {{ "{{" }} content }}
      <footer class="site-footer">
        &copy; 2019 Areg Sarkissian
      </footer>
</main>
```

Note that there exists a `{{ "{{" }} content }}` parameter in the html. This is a Liquid templating parameter that Jekyll uses by including the html content of a processed markdown page into the layout html file. In fact anywhere Jekyll sees Liquid template parameters, even in markdown page content or html code snippets in a markdown page, it will try to substitute content of another page inside.

So the way I was able to display the `{{ "{{" }} content }}` parameter right here in the markdown text of this blog post, was to escape it. Otherwise Jekyll will substitute another page content instead of just allowing the `{{ "{{" }} content }}` text to be displayed. So be aware of that if you want to print Liquid template parameters into your own markdown posts.

Refer to this StackOverflow post to see how to escape Liquid template parameters in your Github pages:

[How to escape liquid template tags](https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags)

The layout change removed all the {{ site.<REPO_SETTINGS> }} tags that github pages uses to add the buttons and content to our layout.

The REPO_SETTINGS parameters are specified by the default repository settings such as repo name. However next, we will specify the settings in the _config.yml file that will override the default settings.




## Adding third party comments for your posts

Although I have not added a commenting service for my blog posts, I still wanted to mention that you can embed third party commenting service to your markdown pages.

Here is a Stack Overflow link that describes how to add Disqus comments to your markdown pages:

[How do I use disqus comments in github pages blog markdown](https://stackoverflow.com/questions/21446165/how-do-i-use-disqus-comments-in-github-pages-blog-markdown)

## Conslusion

[Top](#how-to-setup-services-for-your-github-pages-blog)