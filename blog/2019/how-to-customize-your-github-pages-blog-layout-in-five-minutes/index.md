# How to customize your github pages blog layout in five minutes

Mar 8, 2019 by [Areg Sarkissian](https://aregsar.com/about)

The Github pages default theme repo
All the Github pages default themes files that Jekyll uses to apply the themes during its markdown file transformation process are located in the jekyll themes repo located at https://github.com/pages-themes

Each individual theme is at located in a sub directory https://github.com/pages-themes/THEME_NAME

Since I chose the cayman theme, the theme repo that Jekyll uses by default for this site is at https://github.com/pages-themes/cayman

So the default layout for our cayman theme is at: https://github.com/pages-themes/cayman/blob/master/_layouts/default.html


Follow the steps below to overrid the default layout

Step 1 - Overriding the default layout file

First I created a file _layouts/default.html at the root of the aregsarblog repository.

mkdir _layouts
cd _layouts
touch default.html

Step 2 - Copying the default theme layout file content

> I navigated to the default layout repo page
https://github.com/pages-themes/cayman/blob/master/_layouts/default.html
> clicked on the raw button
> copied the raw text into the _layouts/default.html file

At this point the content of the default.html file in my local repo overrides
the content in the https://github.com/pages-themes/cayman/blob/master/_layouts/default.html
file

Step 3 - Replacing the default theme layout file boilerplate
By default github pages themes add standard repository boilerplate markup
to the default.html file

To replace the boilerplate with my own layout page markup, I replaced the content in the <body> tags 
in the default.html file with:

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


Step 4 - Replace the default layout title and description meta tags

The <head> tag of default.html file that we copied contains a
parameter that when processed by  github pages will generate title and description
 meta tags used for SEO.

The genrated title meta tag uses the repo name for title value that displays 
the site title in the web browser bar. Similarly the genrated description meta tag uses the Github repo description
you specified when you created the repo.

to override the repo name and repo
description that are used by default, I added title and secription settings to the _config.yml that was added by Github pages 

echo ''
echo 'title: aregsar' >> _config.yml
echo 'description: Areg Sarkissians Blog' >> _config.yml




Conclusion 


In this post I showed you how I overrode then replaced the default theme layout file
and also changed the title and description displayed on the browser tab.


Checkout part 3 to see how I override the default theme styling of Github pages
