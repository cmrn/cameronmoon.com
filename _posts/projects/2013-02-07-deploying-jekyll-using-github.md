---
layout: post
title: Deploying a Lightweight Jekyll Site using GitHub
category: projects
excerpt: Jekyll can generate a website from the source files, but I need to get that generated site onto a proper webserver somewhere so everyone can see it. Ideally, I also want to use some kind of version control on the content of the site as well, to make sure I don't lose any data. Let's use Git to solve both these problems!
thumbnail: jekyll-github.png
---
Recently, I've been taking on some projects where I'm either creating new things, or using things in ways that I couldn't find documented before. These projects are the kinds of things I find useful on blogs, where someone has done the same kind of thing that I am trying to do, and documents it. However, I don't like all the other aspects of blogs - the inane personal ramblings, bloated blogging software, etc.

So I went out to find something that was like a blog, but without all the clutter. Something that I can post content to easily, but I don't need to worry about plugins, updates, comments, templates, etc. I fired up my trusty search engine, and came across the world of "Static Site Generators". These were what I was looking for - no serverside code (no PHP vulnerabilities!), super simple concepts (take content from folder A, layout from folder B, mix, and serve), and (mostly) roll-your-own design.

After looking over the different static site generators, I decided on [Jekyll](https://github.com/mojombo/jekyll). It seems to be the most mature and widely used, plus GitHub Pages uses it (so it must be good, right?). The Jekyll GitHub page explains itself best:

> Jekyll is a simple, blog aware, static site generator. It takes a template directory (representing the raw form of a website), runs it through Textile or Markdown and Liquid converters, and spits out a complete, static website suitable for serving with Apache or your favorite web server.

This sounds like the tool I was looking for! I can write my content in Markdown, and Jekyll will make it look nice and put them on my website. But, I still need to tell Jekyll *how* to make things look nice. Well, [Bootstrap](http://twitter.github.com/bootstrap/) (Formerly "Twitter Bootstrap") seems to be the popular choice at the moment, so lets use that.

Bootstrap is a front-end web framework that lets you build nice looking websites. It includes a bunch of neat stuff like a grid system, responsive design functionality, built in components (like dropdowns and thumbnails), and lots more. If you want some tips on making a good looking website using Bootstrap, check out [this excellent guide](http://24ways.org/2012/how-to-make-your-site-look-half-decent/).

Anyway, back to Jekyll. Jekyll can generate a website from the source files, but I need to get that generated site onto a proper webserver somewhere so everyone can see it. Ideally, I also want to use some kind of version control on the content of the site as well, to make sure I don't lose any data. Let's use Git to solve both these problems!

## The Setup

For this to work, you will need: A familiarity with Git, a GitHub account, a Jekyll site, and Git and Jekyll installed on a remote server. I'm also assuming you're running Linux - you can still follow the steps if you aren't, just dont expect the commands to work.

First off, we've got our Jekyll source files that we want to put into version control, so let's create a new repository to hold them.

	cd /path/to/my/site
	git init

Now, we don't want the `_site` directory in the Git repository, since that's just the compiled version of our content. So let's put that directory in the `.gitignore` file, then commit the rest of the files to the repository.

	echo "_site/" >> .gitignore
	git add .
	git commit

There are [a few options](https://github.com/mojombo/jekyll/wiki/Deployment) for deploying the site to your webserver, but I'm only going to cover the one I use. This method pushes the repository containing the site to GitHub, then have it tell our webserver that it should grab a new version of the site. Of course, this means sharing the source of your site with the world, so you have to be okay with that - [lots of people are](https://github.com/mojombo/jekyll/wiki/sites).

Since we're going to push to GitHub, we first need to [create a repo](https://help.github.com/articles/create-a-repo) on GitHub to push to. Once that's done, we can add it as a remote to the local repository that was set up earlier, then push to the remote repo.

	git remote add origin https://github.com/USERNAME/REPO-NAME.git
	git push

Next, we need to tell the webserver how to get the latest copy of the source from GitHub, run it through Jekyll, and put the resulting site in the correct place. To do this, let's create a little PHP script on our webserver that runs the appropriate commands.

	<?php
	`(cd source-files/ ; git pull)`;
	`jekyll`;
	`mv _site/* /path/to/web/directory`;
	?>

Put this in a PHP file in a directory on the webserver that is accessable from the web. Also put in a copy of the local repository (including the .git directory) in a subdirectory next to this script (in this case called `source-files`). So, the directory structure should look something like this:

	public_html/
	| - source/
	|   | - source-files/        <-- Copy of local repo
	|   | - build.php            <-- Script from above
	| - *other website files*

Now, when we go to `http://example.com/build.php`, the site should pull down the latest changes from GitHub, get Jekyll to compile them, and move them to where the site is served from. Of course, you'll probably want to call the script something other than `build.php`, to discourage people trying to make your webserver build the site unneccesarily.

All that's left is to make sure that the PHP script gets run whenever an update is pushed to GitHub. To do this, we can use something on GitHub calls a "[WebHook](https://help.github.com/articles/post-receive-hooks)", which will call a URL whenever a commit is made to the repository. To set up a WebHook, just go to the settings page of the repo, click on "Service Hooks", click "WebHook URLs", and enter the URL of the build script.

That's it! Now when we commit a change to our local copy of the Jekyll site and push it to GitHub, GitHub tells the webserver to grab a new copy of the site and build it.

## The Workflow
Now that everything is set up, how do we use it? Simple:

1. Make changes locally
2. Test changes locally (`jekyll --auto --server`)
3. Commit and push (`git add .`, `git commit`, and `git push`)

This process is good for a number of reasons. Firstly, it lets us make our changes and test them locally, before pushing them to a live webserver. Secondly, it makes us to use version control, which is a good thing. Finally, it publishes the source of our website on GitHub, so anyone who is interested in how the site was made can view the source and learn from it.

If you want to see how this all looks, you can find the source files for this site [on GitHub](https://github.com/cmrn).