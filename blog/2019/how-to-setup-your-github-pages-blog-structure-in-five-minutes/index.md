# How to setup your github pages blog structure in five minutes

Mar 18, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

[Part 6 - Setup third party services for your github pages blog](https://aregsar.com/blog/2019/how-to-setup-third-party-services-for-your-github-pages-blog)

[Part 7 - Github pages tech stack: Jekyll, Liquid, Markdown](https://aregsar.com/blog/2019/github-pages-tech-stack-jekyll-markdown-liquid)

Welcome to part 5 of this series on setting up a blog with Github pages.

In this post I will show you how I set up the directory structure of my Github pages blog to give an example of how the pages and posts can be organized and how they can link to each other.

## The directory structure and URL paths

The directory structure of a Github pages repository determines the URL structure of the site. If a directory contains an `index.md` markdown file then that file becomes the default published page which can be accessed by a URL that ends with the directory name.

A directory can also have other markdown files. In this case, the page URL needs to specify the file name without the `.md` extension as the trailing segment of the URL.

So for example the root directory of this site has an `index.md` file so it will be displayed at the URL `aregsar.com` since it is the default displayed file, but also at these URLs `aregsar.com/`, `aregsar.com/index` and `aregsar.com/index.html`.

> Note that if you type`aregsar.com/index.md` you can see the actual markdown content.

I also have an `about.md` page at the root of this repository. So in this case I can navigate to the published `about` page with the URLs `aregsar.com/about` and `aregsar.com/about.html` but not `aregsar.com/about/` because the later assumes that we are navigating to an `index.md` page in an `aregsar.com/about/` directory which does not exist.

Once again if we navigate to `aregsar.com/about.md` we can see the markdown content of the page.

> Aside: We can specify the link to the about page using markdown inside a markdown file as `[About](https://aregsar.com/about)` where the text of the link is in the square brackets and the URL is inside the parenthesis.

Below I detail the steps I took to create the structure of this blog.

## Step 1 - Created the directory structure and files

I have already shown in this series that I created an `index.md` file in the root of the repository for the home page of this blog.

In addition I created an about page at the root of the repository as well by using the following bash command:

```bash
touch about.md
```

## Step 2 - Added the directory structure for blog posts

To add a directory structure that segments my posts into yearly posts
using a URL path format `argsar.com/blog/<year>/<blog-post-title>/` I created the `/blog/2019/` child directories in the repository root:

```bash
mkdir blog && cd blog
mkdir 2019 && cd 2019
```

This way I have a directory `/blogs/2019/` that can contain all the posts for the year 2019.

## Step 3 - Added my first blog post

To add the file for the first blog post of the site, I chose to use the directory name as the name of the blog post and added an `index.md` file inside the directory to hold the content of the post, as the default displayed page that I described at the begining of this post.

Inside the `/blogs/2019/` directory I typed:

```bash
mkdir how-to-setup-a-github-pages-blog-in-five-minutes
cd how-to-setup-a-github-pages-blog-in-five-minutes
touch index.md
```

Alternatively, I could have chosen not to create a directory for the post and instead
to simply create a markdown file `how-to-setup-a-github-pages-blog-in-five-minutes.md`
But, this way I could add any images used by the post in the same directory as the blog post page.

Another advantage of using a directory with an `index.md` file as the blog post file is that a trailing slash will not cause a 404 file not found and simply redirect to the non trailing slash version of the URL.

I repeated this same process for adding other blog posts for this year.

## Navigation links

There are four types of links that I have in my Github pages blog.

+ Global Html links that I added to the local html layout file for all my pages.
+ Markdown links in the markdown pages of the site for linking between pages and to external web sites.
+ Markdown image links in the markdown pages of the site for linking to image content inside the blog post directory.
+ Relative markdown links in the markdown pages for linking to the headings within the page from any location in the page.

In the four following sections, I will show you how I added each of these four types of links.

> Note: Github pages uses __Github Flavored Markdown__ which is a flavor of markdown with additional syntax for programming language code blocks.

## Added global navigation links in the layout file

In the `_layouts/default.html` file, I added anchor tags just after the opening `<main>` tag as shown below:

```html
 <main id="content" class="main-content" role="main">
      <a href="https://aregsar.com">Posts</a> | <a href="https://aregsar.com/about">About</a> | <a href="https://alwaysdeployed.com/tools">Tools</a>
      {{ "{{" }} content }}
```

Markdown can not be used in the layout.html file since the layout is a html file and will not be transformed by jekyll.

The snippet above shows the anchor tags pasted right above the `{{ "{{" }} content }}` parameter.

## Added markdown navigation links and content to home page

I opened `index.md` file at the root of the repository and replaced the content with:

```markdown
# All Posts

Mar 18, 2019

[Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

Mar 12, 2019

[Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

Mar 11, 2019

[Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

Mar 7, 2019

[Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)
```

You can see this content by navigating to the markdown file at [https://aregsar.com/index.md](https://aregsar.com/index.md)

> Note: content will probably not be exactly the same as I have updated the page since.

## Added image tags to my blog post markdown pages

I added the image file `github-pages-repo-settings-area.jpg` to the `blog/2019/how-to-setup-a-github-pages-blog-with-markdown/` directory.

Then I added the following markdown to a page in this blog to display the image:

```markdown
![Image](https://aregsar.com/blog/2019/ages/how-to-setup-a-github-pages-blog-with-markdown/github-pages-repo-settings-area.jpg)
```

## Added relative links to headers within a markdown page

In the bottom of this blog post I added the following markdown anchor tag to add a relative link to the title h1 heading of this post which has heading text `How to setup your github pages blog structure in five minutes`:

```markdown
[Top](#how-to-setup-your-github-pages-blog-structure-in-five-minutes)
```

Below I'll describe in detail the syntax of relative markdown links and how Jekyll processes them:

The markdown syntax for linking to heading tags within a page uses the text of the heading.

Jekyll first converts the markdown heading text to all lower case characters and uses dashes to replace any spaces in the text. It also removes any periods in the text.

Then Jekyll sets the converted heading text to the id attribute of the header.

So to link to the heading we need to use the converted heading text, place it inside the parenthesis portion of a regular markdown link tag, then prepended the text with a `#` symbol to be used as an CSS id attribute selector.

So in the example above the title heading markdown text of this page is `How to setup your github pages blog structure in five minutes` as shown below:

```markdown
# How to setup your github pages blog structure in five minutes
```

So Jekyll converts that text to `how-to-setup-your-github-pages-blog-structure-in-five-minutes` and sets it to the id attribute of the converted markdown h1 heading tag as shown below:

```html
<h1 id="how-to-setup-your-github-pages-blog-structure-in-five-minutes">How to setup your github pages blog structure in five minutes</h1>
```

And finally to relatively reference this heading html we prepend the hash `#` symbol to the text and set it to the URL portion of the markdown link tag to reference the id attribute of the h1 tag as shown below:

```markdown
[Top](#how-to-setup-your-github-pages-blog-structure-in-five-minutes)
```

More Generic examples follow for most variations of heading content and how they get converted by Jekyll:

In this first example, say we have a markdown h1 heading `# MyPosts` then Jekyll will convert the markdown heading to the following heading tag: `<h1 id='myposts'>MyPosts</h2>`

And the markdown link to this heading tag will use the converted text as shown:

```markdown
[My Postings](#myposts)
```

Here is another example where the heading text has spaces between words. In this case the markdown heading is `# My Posts` which is converted to `<h1 id='my-posts'>My Posts</h2>` and the markdown link will again use the converted text like so:

```markdown
[My Postings](#my-posts)
```

> Note that the link text inside the brackets can be any text we wish just like a standard markdown anchor tag.

Here is an example that uses a period in the heading text where the markdown heading is `# How to create an index.md page` which is converted to `<h1 id='how-to-create-an-indexmd-page'>How to create an index.md page</h2>` and the markdown link will again use the converted text like so:

```markdown
[Top](#how-to-create-an-indexmd-page)
```

> Note that the period in `index.md` part was removed.

## Notes on using GFM style alternative relative URLs between pages

GFM has a Markdown extension that allows us to use relative page references, instead of absolute URLs in our Markdown links which I detailed in the sections above.

Using relative page references, the link to the root index.md file from the about.md page would be

`[Posts](index.md)`

the link to about.md page from the index.md page would be

`[About](about.md)`

Both the index.md and about.md are in the same directory.

The relative from the index.md page to this blog post page setting-up-a-blog-with-github-pages-using-markdown.md would be:

`[Setup a Github pages blog using markdown and Jekyll themes](blog/2019/setting-up-a-blog-with-github-pages-using-markdown/index.md)`

This is because the index.md file for this post is relative to the repository root index.md file

Also the links from this page to the index and about pages at the root would be

`[Posts](../../../index.md)` and `[About](../../../about.md)`

where the `<../ parent>` directory traversal is used to navigate to the root of the repository from the blog post subdirectory.

This style of linking has issues that I wont go into detail of, but you might choose to use this style of linking between pages.

## Conclusion

In this blog post I showed you how I setup the directory structure of this blog, how the structure determined the URL structure of the site.

I also showed you the many forms of navigation links that I added to this blog.

Finally I discussed other alternative structures that I chose not to use for this blog.

In the next post in the series I will list references to third party services that you can add to your Github pages blog.

Thanks for reading.

[Top](#how-to-setup-your-github-pages-blog-structure-in-five-minutes)