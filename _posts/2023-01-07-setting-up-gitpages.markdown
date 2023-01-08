---
layout: post
title:  "Set up a site with Github Pages and Jekyll Tutorial"
date:   2023-01-07 21:17:21 -0600
categories: jekyll update
---
By leveraging Jekyll and Github Pages, you can quickly and easily set up a website for documentation or a simple blog. Jekyll is a static page generator capable of generating just about any data format including HTML. Write pages in markdown and Jekyll will build the HTML pages on your configuration, theme and layouts. By coupling this powerful tool with a Github repository and pages, you now have a powerful way to generic a website.

# Getting Started
Before you can use Jekyll to create a website hosted on Github Pages, you'll need to install some software first. You'll need to install Jekyll, Ruby, Git, a code editor and Bundle. 

It's recommended to using Bundler to install and run Jekyll. It makes life much easier by managing the Ruby gem dependencies and helps prevent environment issues. 

## Install Jekyll, Ruby and Bundler
Follow the [Jekyl installation guide](https://jekyllrb.com/docs/installation/) for your operating system. This includes instructions to intall Ruby, which is the base language used by Jekyll. Don't worry, you won't need to need the Ruby programming language.

Install [Bundler](https://bundler.io/)

## Install Git
In order to interface with GitHub, you'll need to [install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git). Git will allow you to make changes to your website and test it locally before committing the changes to your production website. Install it - you'll learn to use it in a few moments.

## Install a code editor
Any editor will work, but it's recommended to use [Visual Studio Code](https://code.visualstudio.com/download) which is available for Windows, Linux and Apple.


## Create the project
Using the command line of your operating system, execute these commands to create a new Jekyll project. This will be the location where all your website configuration and source files will be located. 

{% highlight ruby %}
mkdir my-website
cd my-website
jekyll new
{% endhighlight %}

If you installed everything correctly, you should be able to execute the command below, then use a web browser to see your newly created website at [http://localhost:4000](http://localhost:4000).

{% highlight ruby %}
bundle exec jekyll serve
{% endhighlight %}

# Editing the website
Go ahead and start up your code editor and point it at your base websitefolder. If you're using Visual Studio Code, use `File => Open Folder` to open it. 

## Structure
The initial structure for your website is fairly lean - and that's good! When you browsed to your website earlier, you should have been a single post named `Welcome to Jekyll!` as well as a link to a page named `About`. Those files plus more are outlined below. Take a moment to look at these files and folders.

| Name | Type | Description |
| ----------- | ----------- |
| _posts | Folder | This folder contains all the `weblog posts` for your website. |
| welcome-to-jekyll.markdown | File | A default post for reference. |
| _config.yml | File | The `site configuration` file. |
| .gitignore | File | Used by `git` to determine which files it should ignore. |
| 404.html | File | A default page if someone tries to browse to a page that doesn't exist. |
| about.markdown | File | A default About page. This file appears in the top-level menu. |
| Gemfile | File | Contains the `ruby gems` used by Jekyll. |
| Gemfile.lock | File | Contains the `ruby gem` versions and dependency map. |
| index.markdown | File | The main page that shows when you browse to the root of your website. |

## Base Theme: Minima
One thing you may have noticed (or not) is there are no HTML files in the folders, but there's certainly a layout and structure to your website. That's because Jekyll set up the project with a default theme named `Minima`. Look in the `_config.yml` file and you'll see a site setting named `theme: minima`. 

You can read more about [Minima on Github](https://github.com/jekyll/minima) and see all the layouts and include files it contains. This default theme is minimalistic, but it certainly saves you a ton of time since you don't have to worry about setting up your own page layout system.

There are hundreds of themes available, but that's an exercise for another day. Let's keep moving. 
- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)


## Markdown
Jekyll leverages an easy to learn markup language named [Markdown](https://www.markdownguide.org/). You can [use this cheatsheet](https://www.markdownguide.org/cheat-sheet/) to quickly come up to speed on how to make your pages quickly look great.

# Git
Now it's time to create a `git repository`, add your website files to it and configure it for `Github Pages`. You must have already installed `git` to proceed and a [Github account](https://github.com/). 

## Github
Login to [Github](https://github.com/) create a new repository. Name your repository whatever you'd like. Make sure you have selected this to be a **Public** repository -- only public repository can use Github Pages. Leave all the other defaults and press **Create Repository**.

It's time to create the local repository, add and commit the files and link the local repository to the remote repository on Github. In the `git remote add origin` command, make sure you replace your actual username & repository name. 

{% highlight javascript %}
echo "# MySite" >> README.md
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/USERNAME/REPOSITORY-NAME.git
git push -u origin main
{% endhighlight %}

After you've executed the `git add .` command, you can use the `git status` command to get a status of all the files to be commited in the next step.

{% highlight javascript %}
$ git add .
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   .gitignore
	new file:   404.html
	new file:   Gemfile
	new file:   Gemfile.lock
	new file:   README.md
	new file:   _config.yml
	new file:   _posts/2023-01-07-welcome-to-jekyll.markdown
	new file:   about.markdown
	new file:   index.markdown
{% endhighlight %}

The final command synchronizes your local changes with the remote Github repository. This is sample output of the final command.

{% highlight javascript %}
$ git push -u origin master
Enumerating objects: 21, done.
Counting objects: 100% (21/21), done.
Delta compression using up to 12 threads
Compressing objects: 100% (20/20), done.
Writing objects: 100% (21/21), 8.52 KiB | 8.52 MiB/s, done.
Total 21 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), done.
To https://github.com/USERNAME/REPOSITORY-NAME.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
{% endhighlight %}

## Github Pages
It's time to enable github Pages for your website repository. Open your browser, login to Github and browse to your repository. Go the `Settings` tab and find `Pages` on the left-hand column. 

The source will be `Deploy from a branch`. Select the `master` branch or whatever branch you decided to call yours. Use the `/root` folder ans press save. Give it 5 minutes or so and browse to your website. 

https://USERNAME.github.io/REPOSITORY-NAME



## Old stuff
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated. 

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
