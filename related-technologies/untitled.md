---
description: >-
  Easy way to publish a modern looking web site non-tech people can manage and
  update. All for free!
---

# How to Publish a no-code website in 10 minutes

## This page is outdated. Please [visit here for the most up-to-date content](https://www.secondstate.io/articles/how-to-learn-rust-without-installing-any-software/).

In this article, I'll introduce a no-code, no-software and no-cost solution to publishing sophisticated web sites managed by non-technical people. The full codebase is [on GitHub here](https://github.com/second-state/hugo-website/fork).

Sir Issac Newton discovered the law of gravity when practicing “social distancing” during the Plague. What will YOU do? One silver lining of quarantine is that all this free time brings out the entrepreneur spirit and creativity in us.

However, especially because of the quarantine, now more than ever, any new idea or project must have a web site. Traditional CMS solutions like Wordpress, Squarespace, or Wix are difficult to use, look dated, are expensive, or all of the above!

We wanted to create a web site that has a sophisticated look and feel, yet is easy to customize. A non-technical person should be able to edit the source files and see the changes appear on the live web site in a few minutes. Ideally, it should also be free \(forever free, not just free-for-now\), and can scale to handle millions of visitors if it becomes popular.

Is this possible?

In this short article, I will walk you through a solution based on the Hugo framework, GitHub Pages, and GitHub Actions. You can get up and running with your shiny new website by the end of this article.

