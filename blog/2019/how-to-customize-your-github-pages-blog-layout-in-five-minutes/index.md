# How to customize your github pages blog layout in five minutes

Mar 8, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

[Part 6 - Setup third party services for your github pages blog](https://aregsar.com/blog/2019/how-to-setup-third-party-services-for-your-github-pages-blog)

[Part 7 - Github pages tech stack: Jekyll, Liquid, Markdown](https://aregsar.com/blog/2019/github-pages-tech-stack-jekyll-markdown-liquid)

Welcome to part 2 of this series on setting up a blog with Github pages.

In this blog post I will show you the steps I took to override the default layout of my Github pages blog after I selected the default theme in part 1.

I will also show how to replace the default content that was included in the default theme layout file.

> Note: Jekyll, the engine behind github pages applies the selected theme to the 
markdown files after they are transformed to html during its markdown file transformation process. The applied theme layout files are stored in a page themes Github repository. For the cayment theme the default theme files are located in the jekyll themes repository located at https://github.com/pages-themes/cayman.

Below I will detail the steps I took to override the default layout of my github pages blog in approximately 5 minutes.

## Step 1 - Creating a local layout file

In this step I created a empty file named `_layouts/default.html` in the `aregsarblog` repository by typing the following bash statements in the root directory of the repository:

```bash
mkdir _layouts && cd _layouts
touch default.html
```

At this point we have an empty local layout file `default.html`.

## Step 2 - Copying the default theme layout file content

The default layout for our cayman theme is located at: `https://github.com/pages-themes/cayman/blob/master/_layouts/default.html`

We need to copy the content of this file over into our local layout file in order to be able to override the layout.

In order to do so, I performed the following steps:

+ I navigated to the default layout repo page
[https://github.com/pages-themes/cayman/blob/master/_layouts/default.html](https://github.com/pages-themes/cayman/blob/master/_layouts/default.html)
+ Clicked on the `raw` button
+ Selected all the raw text content of the file and copied it
+ Using a text editor, I opened the empty local `_layouts/default.html` file that I created in step 1 and pasted the copied text content into it

At this point the content of the `_layouts/default.html` file in my local repository overrides the content in the `https://github.com/pages-themes/cayman/blob/master/_layouts/default.html` layout file.

However since we just copied over the exact same content that is in the default layout file, our blog home page will still look the same.

You can verify this by pushing the changes we made so far to the Github repository and refresh the page to still see the same content displayed.

In order to change the content we need to change the content of our local layout file `_layouts/default.html`.

## Step 3 - Replacing the default theme layout file content

By default github pages themes add standard repository boilerplate markup
to the `default.html` file. This content is a mix of html tags and Liquid template tags.
Liquid is the templating engine used by Jekyll.

To replace the boilerplate with my own layout page markup, I replaced the content inside the `<body>` tags in the `default.html` file with the following html snippet:

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

After pushing this change to my Github repo, I saw the new content displayed in the page.

> Note the __content__ tag in the body section of the layout file is a Liquid template placeholder where Jekyll places the, converted to html, content of your markdown pages into the layout file. A later post in the series will describe how the Github pages engine Jekyll uses the Liquid templating language to inject content.

So at this point the content of our home page `index.md` file is converted to html by Jekyll and then injected into the layout where the placeholder tag is located.

Since we don't have any content in our home page yet, nothing more then the layout content is displayed. You can verify this by refreshing the published page.

In a future post in the series I will demonstrate adding sample markdown content to the `index.md` page to show how we can change the default theme styling.

## Step 4 - Override the default layout title and description meta tags

The `<head>` tag of `default.html` layout file content that we copied in step 2, contains a `seo` Liquid template tag that when processed by Jekyll, will generate title and description html meta tags used for SEO purposes.

The generated `title` meta tag uses the Github repository name as the title text that gets displayed in the web browser tab.

Similarly the genrated `description` meta tag uses the optional Github repository description text that I enetered when I created the repository.

To override the repository name and repository description text that are used by default, I added title and description settings to the `_config.yml` file that was added by Github pages when I selected the theme in part 1 of the series.

Specifically, I typed the following bash statements in the terminal to add two additional lines to the `_config.yml` file.

The first line sets the title text to `aregsar` and the second line sets the description text to `Areg Sarkissians Blog`.

```bash
echo '' >> _config.yml
echo 'title: aregsar' >> _config.yml
echo 'description: Areg Sarkissians Blog' >> _config.yml
```

Jekyll uses these settings in the `_config.yml` file to override the default repository title and description values used when publishing the site.

Finally I pushed the changes to _config.yml file to the remote repository to see the overriden title in the browser tab after the site is republished.

## Conclusion

In this post I showed you how I overrode the default theme layout file and its content. I also showed how I overrode the default theme layout file title and description meta tags.

You can further customize the content of the `default.html` layout file with your own html content. 

I will detail how I added global navigation links to the layout file for this blog in an upcoming post in the series.

Check out part 3 to see how I override the default theme style of my Github pages blog.

Thanks for reading.

[Top](#how-to-customize-your-github-pages-blog-layout-in-five-minutes)
