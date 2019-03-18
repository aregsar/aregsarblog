# how-to-setup-services-for-your-github-pages-blog

Separate this post into 3 posts

one for adding disqus commnets
one for adding mailchimp newsletter email
one for adding gumroad digital downloads

# Outline of this blog post

Below are jump links to each section of this blog:

[Abbreviated Companion Blog Post](#abbreviated-companion-blog-post)

[Github pages technology](#github-pages-technology)

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

### Adding a heading to our index.md page

The Jekyll engine substitutes the content of the index.md file for the `{{ "{{" }} content }}` parameter in the _layouts/default.html page

All subsequent markddown pages you add to the repo will similarly be substituted for the `{{ "{{" }} content }}` parameter when published

Now in the index.md file I added the h1 tag markdown line:

`# hello world`

using the bash script:

```bash
echo '# hello world' >> index.md
```

I saved and pushed the changes.

After some delay you can refresh the page and do a view source in the browser to see that jekyll converted the markdown to the following html:

`<h1 id="hello-world">hello world</h1>`

note that the text of the h1 tag markdown is used a the id of the h1 html tag with a dash used for the space character

### The URL structure of our blog

The url for the index.md file. Note we dont need to specify the file name since Github pages uses index.md as the default website root file.

`[Posts](https://aregsar.com)`

which is the same as

`[Posts](https://aregsar.com/index)`

The url for the about.md file

`[About](https://aregsar.com/about)`

The url for the blog post

`[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)`

Note you can also reference file with an html extension

`[About](https://aregsar.com/about.html)`

`[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown.html)`

or for the index

`[Posts](https://aregsar.com/index.html)`

To referenence the source markdown files that are converted to html use:

`[About](https://aregsar.com/about.md)`

`[Posts](https://aregsar.com/index.md)`

`[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown.md)`

### Adding links to the site layout

I added the html anchor tags for the index and about pages right above the `{{ "{{" }} content }}` parameter in the _layouts/default.html file.

Markdown can not be used in the layout.html file since the layout is a html file and will not be converted by jekyll.

Here is a sippet from default.html that shows the anchor tags pasted right above the `{{ "{{" }} content }}` parameter

```html
 <main id="content" class="main-content" role="main">
      <a href="https://aregsar.com">Posts</a> | <a href="https://aregsar.com/about">About</a>
      {{ "{{" }} content }}
```

### Linking to the blog post from index.md file

I opened index.md file and replaced the content with:

```markdown
# Posts

Jan 30, 2019 by [Areg Sarkissian](https://aregsar.com/about)

[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)
```

And thats all there is to it!

## Linking to headers within a page

The GFM syntax for linking to heading tags within a page uses the text of the heading.

Jekyll first converts the markdown heading text to all ower case characters. Also dashes replace any spaces in the text and any periods in the text are removed.

Then it sets the text to the id attribute of the header.

So to link to the heading we can use the same converted text placed inside the parenthesis portion of a markdown link tag and prepended with a # symbol.

So for example if have markdown heading `# MyPosts` then Jekyll will convert the markdown text to the following heading tag: `<h2 id='myposts'>MyPosts</h2>`

So the markdown link to the heading tag will use the converted name as shown:

```markdown
[My Postings](#myposts)
```

Here is another example where the heading text has spaces between words. In this case the markdown heading is `# My Posts` which is converted to `<h2 id='my-posts'>My Posts</h2>` then the markdown link will again use the converted text like so:

```markdown
[My Postings](#my-posts)
```

Note that the link text inside the brackets can be any text we wish just like a standard markdown anchor tag.

Here is an example that uses a period in the heading text
where the text is `How to create an index.md page`
In this case  the link text would be:

```markdown
[Top](#how-to-create-an-indexmd-page)
```

Note that the period in `index.md` part was removed.

In the bottom of this blog post I added the following markdown anchor tag to link to the title h1 heading of this post with that has heading text `How to setup a github pages blog with markdown`

```markdown
[Top](#how-to-setup-a-github-pages-blog-with-markdown)
```

## Adding image tags to our posts

To have a directory for images used in my blog posts, I added a directory named "images" inside the blog directory.

using the bash script:

```bash
cd blog
mkdir images
```

Then for each blog post I create a directory in the images directory using the blog post title as the directory name. So for images for this post I created the subdirectory `images/how-to-setup-a-github-pages-blog-with-markdown/` in the root.

This is where any images for this blog post are added.

In order to show an image of the Github pages settings area on the Github pages repository settings page, I added the image file github-pages-repo-settings-area.jpg to the directory.

Then I added the following markdown image tag in this blog post to reference the image.

```markdown
![Image](https://aregsar.com/blog/images/how-to-setup-a-github-pages-blog-with-markdown/github-pages-repo-settings-area.jpg)
```

## Adding third party comments for your posts

Although I have not added a commenting service for my blog posts, I still wanted to mention that you can embed third party commenting service to your markdown pages.

Here is a Stack Overflow link that describes how to add Disqus comments to your markdown pages:

[How do I use disqus comments in github pages blog markdown](https://stackoverflow.com/questions/21446165/how-do-i-use-disqus-comments-in-github-pages-blog-markdown)

## Notes on using relative URLs

GFM has a Markdown extension that allows us to use relative page references instead of absolute URLs in our Markdown links.

Using relative page references, the link to the root index.md file from the about.md page would be

`[Posts](index.md)`

the link to about.md page from the index.md page would be

`[About](about.md)`

and the link from the index.md page to this blog post page setting-up-a-blog-with-github-pages-using-markdown.md would be

`[Setup a Github pages blog using markdown and Jekyll themes](blog/setting-up-a-blog-with-github-pages-using-markdown.md)`

Also the links from this page to the index and about pages at the root would be

`[Posts](../index.md)` and `[About](../about.md)`

where ../ parent directory traversal is used to navigate to the root from the blog subdiectory.

This style of linking has issues that I wont go into detail of, but the primary one being that the generated URLs for the anchor tags will contain the .html extension and I wanted my links to use extensionless URLS.

That is why I chose using absolute URL linking for this site