> It is so easy that my 9-year-old son did it. He now manages a web site for his fictional country called [Arenztopia](https://www.arenztopia.com/). Check out the [back story](https://medium.com/@michaelyuan_88928/welcome-to-arenztopia-95cc85253163).

## TL;DR <a id="tl-dr"></a>

If you just want to get started with a working web site as soon as possible, first make sure that you have a free GitHub account.

[**Fork this GitHub repository**](https://github.com/second-state/hugo-website/fork) to your own account.

Go to your forked repository, and click on the Actions tab. You will see a message like the one in the picture below. **Click on** the “I understand my workflows …” button.

Go to the Settings tab of your repository, and then scroll down to GitHub Pages. **Re-select** the `gh-pages` from the dropdown for the web site to build.

Go to the Code tab of the repository, open the `config.toml` file, and edit it. **Change** its `title` attribute to something else, and click on the “Commit changes” button at the bottom. We need this step to trigger the workflow at the new repository.

Wait a few minutes, go to the “published at” web address at GitHub Pages and you will see the template web site.

Next, you can customize the site by editing the `config.toml` file and the files in the `content` folder. Go to the “Add your own content” section at the end of this article to see how. You can check out [the instructions for the Ananke theme here](https://github.com/budparr/gohugo-theme-ananke#getting-started).

That’s it for the quick start! In the next several sections, I will explain in more detail the concepts and processes.

## Hugo basics <a id="hugo-basics"></a>

Older generation CMS solutions like Wordpress are too difficult to adapt to new web site designs, and commercially hosted solutions like SquareSpace are too expensive and not flexible enough. Static web site generators like Hugo offer a good balance among features, flexibility, and ease of authoring.

* Hugo web sites can be customized and modified through configuration files.
* New pages and content sections can be written in markdown instead of HTML. That makes content authoring much easier.
* There are many modern design themes to choose from.
* The output of Hugo is a static HTML web site that can be deployed on any low-cost hosting provider.
* Together with static web site hosting services like GitHub Pages and Netlify, they can offer a completely free solution.

The Hugo software distribution is [available](https://gohugo.io/getting-started/installing/) on all major operating systems. You can [learn about it here](https://gohugo.io/getting-started/quick-start/). But, since we will use GitHub Actions to automatically build our Hugo web sites, we do not actually need to install Hugo software here.

Here is how to do it.

## Create a template website <a id="create-a-template-website"></a>

First, select a Hugo theme. There are [many](https://themes.gohugo.io/). Some are geared toward web sites with one or more content web pages, while others are optimized for blog-like sites with timestamped posts.Hugo themes

Find one you like, download a zip package or clone a GitHub repo, and unpack the theme into a folder. Let’s assume that the theme distribution is unpacked into a folder called `my-theme`. The following are commands in a Linux terminal. You could use the Terminal app on Mac or PowerShell on Windows.

Next, create your web site project directory on your computer.

```text
$ mkdir -r my-site/themes
```

Copy the theme folder into your project.

```text
$ cp -r my-theme my-site/themes
```

Next, copy the theme’s `exampleSite` to the project’s top-level directory.

```text
$ cd my-site
$ cp -r themes/my-theme/exampleSite/* ./
```

Edit the `config.toml` in the project root directory `my-site/`  to point to the right theme.

```text
baseURL = "/"themesDir = "themes"theme = "my-theme"
```

Next, create a GitHub repo called `my-site`, and push the `my-site` directory onto its `master` branch. Here are the steps for [uploading files from GitHub’s web UI](https://help.github.com/en/github/managing-files-in-a-repository/adding-a-file-to-a-repository). Now we are ready to publish the theme example site.

For a Hugo-based system to be useable to a non-developer \(or a young developer who has yet to master the command line tools\), we must automate the process of building and deploying the static web site.

## Automate deployment <a id="automate-deployment"></a>

In the GitHub project, go to Settings and enable GitHub Pages. Select the source to be the `gh-pages` branch.Settings, GitHub Pages

Next, we create a GitHub Actions workflow to run the Hugo command on the source files from `master` branch, and push the generated HTML files to the `gh-pages` branch for publication. From the project’s Actions tab, click on the “set up a workflow yourself” button.Set up a workflow yourself

The workflow is stored in the `master` branch as `.github/workflows/main.yml` file. The content of the file is as follows.

```text
name: github pages

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1  # v2 does not have submodules option now
        # with:
        #   submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.62.2'
          extended: true

      - name: Build
        run: hugo

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

What happens here is that the web site authors and editors will change content and files on the `master` branch. Whenever new content is pushed to the `master` branch, the automated GitHub Actions workflow will [set up the Hugo software](https://github.com/peaceiris/actions-hugo/blob/master/README.md), run the `hugo` command, and turn those files into HTML files for a static web site.

The HTML files are [pushed](https://github.com/peaceiris/actions-gh-pages/blob/master/README.md) to the `gh-pages` branch of the same repository. They will be published on the specified web address by GitHub Pages as configured.

Notice the `cname` attribute in the last line. That is the [custom domain name](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site) we set up with GitHub Pages. If you do not have a custom domain name, just remove this line, and you can access your web site at the domain provided by GitHub Pages.

Now go to the web site, and you should see the theme’s default web page.The HugoSerif template for one of our web sites.

![The HugoSerif template for one of our web sites.](../.gitbook/assets/image%20%287%29.png)

h

## Add your own content <a id="add-your-own-content"></a>

To change the default theme web site to your own content, you just need to change the files on the `master` branch. Please refer to the [documentation](https://gohugo.io/content-management/organization/) of your selected theme. In general, Hugo templates work like this:

* The web pages are authored in markdown format and the `md` files are in the `content` folder.
* Each `md` file has a header section with properties such as the page’s menu placement, priority, timestamp, excerpt, etc.
* The overall configuration, such as the menu items and properties used by multiple pages, are stored in the `data` folder.
* Static content such as raw HTML files, JavaScript files, and image files can be placed in the `static` folder.

In particular, here is how you customize the Ananke theme that comes with our template:

* The [config.toml](https://github.com/second-state/hugo-website/blob/master/config.toml) file allows you to configure the website title, social icons on all pages, and the big featured image on the home page.
* All images should be uploaded to the [static/images](https://github.com/second-state/hugo-website/tree/master/static/images) folder.
* The [content/\_index.md](https://github.com/second-state/hugo-website/blob/master/content/_index.md) file contains the text for the home page.
* To add pages to the site, you can just create [markdown](https://guides.github.com/features/mastering-markdown/) files in the [content](https://github.com/second-state/hugo-website/tree/master/content) folder. An example is the [contact.md](https://github.com/second-state/hugo-website/blob/master/content/contact.md) file. Notice that at the top of the file, there are attributes to control whether this page should be on the website menu.
* To add articles to the site, you can create markdown files in the [content/post](https://github.com/second-state/hugo-website/tree/master/content/post) folder. Those are blog-like content articles that have dates and titles at the top. The most recent two articles will show up on the home page.

If you are interested in learning more and see how we did it, you can watch our progress at

* The Country of Arenztopia \[[GitHub](https://github.com/juntao/arenztopia)\] \[[Web site](https://www.arenztopia.com/)\]
* Second State blog \[[GitHub](https://github.com/second-state/blog)\] \[[Web site](https://blog.secondstate.io/categories/en/)\]

Good luck and stay healthy!





