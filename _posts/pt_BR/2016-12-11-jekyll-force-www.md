---
layout: post
title: Forçar www no endereço do seu Jekyll usando Javascript
lang: en
description: Forçar www no endereço do seu Jekyll usando Javascript
tags: jekyll javascript
comments: true
---	

Eu queria fazer meu site ter sempre a mesma url, com o prefixo www como meu Jekyll não possui um back-end eu não podia fazer no servidor, portanto eu precisava utilizar Javascript ou meta tags. Algumas pessoas comentam que o buscador do Google considera tags meta refresh como redirects (301/302) então essa solução seria a mais adequada.

Teimoso do jeito que sou, decidi utilizar Javascript. Se você desejar fazer com que o seu blog tenha esse mesmo comportamento, utilize o seguinte código:


{% highlight javascript %}
<script>
if (window.location.hostname.indexOf("www") != 0) {
	window.location = window.location.protocol + "//www." + window.location.hostname + window.location.pathname;
}
</script>
{% endhighlight %}

Eu criei um arquivo `_include/force_www.html` e estou usando a variável `jekyll.environment` para incluir essa paret do template e ter esse comportamento somente no ambiente de produção.

Matheus