
# Github pages tech stack: Jekyll, Liquid, Markdown

Apr 1, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

In this post I want to discuss the technologies that Github pages uses under the hood to take the markdown pages in your blog and transform and publish them as html content for consumption via web browsers.

This information may help you to have a better understanding of the content of this series of posts on publishin your own blog on Github pages.

Github pages uses standard Github repositories to host Github pages content. The URL structure of the content follows the repository structure.

## Jekyll transformation engine

The repository content is written in markdown and converted to Html markup using Jekyll.

Jekyll is the name of the engine that Github pages uses to process the markdown pages in the repository to convert them to standard Html pages while applying CSS theme styling to the Html content.

> Note: Any Html content will be published directly by Jekyll after CSS styles have been applied, without requiring to be transformed. In fact the layout file that Github pages uses is a standard Html file within which it injects the Html transformed markdown page content.

Every time we push a change to our Github pages repository, Jekyll reprocesses and republishes the new markup after a brief delay.

## Liquid Templating Language

Jekyll also uses a templating language called Liquid that contains Liquid tags to plug in html content from various sources. Jekyll uses Liquid tags  to plug in individual page content into layout templates and also to substitute text and metadata defined in settings files and other sources into the layout and pages.

Also GitHub Pages uses its own flavor of markdown called [Github Flavored Markdown](https://github.github.com/gfm/) or GFM. GFM is a flavor of Markdown that adds a few syntax extensions the allow for code blocks and syntax highlighting based on programming language.

You will not need to know GFM to understand this post.

You will need a Github account and a git client installed on your development machine.

### Overriding the default Jekyll layout

So after copying the default.html content,
I replaced the content in the `<body>` tags with:

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

Note that there exists a `{{ "{{" }} content }}` parameter in the html. This is a Liquid templating parameter that Jekyll uses by including the html content of a processed markdown page into the layout html file. In fact anywhere Jekyll sees Liquid template parameters, even in markdown page content or html code snippets in a markdown page, it will try to substitute content of another page inside.

So the way I was able to display the `{{ "{{" }} content }}` parameter right here in the markdown text of this blog post, was to escape it. Otherwise Jekyll will substitute another page content instead of just allowing the `{{ "{{" }} content }}` text to be displayed. So be aware of that if you want to print Liquid template parameters into your own markdown posts.

Refer to this StackOverflow post to see how to escape Liquid template parameters in your Github pages:

[How to escape liquid template tags](https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags)

The layout change removed all the {{ site.<REPO_SETTINGS> }} tags that github pages uses to add the buttons and content to our layout.

The REPO_SETTINGS parameters are specified by the default repository settings such as repo name. However next, we will specify the settings in the _config.yml file that will override the default settings.


