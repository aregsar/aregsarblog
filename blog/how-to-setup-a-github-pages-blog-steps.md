# How to setup a github pages blog steps

Jan 29, 2019 by [Areg Sarkissian](https://aregsar.com/about)

## Full version of this Blog Post

If you want to see the detailed explanations for the steps in this post, you can take look at the full version of this post here:

[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)

## Create a repository

Created an empty repo named aregsarblog in my Github account

## Add the main page for your blog

Added index.md main page of this blog

```bash
mkdir aregsarblog && cd aregsarblog
touch index.md
git init
git add index.md
git commit -m "first commit"
git remote add origin git@github.com:aregsar/aregsarblog.git
git push -u origin master
```

## Turning the repository into a GitHub Pages site

On the settings page, scroll down to the Github pages section and activate Github pages by selecting the master branch using the select source dropdown and then click save.

## Setting up the default theme for your site

To select the theme:

On the settings page, scroll down to the Github pages section click on the Change theme buttom then select your theme and click the select theme buttom.

Clicking the select theme buttom will return you to the settings page and will add a `_config.yml` file to the root of your repo.

## Pointing a custom domain name to your github pages site

register a custom domain at our favorite domain registrar, by going to the domain name administration panel of the registrar and create the neccessary records to point the custom domain to our GitHub pages site.

To set that up I logging into my domain registrar and added the following two records:

ALIAS   aregsar.com -> aregsar.github.io

CNAME   www.aregsar.com -> aregsar.github.io

see the following link for more info:

https://help.github.com/articles/using-a-custom-domain-with-github-pages/

## Connecting the custom domain to our Github pages repository

I entered the custom domain `aregsar.com` that I pointed to github pages in previous step into the custom domain name text box in the github pages section and clicked save.

> Wait for DNS record propagation to take effect if you see any github pages error messages. Any error should resolve automatically in a few minutes.

At this point Github pages adds a file named `CNAME` to the root of your repository that contains a single line with the domain name we just added in the Github settings custom domain section.

## enable the Enforce HTTPS checkbox

click the checkbox to enable https

> wait for github pages to generate tls certificates if you see any github pages error messages. Any error should resolve automatically in a few minutes.

## Pulling down changes made to our remote repo by github pages

I did a `git pull` to pull down the _config.yml and CNAME files added to the repo by Github pages.

## Overriding the default theme for your site

created a file _layouts/default.html at the root of the repo

```bash
mkdir _layouts && cd _layouts
touch default.html
```

next I navigated to the default layout repo page  
`https://github.com/pages-themes/cayman/blob/master/_layouts/default.html`

clicked on the raw button and copied the raw text into the `_layouts/default.html` file

> If you chose a different theme then replace `cayman` in the above link with your theme name. All the themes for GitHub pages are located at `https://github.com/pages-themes`

Next, in the `_layouts/default.html` file, I replaced the content inside the `<body>` tags with:

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

## Changing your site title

To override that default title and description meta tag values I added the two lines below to the _config.yml at the root of the repo.

using the bash script:

```bash
echo 'title: aregsar' >> index.md
echo 'description: Areg Sarkissians Blog' >> index.md
```

## Adding a heading to our index.md using markdown

Added h1 markdown tag to index.md in the root of the repo, using the bash script:

```bash
echo '# hello world' >> index.md
```

## overriding default theme styles

I added the assets/css/style.scss with the bash statements below at the root of the repo.

```bash
mkdir assets && cd assets
mkdir css && cd css
echo '---' >> style.scss
echo '---' >> style.scss
echo >> style.scss
echo '@import "{{ site.theme }}";' >> style.scss
```

at this point the style.scss will contain:

```scss
---
---

@import "{{ site.theme }}";
```

## customizing the theme styles

add the .main-content h1 style to the assets/css/style.scss file.

After adding the content of style.scss will be:

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

## Commit changes

I commited changes and pushed to the repo to see the layout, title and style changes.

## create the directory structure and files

I used the bash script below at the root of the repo to add the about.md file
and the blog subdirectoy with the first blog post markdown file.

```bash
echo "# About" >> about.md
mkdir blog && cd blog
echo "# How to setup a github pages blog with markdown" >> how-to-setup-a-github-pages-blog-with-markdown.md
```

## Add global navigation links in the layout file

Next, in the `_layouts/default.html` file, I added anchor tags just after the opening `<main>` tag as shown below:

```html
 <main id="content" class="main-content" role="main">
      <a href="https://aregsar.com">Posts</a> | <a href="https://aregsar.com/about">About</a>
      {{ "{{" }} content }}
```

## Add navigation links and content to index.md

I opened index.md file and replaced the content with:

```markdown
# Posts

Jan 30, 2019 by [Areg Sarkissian](https://aregsar.com/about)

[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)
```

## Relative links to headers within a markdown page

The GFM syntax for linking to header sections within a page is to use the text of the heading inside the parenthesis portion of a markdown link tag and prepended with a # symbol. Note that all characters will be lower cased and dashes replace any spaces in the header text.

In the bottom of this post I added the following markdown to link to the title h1 header of this post.

Since the header text is `How to setup a github pages blog with markdown`

The relative link is specified as:

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