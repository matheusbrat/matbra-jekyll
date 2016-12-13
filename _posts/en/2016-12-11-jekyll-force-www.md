---
layout: post
title: Force www on Jekyll website using Javascript
lang: en
description: Force www on Jekyll website using Javascript
tags: jekyll javascript
comments: true
---	

I wanted to force using my jekyll website to have the "www" prefix and because my Jekyll doesn't have a back-end I couldn't do it on the server, so I needed to use Javascript or Meta tags. A few people says Google Search engine handles meta refresh as 301/302 so it would be better to go with this approach from a SEO perspective. 

If you want to force www prefix on your website using javascript, you can use this snippet:

{% highlight javascript %}
<script>
if (window.location.hostname.indexOf("www") != 0) {
	window.location = window.location.protocol + "//www." + window.location.hostname + window.location.pathname;
}
</script>
{% endhighlight %}

I have created a `_include/force_www.html` file and I'm using `jekyll.environment` to load it, so I'm only loading it on production.

Matheus