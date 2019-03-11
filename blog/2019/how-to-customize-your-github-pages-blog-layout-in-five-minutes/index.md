# How to customize your github pages blog layout in five minutes

Mar 8, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Welcome to part 2 of this series on setting up a blog with Github pages.

In this blog post I will show you the steps I took to override the default layout of my Github pages blog after I selected the default theme in part 1.

I will also show how to replace the default content that was included in the default theme layout file.

> Note: Jekyll, the engine behind github pages applies the selected theme during its markdown file transformation process. The default theme files are located in the jekyll themes repo located at https://github.com/pages-themes/cayman.

Below I will detail the steps I took to override the default layout of my github pages blog in approximately 5 minutes.

## Step 1 - Creating a local layout file

In this step I created a file named _layouts/default.html in the `aregsarblog` repository by typing the following bash statement in the root directory of the repository:

```bash
mkdir _layouts && cd _layouts
touch default.html
```

At this point we have an empty local layout file.

## Step 2 - Copying the default theme layout file content

The default layout for our cayman theme is located at: https://github.com/pages-themes/cayman/blob/master/_layouts/default.html

We need to copy the content of this file over into our local layout file in order to be able to override the layout.

In order to do so, I did the following steps:

+ I navigated to the default layout repo page
https://github.com/pages-themes/cayman/blob/master/_layouts/default.html
+ clicked on the raw button
+ copied the raw text into the local _layouts/default.html file that we created in step 1 using a text editor.

At this point the content of the default.html file in my local repo overrides the content in the https://github.com/pages-themes/cayman/blob/master/_layouts/default.html file.

However since we just copied over the exact same content that is in the default layout file, our blog home page will still look the same until we change the content of our local layout file `_layouts/default.html`.

You can verify this by pushing the changes we made so far to the Github repository and refresh the page to still see the same content displayed.

## Step 3 - Replacing the default theme layout file boilerplate

By default github pages themes add standard repository boilerplate markup
to the `default.html` file that is a mix of html tags and Liquid template tags.

To replace the boilerplate with my own layout page markup, I replaced the content inside the `<body>` tags in the `default.html` file with the following html snippet:

```html
<header class="page-header"  
        role="banner">
      <h1 class="project-name">Areg Sarkissian</h1>
</header>
<main id="content" class="main-content" role="main">
      {{ content }}
      <footer class="site-footer">
        &copy; 2019 Areg Sarkissian
      </footer>
</main>
```

After pushing this change to my Github repo, I saw the new content displayed in the page.

> Note the __content__ tag in the body section of the layout file is a Liquid template placeholder where Jekyll places the, converted to html, content of your markdown pages into the layout file. Liquid is the templating engine used by Jekyll

So at this point the content of our home page index.md file is converted to html by Jekyll and then injected into the layout where the placeholder tag is located.

Since we don't have any content in our home page yet, so nothing more then the layout content is displayed. In a future post in the series I will demonstrate adding sample content to show how we can change the default theme styling.

## Step 4 - Ovrride the default layout title and description meta tags

The `<head>` tag of `default.html` layout file content that we copied in step 2, contains a `{% seo %}` Liquid template tag
parameter that when processed by Jekyll, will generate title and description meta tags used for SEO.

The genrated title meta tag uses the Github repository name as the title text that gets displayed as the title for the blog in the web browser tab.

Similarly the genrated description meta tag uses the optional Github repository description that I enetered when I created the repo as the displayed description in the web browser tab.

To override the repository name and repository description text that are used by default, I added title and description settings to the `_config.yml` that was added by Github pages when I selected the theme in the part 1 of the series.

In order to do so I typed the following in the bash terminal:

```bash
echo ''
echo 'title: aregsar' >> _config.yml
echo 'description: Areg Sarkissians Blog' >> _config.yml
```

This set the title text to `aregsar` and the description text to `Areg Sarkissians Blog`.

Jekyll uses the settings in the `_config.yml` file to override the default values for title and description.

## Conclusion 

In this post I showed you how I overrode the default theme layout file and its content. I also showed how I overrode the default theme file title and description meta tags displayed on the browser tab.

You can further customize the content of the `default.html` layout file with your own Html content as I will detail when I show how I added global navigation links for this blog in another post in the series.

Checkout part 3 to see how I override the default theme style of Github pages.

Thanks for reading.
