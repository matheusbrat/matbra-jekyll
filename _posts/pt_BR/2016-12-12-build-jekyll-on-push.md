---
layout: post
title: Compilando seu site em Jekyll depois do git push
lang: pt_BR
description: Compilando seu site em Jekyll depois do git push
tags: jekyll git githook
comments: true
---	

Se você quer compilar (buildar) o seu blog no seu próprio servidor depois de um git push, você pode usar os git hooks. Para fazer isso, você pode simplesmente extender a minha dica [Deploy depois de push]({% post_url /pt_BR/2016-12-09-deploy-after-git-push %}) para também compilar o seu Jekyll como ambiente de produção. Para fazer isso basta adicionar o código abaixo depois do `rm -rf`

{% highlight sh %}
	cd $LIVE_PATH
	bundle install
	JEKYLL_ENV=production jekyll build
{% endhighlight %}

Matheus