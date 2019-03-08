# How to customize your github pages blog style in five minutes

Github pages default theme styles

The default style that Github pages applies for the cayman theme is located at https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss

inside this file there is import statement @import ‘jekyll-theme-cayman’;

that imports the styles located at

https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss

By default Jekyll compiles this style.scss file to style.css that used to style the page. The compiled style.css is referenced in the <head> tag of _layouts/default.html file using the <link> element shown below:

<link rel="stylesheet" href="/assets/css/style.css?v=76e9344533f4785fd14d43d0d2033d97bf6636b5">


Step 1 Overriding the default theme styles

In order to override the default style.scss we need to first create a file assets/css/style.scss at the root of your repo and add the following four lines at the top the file

---
---

@import "jekyll-theme-cayman";
the @import "jekyll-theme-cayman"; line is transformed to @import "jekyll-theme-cayman"; by Jekyll because of the line theme: jekyll-theme-cayman specified in our _config.yml file.

This is the same import statement used in the default https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss file.

I added the assets/css/style.scss with the bash statements below at the root of this repo.

mkdir assets && cd assets
mkdir css && cd css
echo '---' >> style.scss
echo '---' >> style.scss
echo >> style.scss
echo '@import "jekyll-theme-cayman";' >> style.scss
I pushed the changes for the site to use the local style.scss to override the default theme style.scss

The local style.scss will be used to generate the style.css that is applied to the page.

You can verify this after a brief delay by doing a view source on the refreshed page to see the published style.css link

<link rel="stylesheet" href="/assets/css/style.css?v=c9149429e7d8df2dde257387400cef49363fb589">

Customizing the theme styles
Now that we have overriden the default styles.scss file with our local styles.scss file, we can customize our local styles.scss file to customize the styles for our pages.

To customize we add any custom imports or css or scss styles immediately after the '@import "jekyll-theme-cayman";' line in our local styles.scss file.




Step 2 - Adding a heading to our index.md page

To test changing the styling by adding my own style I added markdown heading content to index.md to 
generate an h1 tag that I could override the styling for


The Jekyll engine substitutes the content of the index.md file for the {{ content }} parameter in the _layouts/default.html page

All subsequent markddown pages you add to the repo will similarly be substituted for the {{ content }} parameter when published

Now in the index.md file I added the h1 tag markdown line:

# hello world

using the bash script:

echo '# hello world' >> index.md
I saved and pushed the changes.

After some delay you can refresh the page and do a view source in the browser to see that jekyll converted the markdown to the following html:

<h1 id="hello-world">hello world</h1>

> note that the text of the h1 tag markdown is used a the id of the h1 html tag with a dash used for the space character

=================

Step 3 - adding our own styles

As an example if we open the imported default cayman theme scss file located at https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss

we will see the following style snipets inside:

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
So we can as an example override the color of the .main-content h1 tag setting it to red by placing the following style snippet right after the import statement in our local style.scss file

.main-content {
  h1 {
   color: #ff0000;
 }
}
so the final file content would be

---
---

@import "jekyll-theme-cayman";
.main-content {
  h1 {
   color: #ff0000;
 }
}

Push the changes and see that the color of our h1 heading we added to index.md changes to red.

The generated .main-content h1 style is

.main-content h1{color:#ff0000}
for the h1 element in main element

<main id="content" class="main-content" role="main">
      <h1 id="hello-world">hello world</h1>
which can be seen by doing view source and navigating to the generated css file referenced in the header

<link rel="stylesheet" href="/assets/css/style.css?v=f97443281054e55039f2bad9d2237e5486d287c7">


