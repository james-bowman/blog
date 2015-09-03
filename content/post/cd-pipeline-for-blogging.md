+++
date = "2015-07-06T19:13:19+01:00"
draft = false
title = "Continuous delivery pipeline for blogging with Hugo and Wercker"
description = "Setting up a continuous delivery pipeline for a blog site with Hugo, Wercker and GitHub Pages"
tags = [ "blog", "cd", "git", "github", "wercker", "hugo", "go", "golang" ]
categories = [
  "DevOps",
]
aliases = ["/post/blogging-cd-pipeline/"]

+++

This is the start of my new blog.  It is something I have been meaning to do for a long time but somehow never quite got around to.  Part of the problem for me was choosing a blogging platform and tool chain - I was overwhelmed by the number of different options available.  I know lots of people use platforms like [Wordpress] but I always liked the idea of static generators like [Jekyll].

I recently stumbled across [Hugo].  Hugo is a static generator like [Jekyll] where content (usually written in [markdown](https://en.wikipedia.org/wiki/Markdown)) is rendered offline as static HTML and then published to a hosting platform.  A benefit of this approach over using a platform like [Wordpress] is that the generated HTML can be hosted anywhere.  [Hugo] interested me in particular because it is written in [Go], which I am familiar with, and is reported to be extremely fast (important when waiting for content updates to render).  The steps I originally followed to setup my blog were inspired by [the tutorial on the Hugo site](http://gohugo.io/tutorials/automated-deployments/). 

## GitHub and GitHub Pages

I decided to use [GitHub Pages] to host the site since they offer a free personal site for each GitHub account and I intended to use [GitHub](http://github.com) for the source content anyway so using the same service for both made sense.

Although I am using GitHub to store both my source content and generated HTML, I wanted to keep them separate and so setup 2 new, empty repositories on GitHub:

1. `blog` - to store the source content.  The name of this repository is not important but for the remainder of this post, I will refer to it as 'blog'.
1. `<GitHub username>.github.io` - to host the generated static HTML.  The name of this repository must exactly match the pattern shown (this is where Github Pages serves the site from).

I use Git on a regular basis to version code I write and am used to using the command line client.  Whilst the steps I describe in the rest of this post assume the use of a command line client, a graphical client could be used to carry out the same operations if preferred.  Either command line or graphical client can be downloaded from the following URL:

https://git-scm.com/downloads

To author content locally, clone the source content Github repository created in the first step.

	git clone https://github.com/<Github username>/blog.git

Where `<Github username>` should be replaced by your Github username.  This will create a new directory called `blog` under the directory from where you executed the command.

So that Git does not try to version control the generated HTML as part of this repository (we want to store that in our second repository) I created a `.gitignore` file inside the blog directory containing `/public` (/public is the output directory to which our generated HTML will be written).

## Hugo

[Hugo] is an open source static site generator developed in [Go].  It is designed to be executed from the command line and it can be downloaded and installed locally.  It can be downloaded [here](https://github.com/spf13/hugo/releases).

Once installed, it can be executed on the command line to setup an initial workspace for all source site content.  The command should be executed from within the `blog` directory created when we cloned the Github repository above. 

	hugo new site .

Hugo uses interchangeable 'themes' to style generated websites.  There are a bunch of pre-developed themes to choose from on [GitHub](https://github.com/spf13/hugoThemes/).  To add one of these themes simply clone it into a `theme` folder under the content workspace as follows:

	mkdir themes
	cd themes
	git clone URL_TO_THEME 

Finally edit the `config.toml` file in the root of the content workspace and update the title and baseurl attributes to the title of your site and `http://<Github username>.github.io` respectively.
	
### Creating Content

Content in Hugo is stored under the `/content` folder in the workspace.  To add new content, use the `hugo new` command to create the file as follows: 

	mkdir content/post
	hugo new post/hello.md

This creates a new content [markdown](https://en.wikipedia.org/wiki/Markdown) file pre-populated with hugo meta-data in a header.

Edit the markdown file to add markdown text below the +++ and when finished, change the `draft` attribute in the header to `false`.  Now commit and push changes up to Github.

	git add .
	git commit
	git push origin master
	
### Generating the HTML

To generate the site from within the content workspace simply type the following command.  This will render the markdown content files as static HTML and place them in the `/public` output folder.

	hugo --theme=<theme folder name>

The specified theme folder name must exactly match the local folder name of the cloned theme e.g. if you are using hyde, the theme will have been cloned into the folder `themes/hyde` so the following command should be used:

	hugo --theme=hyde

### Publishing to GitHub Pages

Publishing to [GitHub Pages] is as simple as copying the /public directory (containing the generated website) and pushing the contents to the remote `<github username>.github.io` repository.  GitHub Pages will then serve the HTML from that repository at the following URL: `http://<GitHub username>.github.io/`.  As an alternative to copying the contents of the /public directory, one could simply create the /public directory as a symbolic link/shortcut to the local copy of the `<github username>.github.io` repository.

Here is the sequence of commands required to copy the /public directory and push to GitHub Pages:

	mkdir ../output
	cd ../output
	git init
	git remote add origin git@github.com:<Github username>/<Github username>.github.io.git

Then to push each set of changes:
	
	cd ../output
	cp -R ../blog/public/* . 
	git add .
	git commit -m "generated content"
	git push origin master

We have now [created content]({{< ref "#creating-content" >}}), [rendered it as HTML]({{< ref "#generating-the-html" >}}) and [published it]({{< ref "#publishing-to-github-pages" >}}) to a live website.  This represents all of the component parts of a nice publishing workflow - lets automate it!

## Automating the process with Wercker

[Wercker] is a hosted [Continuous Integration](http://www.thoughtworks.com/continuous-integration)/[Continuous Delivery](https://en.wikipedia.org/wiki/Continuous_delivery) service that has a growing community developing reusable build and deployment pipelines.  It has existing Hugo and GitHub Pages integrations built by the Werker community making it ideal for my needs.

Wercker is free to sign up for an account and you can sign up with your GitHub account.  Once registered, ensure that your account profile is connected to your GitHub account.  This can be achieved by clicking on your icon in the top right hand corner of the screen and then selecting `settings`.  Then select `Git connections` to connect your GitHub account.

Now we can setup our pipeline by creating a new application.  Select `Create` and select `Application`.  Work through the process of setting up a new application selecting GitHub as the Git provider, and choosing 'blog' as the repository.  Select yourself as the owner and configure access so that `wercker will checkout the code without using an SSH key`.  Finally uncheck the `Docker enabled` checkbox and click next step.  You can finally choose whether to make your pipeline public or private.  This is a personal choice and has no impact on how things work.  Werker will now start building your code.

Now we need to create a `wercker.yml` file in the root of the content workspace (/blog folder).  This file should contain the following:

``` yaml
box: wercker/default
build:
    steps:
        - arjen/hugo-build:
        version: 0.14
        theme: <theme folder name>
        config: config.toml
deploy:
    steps:
        - lukevivier/gh-pages@0.2.1:
        token: $GIT_TOKEN
        domain: <GitHub username>.github.io
        basedir: public
        repo: <GitHub username>/<GitHub username>.github.io
```

Be careful to replace any text within <> with your actual values e.g. `<GitHub username>` should be replaced with your specific GitHub username.  Once finished, commit and push the wercker.yml file to GitHub:

	git add wercker.yml
	git commit -m "Add wercker.yml"
	git push origin master

You may have noticed that our wercker.yml file contained a variable: `$GIT_TOKEN`.  This variable allows us to specify our GitHub access credentials for deploying to GitHub Pages outside of the pipeline and therefore keep them secret.  We will now set the value for this variable but first need to generate a GitHub access token.  Instructions for doing this can be found [here](https://help.github.com/articles/creating-an-access-token-for-command-line-use/).

To set the value for the `$GIT_TOKEN` variable, select the `settings` tab within wercker.  Now select `pipeline` and `Add new variable`.  Enter `$GIT_TOKEN` as the `Environment Variable` and paste your generated GitHub token into the `value` field.  Check the `protected` check box so that the token is kept hidden and not displayed.  Finally click the `save` button to complete the setup.  

Now everytime, you commit and push source content changes to the `blog` repository, Wercker will automatically render the updated content as HTML and publish it to GitHub Pages.


[Wordpress]: https://wordpress.com/
[Jekyll]: http://jekyllrb.com/
[Hugo]: http://gohugo.io
[Go]: https://golang.org/
[GitHub Pages]: https://pages.github.com/
[Wercker]: http://wercker.com/
