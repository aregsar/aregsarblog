# How to customize your github pages blog style in five minutes

Mar 11, 2019 by Areg Sarkissian

Welcome to part 3 of this series on setting up a blog with Github pages.

In this blog post I will show you the steps I took to override the default theme style of my Github pages blog after I selected the default theme in part 1.

> Note: Jekyll, the engine behind github pages applies the selected theme during its markdown file transformation process. For the cayment theme the default theme files are located in the jekyll themes repository located at https://github.com/pages-themes/cayman.

## How styles are applied by Github pages

Before showing the steps I took to override he default theme style of this blog, It would be helpful to describe how styles are applied by Github pages.

The default style that Jekyll applies for the cayman theme is located at https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss.

Inside this file there is an import statement:

`@import ‘jekyll-theme-cayman’;`

This statement imports the styles located at https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss

By default Jekyll compiles the `https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss` file to a `style.css` file that used to style this blog.

The compiled `style.css` is referenced using a `<link>` tag inside
the `<head>` tag of `_layouts/default.html` layout file.

Here is an example of the generated tag:

`<link rel="stylesheet" href="/assets/css/style.css?v=76e9344533f4785fd14d43d0d2033d97bf6636b5">`

In the following sections I will detail the steps I took to override the default style of my github pages blog in approximately 5 minutes.

## Step 1 - Added a heading to the empty home page file

To test overriding the style I added a h1 tag to `index.md` file in the root of the repository by adding the markdown content below:

```bash
echo '# hello world' >> index.md
```

After I pushed this change to the remote repository,
I refreshed the page and did a view source in the browser to verify that jekyll converted the markdown to the following html:

`<h1 id="hello-world">hello world</h1>`

> Aside: You can see that the text of the h1 tag markdown is used a the id of the h1 html tag with a dash used for the space character

## Step 2 - Created a local style.css file

To override the default theme style we need to add a local `assets/css/style.scss` file to our repository.

So I added the `assets/css/style.scss` file by typing the following from root directory of the repository:

```bash
mkdir assets && cd assets
mkdir css && cd css
echo '---' >> style.scss
echo '---' >> style.scss
echo >> style.scss
echo '@import "{{ site.theme }}";' >> style.scss
```

At this point the `style.scss` will contain the following content:

```scss
---
---

@import "{{ site.theme }}";
```

Jekyll transforms the `@import "{{ site.theme }}";` line in the file to 
`@import "jekyll-theme-cayman";` by using the setting `theme: jekyll-theme-cayman` specified in our `_config.yml` file.

So the tranformed content will be:

```scss
---
---

@import "jekyll-theme-cayman";
```

This is the same import statement used in the default theme style https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss file.

## step 3 - Customized the theme styles

Now that we have overriden the default `styles.scss` file with our local `styles.scss` file, we can add styles our local `styles.scss` file to customize the styles for our pages.

To customize we add any custom imports or css or scss styles immediately after the `@import "{{ site.theme }}";` line in our local styles.scss file.

Styles added to this file override the defaut styles applied by Jekyll to our pages.

To test this out, I added the `.main-content h1` style to the `assets/css/style.scss` right after the import statement.

After I added the style, the `style.scss` file content looked like:

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

I then saved and pushed the change to my Github pages repo and refreshed the page to verify that the color of the h1 heading that I added to the `index.md` file changed to red.

The added style overrode the color of the `.main-content h1` tag, that was generated from content of `index.md` file and injected into the main tag of the layout file.

After pushing the change to the remote repository
Jekyll uses the `assets/css/style.scss` file to generate the `style.css` that is applied to the page.

I verified this by doing a view source on the refreshed page to see the published style.css link tag: `<link rel="stylesheet" href="/assets/css/style.css?v=f97443281054e55039f2bad9d2237e5486d287c7">`

To see the overide style I navigated to the generated css `style.css` reference in the `<link>` tag to see the style `.main-content h1{color:#ff0000}` which is applied to the h1 tag inside the main tag:

```html
<main id="content" class="main-content" role="main">
      <h1 id="hello-world">hello world</h1>
```

## Details of overriding styles

As discussed in the beginning of this article, the statement
`@import ‘jekyll-theme-cayman’;` imports the default cayman theme scss file located at https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss

If we open the file we will see the following style snipets inside:

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

As you can see there already exists a `.main-content h1` style in the file.
This is the style that gets overriden when we added the our own `.main-content h1` style to our local `style.scss` file.

## Conclusion