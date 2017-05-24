---
layout: page
res_class: Development
title: Building a Basic Jekyll Website
concept: What I've done & how I did it.
---

By: Simon Goring

***

**Table of Contents**

* TOC
{:toc}

***

# The Motivation

I've used a [Wordpress](http://wordpress.com) website for a long time now, mostly as a [blogging platform](http://downwithtime.wordpress.com), but it also included a place to report my [Publications](https://downwithtime.wordpress.com/publications/), a [Mentoring Statement](https://downwithtime.wordpress.com/mentoring-statement/) and some other things.

I've become acutely aware that a blog is only really useful so long as it is continually updated.  Once it starts getting stale then it becomes a liability.  It looks abandoned.

So, I wanted to redesign my website, I wanted to be able to add some of the interactive documents I've been developing, and I wanted a bit more control over the site.

So let's start with GitHub pages.

# GitHub Pages

Anyone with a GitHub account has access to a GitHub page.  Setting one up means that you can have an `http://username.github.io` website, but you can also redirect a URL that you own to your GitHub page.  GitHub maintains good tutorials, both on the basics of a [GitHub page](https://pages.github.com/) and also on [redirecting to a custom domain](https://help.github.com/articles/using-a-custom-domain-with-github-pages/).

GitHub pages can be as simple as a single `index.html` page, or as complex as you want.  So long as the site is ultimately served up as HTML you're good to go.  So, how do we build a webpage?

# Choosing Jekyll

## Conditions for a Back-End

I have a few conditions, and needs.  In particular I've been using RMarkdown a lot lately.  I have been generating tutorials directly in RMarkdown, but also rendering some to HTML to take advantage of some of the `d3.js` applications that have been ported to R (the nice [networkD3](https://cran.r-project.org/package=networkD3) package for example).  Most of these just live in my GitHub repository, either rendered or raw, but I'd like to be able to serve them up in one place, so, condition one:

	1.  Must support HTML and Markdown

I'm not great at HTML, and I generally find editing it directly really frustrating.  When pages start getting complicated I've totally screwed pages up and have basically had to abandon my plans.  It's partly a function of the fact I'm not a web developer, but it's also a function of the fact that HTML's markup is so extensive.  I want a clean palette to edit while still being able to customize my site & documents.

	2. Documents must be plain text as much as possible, but still be extensible.

I also want something that's going to be supported fairly widely, that's open, and that I can (theoretically) contribute to.

	3.  Must be open source.

It should be fairly obvious that, at this point, Jekyll seems to be the best option for me.  Maybe there's a bit of *post hoc* justification here, but in general I think I'm happy with my conditions.

## How Jekyll Meets the Conditions

**Must support HTML and Markdown** Jekyll uses a back end that supports both Markdown and HTML.  With a little bit of configuration it can support multiple different types of Markdown (the default is [kramdown](http://kramdown.gettalong.org/), but it can also support [redcarpet]() and others).  I've ported some `Rmd` documents over and didn't really notice any issues, but in general RMarkdown uses a pretty basic Markdown standard compared to others.

**Documents must be plain text but still extensible** 

# Creating a Jekyll-based Website

First, create a repository on GitHub to house your Jekyll website.  If you're going to use GitHub pages then you will create a repository called `USERNAME.github.io`, where `USERNAME` is your own username.  Make sure to include a `README` and `License` (I like the MIT License) files.  You can also include a Jekyll `gitignore` file.  

Once you've done this, clone the empty repository to wherever you'll be editing the documents locally (on your computer), navigate into the folder and generate a new Jekyll site (make sure you follow the setup instructions [here](https://jekyllrb.com/docs/installation/) first!)

```bash
cd GitHub
git clone https://github.com/SimonGoring/simongoring.github.io
cd simongoring.github.io
jekyll new . --force
```

Using the `--force` flag stops you from getting a conflict error (the directory `exists and is not empty.`) when you've been a responsible citizen and started your directory with a README and License file.

Even if you only get this far you can build & serve the website for the first time, and then `commit` and `push` to see your glorious template website live online:

```bash
$ jekyll build

> Configuration file: C:/Users/XXXXXXXX/Documents/GitHub/USERNAME.github.io/_config.yml
>            Source: C:/Users/XXXXXXXX/Documents/GitHub/USERNAME.github.io
>       Destination: C:/Users/XXXXXXXX/Documents/GitHub/USERNAME.github.io/_site
> Incremental build: disabled. Enable with --incremental
>      Generating...
>                    done in 2.082 seconds.
> Auto-regeneration: disabled. Use --watch to enable.

$ git status

> On branch master
> Your branch is up-to-date with 'origin/master'.
> 
> Untracked files:
>  (use "git add <file>..." to include in what will be committed)
>
>        .gitignore
>        _config.yml
>        _includes/
>        _layouts/
>        _posts/
>        _sass/
>        about.md
>        css/
>        feed.xml
>        index.html
>
> nothing added to commit but untracked files present (use "git add" to track)

$ git add --all
$ git commit -m "Generated my Jekyll-based website and built it for the first time!"
$ git pull
> Already up-to-date

$ git push
```

And now, if you navigate to `http://USERNAME.github.io` you'll see your pretty website!