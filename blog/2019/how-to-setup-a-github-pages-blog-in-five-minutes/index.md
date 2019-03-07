# How to setup a github pages blog with markdown

Mar 7, 2019 by [Areg Sarkissian](https://aregsar.com/about)

Welcome to the first post on my new blog.

In this post I will show you how I set up this blog with [GitHub Pages](https://pages.github.com/), [Markdown](https://commonmark.org/help/) and bash commands and how you can easily do the same.

Below I will detail the steps I took to set up my github pages blog in approximately 5 minutes.

## Step 1 - Created a empty standard Github repository

In this step I signed in to my Github account and used the steps below to create an empty github repository named `aregsarblog` to host my blog.

+ navigated to the github "create a new repository" page
+ entered "aregsarblog" in the "repository name" field
+ enetered "aregs blog" in the optional "description" field
+ clicked on the "create repository" button

At this point my empty Github repository is created.

> Note: The Github pages repository must be a public repository if using a free github account

## Step 2 - Created a local git repository

In this step I created a local directory named `aregsarblog` and inside the directory I initialized a local git repository and associated it with the remote `aregsarblog` Github repository.

```bash
mkdir aregsarblog
cd aregsarblog
git init
git remote add origin git@github.com:aregsar/aregsarblog.git
```

At this point my local Github repository is created and is ready to track my remote Github repository.

## Step 3 - Added a markdown index page

In this step I added and empty `index.md` markdown file which will be the home page of our site.

I committed and pushed the file to the remote repository with the local master branch set up to track remote master branch.

```bash
touch index.md
git add index.md
git commit -m "first commit"
git push -u origin master
```

At this point my Github pages home page `index.md` file exists in the Github repository.

## Step 4 - Turned the standard Github repository into Github pages repository

To activate Github pages I did the following:

+ Navigated to the settings page of my `aregsarblog` repository.
+ On the settings page, I scrolled down to the Github pages section.
+ In the Github pages section, I seleted the master branch using the select source dropdown.

Once I selected the source branch, the message below appeared at the top of the github pages section:

`your site is published https://aregsar.github.io/aregsarblog/`

You can navigate to that URL to see the published page.
Currently it only displays the name of the repository.

The displayed content is part of the default layout of the page. I will have a post in the series that will cover how to customize the layout and content of the page.

In the next step I added a theme from the default themes available.

## Step 5 - Selecting a Github pages default theme

To give my Github pages default theme I did the following:

+ I navigated to the settings page of my `aregsarblog` repository.
+ On the settings page, I scrolled down to the Github pages section.
+ In the Github pages section, I Click on the change theme button.
+ Chose the cayman theme which was auto selected by default.
+ Clicked the select theme button.

Clicking the select theme buttom returned me to the settings page and while adding a `_config.yml` file to the root directory of my remote repository.

To pull down the `_config.yml` file to my local repo did a `git pull`

The `_config.yml` file that was added contains the line:

theme: jekyll-theme-cayman

This tells the Github pages processing engine what theme to select and apply to our proceessed markdown pages when publishing the site.

The actual theme name is the last segment which in our case is cayman.

You can navigate again to the URL published page at.

`https://aregsar.github.io/aregsarblog/` to see the updated page.

> Note: The Github pages processing engine takes some time to publish your pages after a change. So keep refreshing the page until the new content appears.

## Conclusion

If you followed my steps, then congratulations, you just created your basic GitHub Pages blog site.

Be sure to check out part 2 of this post to see how I override the default theme to be able to customize it.

Thanks for reading.
