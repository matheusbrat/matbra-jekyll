---
layout: post
title: Build Jekyll as production after push
lang: en
description: Build Jekyll as production after push
tags: jekyll git githook
comments: true
---	

If you want to build your Jekyll blog on your own server after a git push you can use git hooks. To do it, you can extend the [Deploy after git push]({% post_url /en/2016-12-09-deploy-after-git-push %}) and add this tree lines (after `rm -rf`), to install dependencies and to build it as production environment. 

{% highlight sh %}
	cd $LIVE_PATH
	bundle install
	JEKYLL_ENV=production jekyll build
{% endhighlight %}

Matheus