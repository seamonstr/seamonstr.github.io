---
title: Setting up a new blog
---

What's going on!?
-----------------

Although I left a purely technical career over a decade ago, I have a habit of keeping up with techie items on a regular basis. New programming languages, hosting tech, cloudy stuff, source code management; also geeky things like 3d printing & other CNC manufacturing with its attendant gcode generators and managers, building robots and writing their firmware, etc. 

Last year I even did a MOOC on the mathematics behind a lot of AI (linear regression, logistic regression, neural nets, K-mean analysis, dimension reduction and loads of other stuff).

However, I have a bad habit of not writing any of this stuff down when I do it. When I come back to it after a month or two I've totally forgotten what I did and how it works - and I'm back nearly to square one.

Tut.

So, I'm going to _try_ to keep a blog up to date with the things I do so that I can just refer back to it myself later on and remember what I did and how I left it!

This mini-project - starting a blog site
----------------------------------------

So, I need a blog. Marvellous. I've had various blogs over the years that I could just resurrect, but I'm not too bothered - I doubt there's much there of interest!

Because most of the stuff that I do ends up on my GitHub repo, it would be great if there was a blogging tech that would pull content from there too. Tada - there is! It's called GitHub pages. 

The idea behind GitHub Pages is that you can publish content from within one of your repos. You write the content in [Markdown](https://www.markdownguide.org/), although you can specify some pages (like your site's `home` or `about` pages) in HTML.

Then, you go into that repo's GitHub settings and configure a "publishing source" in the `Pages` section of the settings. A publishing source is essentially the branch and directory of the root of your content. Also in the settings will be the URL where GitHub pages will publish your site; in my case this is [https://seamonstr.github.io/](https://seamonstr.github.io/).

Content, huh... what content's that, then?
------------------------------------------

The system that GitHub Pages uses to render your markdown into HTML is something called [Jekyll](https://jekyllrb.com/), a Ruby based scripty thing that reads all of your Markdown and HTML and generates a static HTML site from it.

While you could start your Jekyll site from scratch, it's far easier to just clone it from [this convenient repo](https://github.com/texts/texts.github.io), and then use that as the basis for your site.
1. Modify `_config.yml` in to add your own details; name, blog description and so on. 
1. If you're feeling creative you can modify the front page of your site by changing the `index.html` file. Something interesting to note in this file is that Jekyll uses `liquid` template text to generate the pagination of all of your posts (as well as all other scripty HTML rendering). It generates a number of `index.html` pages, one for each pageful of posts.
1. You can delete the sample posts from the `_posts` directory and then add your own. Note that the filename for a post has to be in the format used in the samples - a date and a post name.
1. The top of the markdown file is a section delimeted by two `---` lines. This is called "front matter" in Jekyll speak, and it's specially processed by Jekyll as YAML. There are [a number of variables](https://jekyllrb.com/docs/front-matter/) you can set on a per-page basis in this block. The obvious one you want is the page title, denoted by `title: `. 

As soon as you push your changes to the repo your site (at the URL above) will be updated with the new content. This is done by GitHub wiring in some deployment automation - if you go to the `Deployments` page for your repo you'll see every time it's pushed live by the `github-pages` user. 

Testing locally
---------------

Final point: it's really useful to be able to look at the site on your local machine before you push your post to your site.  As Jekyll generates static HTML, this is a doddle.

You'll need [Ruby fully installed locally](https://www.ruby-lang.org/en/documentation/installation/), and then you'll need to [install jekyll](https://jekyllrb.com/docs/installation/ubuntu/).

Then, you make the changes you want to your pages in your local repo (new blog, for example), and run `bundle exec jekyll serve` from the root of your publishing source. This will generate your HTML and start a little webserver to serve it.

A last gotcha is that Jekyll apparently deprecated its old method of doing pagination on the first page of the site, and the version of Jekyll run by GitHub Pages is still an old one.
This means that your front page will work fine on Github Pages, but it won't work locally.

You can fix this by adding the following as a new line in your `_config.yml` file:
```
gems: [jekyll-paginate]
```
...and your local file should work fine!

Let's see how we go
-------------------

Great, I now have a blog, I roughly understand how it works, and I'm currently filled with good intentions to keep it updated.

Let's see how long we last...

