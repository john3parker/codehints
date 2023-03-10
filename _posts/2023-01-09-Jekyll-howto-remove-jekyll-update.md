---
layout: post
title:  "How to remove jekyll/update from your Jekyll website URL"
date:   2023-01-09 07:39:21 -0600
categories: jekyll config
---

Using Jekyll to maintain your website documentation or blog is super easy. It's very configurable making it powerful yet sometimes frustrating. If you're new to Jekyll, you'll may notice the initial post has `jekyll/update` in the URL. I've seen a lot of questions asking how to remove it. It's super easy so let's get to it.

## Permalinks
Jekyll has a default configuration for your permalinks. In the default install, you don't find it in your `_config.yml` file. As you can see from the config below, it has `:categories` followed by a date pattern `:year/:month/:day/` and a filename pattern `:title:output_ext`. 

It's the `:categories` setting which is createing the `jekyll/update` in your post URL. 

{% highlight yaml %}
permalink: /:categories/:year/:month/:day/:title:output_ext
{% endhighlight %}

Edit your post and look at the front matter settings at the top. You may have already noticed the `categories` setting.

{% highlight yaml %}
categories: jekyll update
{% endhighlight %}

Change those 2 categories to something else and presto, you've removed `jekyll/update` from your URL. 

## Change the Permalink
Alternatively, if you don't want your page categories to ever show up in your URL you can change the permalink setting in the `_config.yml`. 


{% highlight yaml %}
permalink: /:year/:month/:day/:title:output_ext
{% endhighlight %}

For more information about [Jekyll and Permlinks](https://jekyllrb.com/docs/permalinks/) refer to the documentation. 