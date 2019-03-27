# How to setup a Github pages blog in five minutes

Mar 7, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Articles in this series:

[Part 1 - Setup a Github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-github-pages-blog-in-five-minutes)

[Part 2 - Customize your github pages blog layout in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-layout-in-five-minutes)

[Part 3 - Customize your github pages blog style in five minutes](https://aregsar.com/blog/2019/how-to-customize-your-github-pages-blog-style-in-five-minutes)

[Part 4 - Setup a custom domain for your github pages blog in five minutes](https://aregsar.com/blog/2019/how-to-setup-a-custom-domain-for-your-github-pages-blog-in-five-minutes)

[Part 5 - Setup your github pages blog structure in five minutes](https://aregsar.com/blog/2019/how-to-setup-your-github-pages-blog-structure-in-five-minutes)

[Part 6 - Setup third party services for your github pages blog](https://aregsar.com/blog/2019/how-to-setup-third-party-services-for-your-github-pages-blog)

[Part 7 - Github pages tech stack: Jekyll, Liquid, Markdown](https://aregsar.com/blog/2019/github-pages-tech-stack-jekyll-markdown-liquid)

Welcome to the first post on my new blog.

In this post I will show you how I set up this blog using [GitHub Pages](https://pages.github.com/) and [Markdown](https://commonmark.org/help/) and how you can easily do the same.

Below I will detail the steps I took to set up my github pages blog in approximately 5 minutes.

## Step 1 - Created an empty standard Github repository

In this step I signed in to my Github account and used the steps below to create an empty github repository named `aregsarblog` to host this blog.

Below are the steps I used to create the repository:

+ Navigated to the "create a new repository" page on Github
+ Entered "aregsarblog" in the "repository name" field
+ Entered "aregs blog" in the optional "description" field
+ Clicked on the "create repository" button

At this point an empty Github repository is created.

> Note: The Github pages repository must be a public repository if using a free github account

## Step 2 - Created a local git repository

In this step I created a local directory named `aregsarblog` and inside the directory I initialized a local git repository and associated it with the remote `aregsarblog` Github repository.

I opened a bash terminal and typed the following:

```bash
mkdir aregsarblog && cd aregsarblog
git init
git remote add origin git@github.com:aregsar/aregsarblog.git
```

At this point the local git repository is created and is ready to track my remote Github repository.

## Step 3 - Added a home page

In this step I added an empty `index.md` markdown file which is the home page of this blog.

I committed and pushed the file to the remote repository with the local master branch set up to track the remote master branch.

In the bash terminal I typed the following:

```bash
touch index.md
git add index.md
git commit -m "first commit"
git push -u origin master
```

At this point the Github pages home page `index.md` file exists in the root directory of the Github repository.

## Step 4 - Turned the standard Github repository into a Github pages repository

To activate Github pages I did the following:

+ Navigated to the settings page of my `aregsarblog` repository on Github.
+ On the settings page, I scrolled down to the Github pages section.
+ In the Github pages section, I seleted the `master` branch using the select source dropdown.

Once I selected the source branch, the message below appeared at the top of the github pages section:

`Your site is published at https://aregsar.github.io/aregsarblog/`

> Note: for a brief period of time, until Github pages publishes your blog, you may see the message:
Your site is ready to be published at https://aregsar.github.io/aregsarblog/.
keep refreshing the repository settings page until the message changes to the site is published message.

Once the site is published message is displayed, you can navigate to the published URL for your own site to see the published page. At this point the page only displays the name of the repository.

The displayed content is part of the default layout template used to layout the `index.md` page. I will have a future post in the series that will cover how to customize the layout and content of the page.

## Step 5 - Selected a Github pages default theme

In this step I added a theme from the default themes available from the Github pages section in the settings page.

To select a theme for my Github pages site, I did the following:

+ I navigated to the settings page of my `aregsarblog` repository.
+ On the settings page, I scrolled down to the Github pages section.
+ In the Github pages section, I clicked on the change theme button.
+ I chose the cayman theme which was auto selected by default.
+ Clicked the select theme button.

Clicking the select theme buttom returned me to the settings page and added a `_config.yml` file to the root directory of my remote repository.

To pull down the `_config.yml` file from my remote repository to my local repository, In the bash terminal I typed: `git pull`.

Inspecting the added `_config.yml` file, you can see that it only contains the single line:

`theme: jekyll-theme-cayman`

This line tells Jekyll, the Github pages processing engine, the theme to select and apply to our processed markdown pages when publishing the site.

The actual theme name is the last segment which in my case is `cayman`.

To see the updated page with the selected theme applied, you can navigate again to the URL of the published page.

> Note: The Github pages processing engine takes a little bit of time to publish your pages after a change to the repository. So keep refreshing the published page until the new content appears.

## Conclusion

If you followed my steps outlined in this post, then congratulations, you just created your basic GitHub Pages blog site.

Be sure to check out part 2 of this post to see how I override the default theme layout to be able to customize it.

Thanks for reading.

[Top](#how-to-setup-a-github-pages-blog-in-five-minutes)
