
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

This information may help you to have a better understanding of the content of this series of posts on publishin your own blog on Github pages.

## Markdown files and Github Flavored Markdown

Github pages uses standard Github repositories to host Github pages content.
The URL path structure of the published pages follows the repository diectory hierarchy and content structure.

Each directory can contain a default markdown page named `index.md`. Each directory can also contain other markdown and html pages.

GitHub Pages uses its own flavor of markdown called [Github Flavored Markdown](https://github.github.com/gfm/) or GFM.

GFM is a flavor of Markdown that adds a few syntax extensions the allow for code blocks and syntax highlighting based on programming language.

## The Jekyll transformation engine

The markdown pages in the repository are converted to Html markup by Jekyll,.
The repository content that is written in markdown, is converted to Html markup using Jekyll,the engine that Github pages uses to process the markdown pages in the repository.

After the markdown is transformed to html is published to the public Github pages URL of the repository.

Any pure Html content will be published directly by Jekyll, without  transformation.

Every time we push a change to our Github pages repository, Jekyll reprocesses and republishes the new markup after a brief delay.

## The Liquid Templating Language

Jekyll also uses a templating language called Liquid that contains Liquid tags to plug in html content from various sources into markdown or html files.

Jekyll uses Liquid tags to plug in the transformed page content into layout templates and also to substitute text and metadata defined in settings files and other sources into the Liquid tags in the layout file and transformed pages.

## Plugging content into the default site layout

The layout file that Github pages uses for the pages in your repository is a standard Html file, within which it injects the Html transformed markdown page content.

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

This is a Liquid templating parameter that Jekyll uses to plug in the html content of a processed markdown page into the layout html file at the location of tag.

In fact anywhere Jekyll sees Liquid template parameters, even in markdown page content or html code snippets in a markdown page, it will try to substitute the transformed content of another markdown page inside.

> Aside: If you want to actually literally include Liquid tags in your pablished content,
you need to escape the tag. The way I was able to display the `{{ "{{" }} content }}` parameter right here in the markdown text of this blog post, was to escape it.
Otherwise Jekyll will substitute another page content instead of just allowing the `{{ "{{" }} content }}` text to be displayed.
Refer to this StackOverflow post to see how to escape Liquid template parameters in your Github pages: [How to escape liquid template tags](https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags)

The layout file also contains Liquid tags that such as `{{ "{{" }}site.<REPO_SETTING_PARAM> }}` that github pages uses to add the buttons and content to our layout.

The <REPO_SETTING_PARAM> value can be any default setting such the default repository name which will replace `{{ "{{" }}site.name }}`.

The default setting value can be overridden by adding a _config.yml file to the root of the repositiry that can contain name value pairs. 

So for instance if we have a `name` setting name in the _config.yml file, it's value will override the defualt repository name setting.

## Conclusion

In this post I detailed the basics of how Github pages processing engine Jekyll transforms markdown and html content, plugging in content and settings into Liquid templating tags in the process.

Thanks for reading