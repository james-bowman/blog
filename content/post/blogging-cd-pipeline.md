+++
date = "2015-07-06T19:13:19+01:00"
draft = true
title = "blogging cd pipeline"
tags = [ "blog", "cd", "git", "github", "wercker", "hugo", "go", "golang" ]
categories = [
  "DevOps",
]

+++

This is the start of my new blog.  It is something I have been meaning to do for a long time but somehow never quite got around to.  Part of the problem for me was choosing a blogging tool chain and platform - I was overwhelmed by the number of different options available.  I know lots of people use platforms like [Wordpress] but I always liked the idea of static generators like [Jekyll].

<!--more-->

Whilst I was doing some [Go] development and reviewing available libraries I stumbled across [Hugo].  Hugo is a static generator like [Jekyll] where content (usually written in a format like [markdown](https://en.wikipedia.org/wiki/Markdown)) is rendered offline as static HTML and then published to a hosting platform.  A benefit of this approach over using a platform like [Wordpress] is that the generated HTML can be hosted anywhere.  Hugo interested me mainly because it is written in [Go], which I am familiar with, and is reported to be extremely fast (important when waiting for content updates to render).  The steps I originally followed to setup my blog were primarily inspired by the tutorial on the Hugo site. 

# GitHub and GitHub Pages

I decided to use [GitHub Pages](https://pages.github.com/) to host the site since they offer a free personal site for each GitHub account and I intended to use [GitHub](http://github.com) for the source content anyway so using the same service for both made sense.

Although I am using GitHub to store both my source content and generated HTML, I wanted to keep them separate and so set up 2 repositories on GitHub:

1. `Blog` - to store the source content.  The name of this repository does not matter.
1. `<GitHub username>.github.io` - to host the generated static HTML.  The name of this repository is important and must match the pattern shown  (this is where Github Pages serves the site from).

# Hugo

Next download and install Hugo.

Clone the source content Github repository created in the first step.

Inside the local copy of the repository run the following command to setup the Hugo structure:

	hugo new site blog

add a theme(s)

	git clone ...

add some content

	hugo new about.md

commit and push changes to GitHub

	git add
	git commit
	git push origin master
	
# Generating the HTML


# Publishing to GitHub Pages

		
# Automating the process with Wercker

Now I have all of the component parts


# What next?

Custom domains


[Wordpress]: https://wordpress.com/
[Jekyll]: http://jekyllrb.com/
[Hugo]: http://gohugo.io
[Go]: https://golang.org/
