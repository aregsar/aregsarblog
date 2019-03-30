
# Github pages tech stack: Jekyll, Liquid, Markdown

Mar 20, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

[Part 6 - Setup third party services for your github pages blog](https://aregsar.com/blog/2019/how-to-setup-third-party-services-for-your-github-pages-blog)

[Part 7 - Github pages tech stack: Jekyll, Liquid, Markdown](https://aregsar.com/blog/2019/github-pages-tech-stack-jekyll-markdown-liquid)

Welcome to the final part in this series of posts.

In this post I want to discuss the technologies that Github pages uses under the hood to take the markdown pages in your blog and transform and publish them as html content for consumption via web browsers.

## Markdown files and Github Flavored Markdown

Github pages uses standard Github repositories to host Github pages content.
The URL path structure of the published pages follows the repository directory hierarchy and content structure.

Each directory can contain a default markdown page named `index.md`. Each directory can also contain other markdown and html pages.

GitHub Pages uses its own flavor of markdown called [Github Flavored Markdown](https://github.github.com/gfm/) or GFM.

GFM is a flavor of Markdown that adds a few markdown syntax extensions for displaying programming language code blocks.

## The Jekyll transformation engine

The repository content that is written in markdown, is converted to html markup using [Jekyll](https://jekyllrb.com/), the engine that Github pages uses to publish the markdown pages in the repository.

After the markdown is transformed to html, it is published to the public Github pages URL of the repository.

Every time we push a change to our Github pages repository, Jekyll reprocesses and republishes the new markup after a brief delay.

## The Liquid Templating Language

Jekyll also uses a templating language called [Liquid](https://shopify.github.io/liquid/). Jekyll uses Liquid tags embedded in markdown pages and in the html layout file to plug in content from various sources into the rendered page.

To layout the pages in this blog, Jekyll plugs in the transformed markdown page content into the Liquid tag `{{ "{{" }} content }}` that is a placeholder, in the layout template, for the transformed markdown page. 

Jekyll also substitutes text and metadata, defined in settings files and other sources, into Liquid tags in the layout file and Liquid tags in markdown pages.

## Plugging page content into the default site layout

The layout file that Github pages uses to layout the pages in the repository is a standard html file, within which Jekyll injects the html content of the transformed markdown page.

Below is a snippet of code from the layout file of this blog:

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

Note that there exists a `{{ "{{" }} content }}` parameter in the html.

This is a Liquid templating tag that Jekyll uses to plug in the html content of a processed markdown page into the layout html file at the location of tag.

In fact anywhere Jekyll sees Liquid template parameters, even in markdown page content or html code snippets inside a markdown page, it will try to substitute the transformed content of another markdown page inside.

> Aside: If you want to literally include Liquid tags in your published content to be displayed as is in a page,
you need to escape the Liquid tag. Otherwise Jekyll will substitute transformed content of another page into the tag, instead of just allowing the text to be displayed as is.
The way I was able to display the `{{ "{{" }} content }}` parameter tag right here in the markdown text of this blog post, was to escape it. Refer to this StackOverflow post to see how to escape Liquid template parameters in your Github pages: [How to escape liquid template tags](https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags)

## Plugging repository setting values into the default site layout

The layout file also contains Liquid tags in the format of `{{ "{{" }} site.<REPO_SETTING_PARAM> }}` that github pages uses to add buttons and other content to the layout file.

The <REPO_SETTING_PARAM> parameter in the tag can be the name of any default setting for the repository. When Jekyll processes the files, it will substitute the value of that setting in the tag.

For example the  `{{ "{{" }} site.name }}` setting uses the default repository name `aregsarblog` for the `site.name` parameter.

The default setting value can be overridden by adding a _config.yml file to the root of the repositiry that can contain name value pairs for the settings to override.

So for instance if we have a `name=aregsar` setting in the _config.yml file, the value `aregsar` will override the defualt repository name `aregsarblog` in the `{{ "{{" }} site.name }}` tag.

## Conclusion

In this post I detailed the basics of how Github pages processing engine, Jekyll, transforms markdown and html content, plugging in content and settings into Liquid templating tags in the process.

Thanks for reading