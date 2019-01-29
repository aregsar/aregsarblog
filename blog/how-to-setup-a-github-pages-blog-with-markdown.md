# How to setup a github pages blog with markdown

Jan 29, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Welcome to the first post on my new blog.

In this post I will show you how I set up this blog with [GitHub Pages](https://pages.github.com/) and [Markdown](https://commonmark.org/help/) and how you can easily do the same.

GitHub Pages uses [Github Flavored Markdown](https://github.github.com/gfm/) or GFM. GFM is a flavor of Markdown that adds a few syntax extensions for code blocks and syntax highlighting.

You will not need to know GFM to understand this post.

You will need a Github account and a git client installed on your development machine.

## Creating a repository to host your static Markdown pages on Github

The first step to setup a GitHub Pages blog is to create an empty GitHub repository to host your blog.

So for this blog I created an empty repo named aregsarblog in my Github account.

## Adding the main page for your blog

Once the empty repository is created, we will add an index.md Markdown file at the root of the repository.

This file will be converted to html by Jekyll, the Markdown transformation engine run by Github pages.

The processed html file will then be served when you navigate to the root URL of your site.

In order to setup the main page for this blog I used bash commands to create a new local repository named aregsarblog that contains an `index.md` file and pushed it up to the remote Github repo that I created in the previous step.

```bash
mkdir aregsarblog && cd aregsarblog
touch index.md
git init
git add index.md
git commit -m "first commit"
git remote add origin git@github.com:aregsar/aregsarblog.git
git push -u origin master
```

At this point the index.md file should apear in the master branch.

## Turning the repository into a GitHub Pages site

To turn your newly created repository into a Github pages site, Go to the settings page for the repository by clicking on the settings button at the top of your repository page.

On the settings page, scroll down to the Github pages section and activate Github pages by selecting the master branch using the select source dropdown and then click save.

At this point for me I see the message below at the top of the
github pages section

your site is published `https://aregsar.github.io/aregsarblog/`

So that's it. Congratulations, you just created your first GitHub Pages site.

## Setting up the default theme for your site

In the Github pages section, click on the change theme button and select a default theme for your site.

I chose the Caymen theme for this site.

To select the theme click on the choose a theme buttom then select your theme and click the select theme buttom.

Clicking the select theme buttom will return you to the settings page and will add a _config.yml file to the root of your repo

This file will contain a single line that shows the selected theme.

For my blog it contains:

`theme: jekyll-theme-cayman`

The actual theme name is the last segment which in our case is cayman. This is the name we will use to locate the theme repository for the cayman theme in order to customize our default theme.

If you navigate to where the site is published, which in my case is `https://aregsar.github.io/aregsarblog`, you will see that the default theme has added repository related elements such as the `view on Github` button to the page.

In the following sections I will show you how to customize the default theme to remove the standard repository elements in the default theme and make the site look like this blog.

But before we do that lets associate a custom domain name to our blog.

## Setting up a custom domain name for our site

To setup a custom domain for your page at anytime, go to the github pages section of the settings page for your repository and enter in the custom domain name that you registered with your domain registrar.

For this site I regitsted the domain name aregsar.com

then I entered the name in the custom domain name text box in the github pages section and  and clicked save.

After clicking save check the Enforce HTTPS checkbox if it not already checked to enable HTTPS access.

At this point a file named `CNAME` was added to the root of your repository that contains a single line with the domain name we just added in the Github settings custom domain section.

For this blog it contains the line `aregsar.com`

Adding the custom domain name also changes the publish URL for the site as indicated by the message at the top of the github pages section:

Your site is ready to be published at `https://aregsar.com/`

After a small delay when the site is republished by Jekyll the message will change to:

Your site is published at `https://aregsar.com/`

## Pulling down changes made to our remote repo by github pages

Since Github pages settings changes added the _config.yml and CNAME files to our remote repo we need to pull it down to our local repo so we will be in sync.

So I did a git pull from my local repo directory to pull down the two added files.
Your local repo directory should now contain the following three files:

index.md

_config.yml

CNAME

## Pointing a custom domain name to your site

In order for the site to actually be published at the custom domain we specified, we need to add DNS records to point to the Github pages server.

So go to the domain name administration panel of your domain registrar and create the neccessary records to point your domain to your GitHub pages site.

The process of pointing your custom domain to your Guthub pages repository URL is detailed in the following links 

https://help.github.com/articles/using-a-custom-domain-with-github-pages/

https://help.github.com/articles/setting-up-an-apex-domain/

https://help.github.com/articles/setting-up-a-www-subdomain/

This blog has the domain https://aregsar.com set as the Github Pages domain. 
So after logging into my domain registrar I added the following two records

ALIAS   aregsar.com -> aregsar.github.io

CNAME   www.aregsar.com -> aregsar.github.io

As you can see I added a ALIAS record for root domain argesar.com mapped to the github pages publishing domain aregsar.github.io

I also added a CNAME record for the www subdomain www.aregsar.com also mapped to aregsar.github.io.

Github pages recomends adding a www subdomain and will redirect it to the root domain.

Note if your registrar may not support ALIAS records and you are forced to create A records instead. In that case follow the instructions at https://help.github.com/articles/setting-up-an-apex-domain/ to add four A records instead that point to the IP address of the Github pages servers. The CNAME record should still map to aregsar.github.io in this case.

## The Github pages default theme repo

All the Github pages default themes files that Jekyll uses to apply the themes during its markdown file transformation process are located in the jekyll themes repo located at https://github.com/pages-themes

Each individual theme is at located in a sub directory https://github.com/pages-themes/THEME_NAME

Since I chose the cayman theme, the theme repo that Jekyll uses by default for this site is at https://github.com/pages-themes/cayman

## Overriding the default theme for your site

### Overriding the default Jekyll layout to get rid of the github repo default layout boilerplate

the default layout for our cayman theme is at:
https://github.com/pages-themes/cayman/blob/master/_layouts/default.html

to override the layout I created a file _layouts/default.html at the root of the repo

```bash
mkdir _layouts && cd _layouts
touch default.html
```

next I navigated to the default layout repo page  
https://github.com/pages-themes/cayman/blob/master/_layouts/default.html

clicked on the raw button and copied the raw text into your the `_layouts/default.html` file

Now we can make changes to the our local default.html file to remove the boilerplate that github pages adds for standard repository pages.

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

OK so finally I pushed the changes to the repo to create the stripped down layout of this site.

By the way, there will be a slight delay, usually under a minute, every time you push a change for the changes to display on the site so be patient.
At this point all the default content and buttons that github added to layout content is removed

The layout change removed all the {{ site.<REPO_SETTINGS> }} tags that github pages uses to add the buttons and content to our layout.

The REPO_SETTINGS parameters are specified by the default repository settings such as repo name. However next, we will specify the settings in the _config.yml file that will override the default settings.

### The SEO markup tag and changing your site title

The <head> tag of default.html file contains a {% seo %} parameter that when processed by Jekyll will generate title and description meta tags used for SEO.

The genrated title meta tag uses the repo name for title value that displays the site title in the web browser bar. Similarly the genrated description meta tag uses the Github repo description you specified when you created the repo.

To override that default title and description meta tag values I added the two lines below to the _config.yml at the root of the repo.

title: aregsar

description: Areg Sarkissians Blog

Save and push to see the title text change.

As always give Github pages a little time to process the change you just pushed to the repo.

Initially for this blog the title in the browser tab was the name of the repo aregsarblog. But after I added the title to _config.yml the title changed to the current title of aregsar.

### Adding a heading to our index.md using markdown

The Jekyll engine substitutes the content of the index.md file for the `{{ "{{" }} content }}` parameter in the _layouts/default.html page

All subsequent markddown pages you add to the repo will similarly be substituted for the `{{ "{{" }} content }}` parameter when published

Now I opened  index.md and added the h1 tag markdown line below

`# hello world`

I saved and pushed the changes.

After some delay you can refresh the page and do a view source in the browser to see that jekyll converted the markdown to the following html:

`<h1 id="hello-world">hello world</h1>`

note that the text of the h1 tag markdown is used a the id of the h1 html tag with a dash used for the space character

### The default theme styles

The default style that Github pages applies for the cayman theme is located at 
https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss

inside this file there is import statement
@import 'jekyll-theme-cayman';

that imports the styles located at

https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss

By default Jekyll compiles this style.scss file to style.css that used to style the page. The compiled style.css is referenced in the `<head>` tag of _layouts/default.html file using the `<link>` element shown below:

`<link rel="stylesheet" href="{{ '/assets/css/style.css?v=' | append: site.github.build_revision | relative_url }}">`

### overriding default theme styles

In order to override the default style.scss we need to first create a file assets/css/style.scss at the root of your repo
and add the following four lines at the top the file

```scss
---
---

@import "{{ site.theme }}";
```

the `@import "{{ site.theme }}";` line is transformed to
`@import "jekyll-theme-cayman";` by Jekyll because of the line `theme: jekyll-theme-cayman` specified in our _config.yml file.

This is the same import statement used in the default https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss file.

I added the assets/css/style.scss with the bash statements below at the root of this repo.

```bash
mkdir assets && cd assets
mkdir css && cd css
echo '---' >> style.scss
echo '---' >> style.scss
echo >> style.scss
echo '@import "{{ site.theme }}";' >> style.scss
```

I pushed the changes for the site to use the local style.scss to override the default theme style.scss

The local style.scss will be used to generate the style.css that is  applied to the page.

You can verify this after a brief delay by doing a view source on the refreshed page to see the published style.css link

`<link rel="stylesheet" href="/assets/css/style.css?v=c9149429e7d8df2dde257387400cef49363fb589">`

### customizing the theme styles

Now that we have overriden the default styles.scss file with our local styles.scss file, we can customize our local styles.scss file to customize the styles for our pages.

To customize we add any custom imports or css or scss styles immediately after the `'@import "{{ site.theme }}";'` line in our local styles.scss file.

As an example if we open the imported default cayman theme scss file located at
https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss

we will see the following style snipets inside:

```scss
@import "variables";
body {
  color: $body-text-color;
}

a {
  color: $body-link-color;
}

.page-header {
  color: $header-heading-color;
  text-align: center;
  background-color: $header-bg-color;

.main-content {
  h1,
  h2,
  h3,
  h4,
  h5,
  h6 {
    margin-top: 2rem;
    margin-bottom: 1rem;
    font-weight: normal;
    color: $section-headings-color;
  }
```

So we can as an example override the color of the .main-content h1 tag setting it to red by placing the following style snippet right after the import statement in our local style.scss file

```scss
.main-content {
  h1 {
   color: #ff0000;
 }
}
```

so the final file content would be

```scss
---
---

@import "{{ site.theme }}";
.main-content {
  h1 {
   color: #ff0000;
 }
}
```

Push the changes and see that the color of our h1 heading we added to index.md changes to red.

The generated .main-content h1 style is

```css
.main-content h1{color:#ff0000}
```

for the h1 element in main element

```html
<main id="content" class="main-content" role="main">
      <h1 id="hello-world">hello world</h1>
```

which can be seen by doing view source and navigating to the generated css file referenced in the header

```html
<link rel="stylesheet" href="/assets/css/style.css?v=f97443281054e55039f2bad9d2237e5486d287c7">
```

## Setting up links between markdown pages in your repo using absolute URL markdown

To explain how to setup navigation links between your pages, I will use the structure of my blog as an example.

For this blog I have an index.md page and an about.md page in the root of my repository.

I also have a subdirectory named blog in the root of the repository that contains all the posts on this blog.

Each post is a markdown file within the blog directory.

To mimick this structure add a about.md page and a blog subdirectory to the root of the directory

Then go to the blog subdirectory and add a markdown file for your first blog post.

I use the title of this post as the file name.

I used the bash script below at the root of the repo to add the files

```bash
echo "# About" >> about.md
mkdir blog && cd blog
echo "# How to setup a github pages blog with markdown" >> how-to-setup-a-github-pages-blog-with-markdown.md
```

### linking to markdown files from other markdown files using absolute URLs

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

### creating a link for the index.md and about.md files in the site layout html file

I added the html anchor tags for the index and about pages right above the `{{ "{{" }} content }}` parameter in the _layouts/default.html file.

Markdown can not be used in the layout.html file since the layout is a html file and will not be converted by jekyll.

Here is a sippet from default.html that shows the anchor tags pasted right above the `{{ "{{" }} content }}` parameter

```html
 <main id="content" class="main-content" role="main">
      <a href="https://aregsar.com">Posts</a> | <a href="https://aregsar.com/about">About</a>
      {{ "{{" }} content }}
```

### Adding a link to the blog post in the index.md page

I opened index.md file and replaced the content with:

```markdown
# Posts

Jan 30, 2019 by [Areg Sarkissian](https://aregsar.com/about)

[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)
```

And thats all there is to it!

## linking to headers within a page

The GFM syntax for linking to header sections within a page is to use the text of the heading inside the parenthesis portion of a markdown link tag and prepended with a # symbol. Note that all characters will be lower cased and dashes replace any spaces in the header text.

So for example if have a heading text, `<h2>MyPosts</h2>`, then the link portion of markup is:

[My Postings](#myposts)

For heading with spaces in the heading text, we need to use a dash in the link portion of the tag.

So if the heading is `<h2>My Posts</h2>` then the link portion should have dashes instead of spaces like so:

[My Postings](#my-posts)

Note that the link text inside the brackets can be any text we wish just like a standard markdown anchor tag.

So in the bottom of this post I added the following markdown to link to the title h1 header of this post.

Since the header text is `How to setup a github pages blog with markdown`

The relative link is specified as:

```markdown
[Top](#how-to-setup-a-github-pages-blog-with-markdown)
```

## Notes on using GFM relative URLs to link between pages

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

[Top](#how-to-setup-a-github-pages-blog-with-markdown)