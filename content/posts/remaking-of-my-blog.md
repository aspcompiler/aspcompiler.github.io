---
title: "The Remaking of My Blog with Hugo"
date: 2023-01-28T22:16:35-08:00
draft: false
---

# The remaking

I have not blogged since I joined Amazon in 2016. My initial focus was [delivery results](https://www.amazon.jobs/content/en/our-workplace/leadership-principles) and then I had to deal with a massive volume of customer interactions and data as the result. Now I want to blog again as  I spend significant time doing R&D at AWS Center for Quantum Computing and I need to capture what I learnt, and hopefully that benefits others as well. My original blog site was developed on [github pages using Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll). The Jekyll has changed significantly so that the side will not build.

# The first attempt: updating Jekyll

Updating Jekyll has not been a smooth ride. Jekyll is developed in [Ruby](https://www.ruby-lang.org/en/). The latest version of Ruby is currently 3.2 but github pages is only compatible with 3.1.3. So one actually has to start with a Ruby version manager such as [chruby](https://jekyllrb.com/docs/installation/macos/) to have multiple versions of Ruby in a system. Then one has to install Ruby Gems into per project environment using [bundler](https://jekyllrb.com/docs/ruby-101/). Then once every few months, I need to update my Gems and ensure that they are compatible with each other. That is a bit too much maintenance if I just need a tool to write a blog.

# Trying Hugo

[Hugo](https://gohugo.io/) is another popular static site generator written in [Go](https://go.dev/). Hugo developers have resolved all dependency conflicts at build time to make it a single executable so that the installation is extremely easy.

Hugo deploys hugo modules as git submodules so [git](https://git-scm.com/) is required. This is a bit harsh for non-software-developers but OK for me. Using git submodules for distributing/versioning dependencies definitely is not as easy as versioned packages. However, I have only one submodule which is my [Dairy theme](https://github.com/AmazingRise/hugo-theme-diary) which has more features than I currently need. Let us see how it goes.

# Getting Started

The installation is very straight-forward. Just follow the [instructions for each platform](https://gohugo.io/categories/installation/). 

Then one can just follow [quick start](https://gohugo.io/getting-started/quick-start/). When I get to the step of adding `ananke` theme, I jumped to the [Dairy instructions](https://github.com/AmazingRise/hugo-theme-diary#quick-start) instead.

# Configuration

Most of the configurations are in the `config.toml` file. One first needs to follow the instructions to set `baseUrl`, `title`, `theme`, etc.

## Configuring Main Menu

Configuring the main menu is pretty straightforward. Just add several `[[menu.main]]` elements. The double square-bracket means array in toml.

```
[[menu.main]]
url = "/about"
name = "About"
weight = 1
[[menu.main]]
url = "/posts"
name = "Posts"
weight = 2
[[menu.main]]
url = "/index.xml"
name = "RSS Feed"
weight = 3
```

Hugo recognizes some of the urls like `posts` and `index.xml` out of the box while we have to add contents for `about`.

## Comments

Hugo supports several comment systems from free to paid. One of the easiest free ones is [utterance](https://utteranc.es/) which uses github issues to store comments. The configuration for utterance is:

```
[params.utterances]
repo="aspcompiler/aspcompiler.github.io"
theme="github-light"
```

## Search

One of the easiest ways to get search functionality is to configure [google search](https://github.com/AmazingRise/hugo-theme-diary/wiki/Customization#add-google-search-box-on-your-site). 

## Syntax Highlighter

Dairy theme has a default [code highlighter](https://github.com/AmazingRise/hugo-theme-diary/wiki/Customization#about-highlight) and we will see how it goes.

## CI/CD

To deploy to `site.github.io`, one has to configure github action to build the site. This is basically adding and config the page `.github/workflows/gh-pages.yml`. Follow the [instructions](https://gohugo.io/hosting-and-deployment/hosting-on-github/) from Hugo.