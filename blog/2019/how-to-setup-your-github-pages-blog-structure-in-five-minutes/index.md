# how-to-setup-your-github-pages-blog-structure-in-five-minutes


## create the directory structure and files

Your Github pages URL structure follows your repository directory structure

each sub directory can have an index.md file in which case if the directory name is the last segment of the URL the processed content of the index.md page will be displayed.

Markdown files by any other name will be accessed by using the filename without the extension as the last segment of the URL provided the URL path follows the directory structure.

So to start this blog I first created an about page by adding the following:

Added an about page to the site root
```bash
touch about.md
```


## Added global navigation links in the layout file

Next, in the `_layouts/default.html` file, I added anchor tags just after the opening `<main>` tag as shown below:

```html
 <main id="content" class="main-content" role="main">
      <a href="https://aregsar.com">Posts</a> | <a href="https://aregsar.com/about">About</a>
      {{ "{{" }} content }}
```


## Added the directory structur for blog posts





To add a directory structure that segments my posts into yearly posts I typed:

```bash
mkdir blog && cd blog
mkdir 2019 && cd 2019
```

## Added my first blog post

To add the file for the first blog post of the site, I chose to use the directory name as the name of the post and add an index.md file inside the directory to hold the content of the post. This way I could add any images used by the post in the same directory.


Inside the `\blogs\2019\` directory I typed:

```bash
mkdir how-to-setup-a-github-pages-blog-in-five-minutes
cd how-to-setup-a-github-pages-blog-in-five-minutes
touch index.md
```

I could have chosen not to create a directory for the post instead use the name I used for the directory to simply creat a file by that name and put the file where I created the directory.  


## Add navigation links and content to index.md

I opened index.md file and replaced the content with:

```markdown
# Posts

Jan 30, 2019 by [Areg Sarkissian](https://aregsar.com/about)

[How to setup a github pages blog with markdown](https://aregsar.com/blog/how-to-setup-a-github-pages-blog-with-markdown)
```

## Adding image tags to our posts


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

