# How to customize your github pages blog style in five minutes

Mar 12, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

[Part 6 - Setup third party services for your github pages blog](https://aregsar.com/blog/2019/how-to-setup-third-party-services-for-your-github-pages-blog)

[Part 7 - Github pages tech stack: Jekyll, Liquid, Markdown](https://aregsar.com/blog/2019/github-pages-tech-stack-jekyll-markdown-liquid)

Welcome to part 3 of this series on setting up a blog with Github pages.

In this blog post I will show you the steps I took to override the default theme style of my Github pages blog after I selected the default theme in part 1.

> Note: Jekyll, the engine behind github pages applies the selected theme during its markdown file transformation process. The default theme files for the caymen theme that I selected, are located in the jekyll themes repository at: [https://github.com/pages-themes/cayman](https://github.com/pages-themes/cayman).

## How styles are applied by Github pages

It would be helpful to describe how styles are applied by Github pages, before showing the steps I took to override the default theme style of this blog.

The default style that Jekyll applies for the cayman theme is located at [https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss](https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss).

Inside this file there is an import statement:

`@import ‘jekyll-theme-cayman’;`

This statement imports the styles located at [https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss](https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss)

By default Jekyll compiles the `https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss` file to a `style.css` file that is used to style this blog.

The compiled `style.css` is referenced using a `<link>` tag inside the `<head>` tag of the layout file used for this blog. To override the default `style.scss` file used by Jekyll to generate the `style.css` file, we need to add our own `style.scss` file to our repository that can be used to override the styles imported by the default `style.scss` file.

In the following sections I will detail the steps I took to override the default style of my github pages blog in approximately 5 minutes.

## Step 1 - Added a heading to the empty home page file

To test overriding the style I added a h1 tag to the `index.md` home page file in the root of the repository by adding the markdown content to the page using the bash statement below:

```bash
echo '# hello world' >> index.md
```

After I pushed this change to the remote repository, I refreshed the page and did a view source in the browser to verify that jekyll converted the markdown to the following html:

`<h1 id="hello-world">Hello World</h1>`

> Aside: You can see that the text set to the id attribute of the h1 tag is the transformed display text of the tag, where the space character is replaced by a dash and the text is lowercased.

## Step 2 - Created a local style.css file

To override the default theme style we need to add a local `assets/css/style.scss` file to our repository.

I added the `assets/css/style.scss` file by typing the following in a terminal window at the root directory of the repository:

```bash
mkdir assets && cd assets
mkdir css && cd css
echo '---' >> style.scss
echo '---' >> style.scss
echo >> style.scss
echo '@import "{{ "{{" }} site.theme }}";' >> style.scss
```

At this point the `style.scss` will contain the following content:

```scss
---
---

@import "{{ "{{" }} site.theme }}";
```

Jekyll transforms the `@import "{{ "{{" }} site.theme }}";` line in the file to `@import "jekyll-theme-cayman";` by using the setting `theme: jekyll-theme-cayman` specified in our `_config.yml` file.

So the content of style.scss file will be transformed to:

```scss
---
---

@import "jekyll-theme-cayman";
```

This is the same import statement used in the default theme style [https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss](https://github.com/pages-themes/cayman/blob/master/assets/css/style.scss) file.

Since we did not add any overriding styles to the `style.scss` file the theme of the site will remain the same.

I verified this by pushing my changes to the remote repository and refreshing the home page after the site is republished to see that the same default cayman style is still applied.

## Step 3 - Customized the theme style

Now that we have overriden the default `styles.scss` file with our local `styles.scss` file, we can add styles to our local `styles.scss` file to customize the styles for our pages.

To customize we can add any custom imports or css or scss styles immediately after the `@import "{{ "{{" }} site.theme }}";` line in our local styles.scss file.

Styles added to this file override the defautt styles at [https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss](https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss) that are applied by Jekyll to our pages.

To test this out, I added the `.main-content h1` style to my local `assets/css/style.scss` file, right after the import statement.

After I added the style, the `style.scss` file content looked like:

```scss
---
---

@import "{{ "{{" }} site.theme }}";
.main-content {
  h1 {
   color: #ff0000;
 }
}
```

I then saved and pushed the change to my Github pages repository and refreshed the page to verify that the color of the h1 heading that I added to the `index.md` file changed to red.

Viewing the source of the page, you can see that the added style overrides the color of the h1 tag by inspecting in the html snippet of the home page shown below:

```html
<main id="content" class="main-content" role="main">
      <h1 id="hello-world">hello world</h1>
```

The html was generated by Jekyll, from the transformed content of the `index.md` markdown file that was then injected into the `main` tag of the layout file.

Jekyll uses the local `assets/css/style.scss` file to generate the `style.css` that is applied to the page.

I verified this by viewing source on the refreshed page to see the published style.css link tag: `<link rel="stylesheet" href="/assets/css/style.css?v=f97443281054e55039f2bad9d2237e5486d287c7">`

To see the actual overiding style I navigated to the generated css `style.css` that is referenced in the `<link>` tag to see the style `.main-content h1{color:#ff0000}` which is applied to the `h1` tag inside the `main` tag shown again here:

```html
<main id="content" class="main-content" role="main">
      <h1 id="hello-world">hello world</h1>
```

So this is the way you can override other styles for your own Github pages site.

## Details of how overriding the styles works

As discussed in the beginning of this article, the statement `@import ‘jekyll-theme-cayman’;` imports the default cayman theme scss file located at [https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss](https://github.com/pages-themes/cayman/blob/master/_sass/jekyll-theme-cayman.scss)

If we open that file we will see the following style snipets inside:

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

This is the style that gets overriden when we added our own `.main-content h1` style to our local `style.scss` file.

Content from each of the other pages of the site, when published, also get injected inside the `main` tag of the layout file.

So any h1 headings in any of the pages of this blog will get the same stying.

We can similarly override other html tag styles, by adding more overriding styles in our local `style.scss` file.

## Conclusion

In this post I showed you how I added a local style file that you can use to add custom styles that override the built in default theme styles in the default theme style file.

I also showed you the file where the default theme style declarations reside to see the available styles to override.

Check out the next post in the series to see how I added a custom domain name for this blog.

Thanks for reading.

[Top](#how-to-customize-your-github-pages-blog-style-in-five-minutes)