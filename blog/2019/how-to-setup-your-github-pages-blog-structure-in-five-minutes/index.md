# How to setup your github pages blog structure in five minutes

Mar 19, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

In this post I will show you how I set up the directory structure of my Github pages blog to give an example of how the pages and posts can be organized and how they can link to each other.

The directory structure of a Github pages repository determines the URL structure of the site. If a directory contains an `index.md` markdown file then that file becomes the default published page which can be accessed by a URL that ends with the directory name. A directory can also have other markdown files. In that scenario the URL needs to specify the file name without the `.md` extension as the trailing segment of the URL.

So for exampe the root directory of this site has an index.md file so the URLs `aregsar.com`, `aregsar.com/`, `aregsar.com/index`.

Also note that if you type`aregsar.com/index.md` you can see the actual markdown content.

Also I have an `about.md` page at the root of this repository. So in this case I can navigate to the published page with the URL `aregsar.com/about`. But if I add a trailing forward slash `aregsar.com/about/` I will get a 404 file not found becaus that URL is looking for an `index.md` file inside the `about` directory and I don't have an about directory.

Once again if we navigate to `aregsar.com/about.md` we can see the markdown content of the page.

Below I show the steps I took to create the structure of the blog.

## Step 1 - Created the directory structure and files

To start this blog I first created an about page by adding the following:

Added an about page to the site root

```bash
touch about.md
```

## Step 2 - Added the directory structur for blog posts

To add a directory structure that segments my posts into yearly posts I typed:

```bash
mkdir blog && cd blog
mkdir 2019 && cd 2019
```

This way I have a directory named by this year that will contain all the posts for this year.

## Step 3 - Added my first blog post

To add the file for the first blog post of the site, I chose to use the directory name as the name of the blog post and added an index.md file inside the directory to hold the content of the post.

Inside the `\blogs\2019\` directory I typed:

```bash
mkdir how-to-setup-a-github-pages-blog-in-five-minutes
cd how-to-setup-a-github-pages-blog-in-five-minutes
touch index.md
```

Alternatively, I could have chosen not to create a directory for the post and instead
to simply create a markdown file `how-to-setup-a-github-pages-blog-in-five-minutes.md`
But, this way I could add any images used by the post in the same directory.

Another advantage of using a directory with an index.md file for the blog post is that a trailing slash will not cause a 404 and simply redirect to the non trailing slash version of the URL.

I repeated this same process for adding blog posts for this year.

## Navigation links

There of four types of links that I have in my Github pages blog.

+ Global Html links that I added to the local html layout file for all my pages.
+ Markdown links in the markdown pages of the site for linking between pages and to external web sites.
+ Markdown image links in the markdown pages of the site for linking to image content inside the blog post directory.
+ Relative markdown links in the markdown pages for linking to the headings within the page from any location in the page.

In the four following sections, I will show you how I added each of these four types of links.

## Added global navigation links in the layout file

Next, in the `_layouts/default.html` file, I added anchor tags just after the opening `<main>` tag as shown below:

```html
 <main id="content" class="main-content" role="main">
      <a href="https://aregsar.com">Posts</a> | <a href="https://aregsar.com/about">About</a> | <a href="https://alwaysdeployed.com/tools">Tools</a>
      {{ "{{" }} content }}
```

## Add navigation links and content to index.md

I opened `index.md` file and replaced the content with:

```markdown
# Posts

Jan 30, 2019 by [Areg Sarkissian](https://aregsar.com/about)

[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)
```

## Adding image tags to our posts

I added the image file `github-pages-repo-settings-area.jpg` to the `blog/images/how-to-setup-a-github-pages-blog-with-markdown/` directory

Then I added the following markdown to a page in this blog to display the image:

```markdown
![Image](https://aregsar.com/blog/images/how-to-setup-a-github-pages-blog-with-markdown/github-pages-repo-settings-area.jpg)
```

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

Thanks for reading.

[Top](#how-to-setup-your-github-pages-blog-structure-in-five-minutes)