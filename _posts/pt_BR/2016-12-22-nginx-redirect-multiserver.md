---
layout: post
title: Redirecionar acessos para um servidor Wordpress secundário usando Nginx
lang: pt_BR
description: Redirecionar requests para um servidor especifico quando a página não for encontrada, utilizando nginx
tags: jekyll nginx redirect
comments: true
--- 

Alguns de vocês provavelmente acompanhou, mas recentemente eu decidi [atualizar meu antigo wordpress blog do PHP4~5 para um mais recente](http://www.matbra.com/pt_BR/2016/12/07/install-nginx-php-on-amazon-linux.html). Deixando meu host compartilhado e indo para o heroku e posteriormente para Amazon EC2.

Eu precisava decidir se eu manteria o Wordpress ou se eu mudaria para uma tecnologia diferente, como Jekyll? Ou o que? Eu pensei bastante sobre isso e no fim eu decidi utilizar Jekyll, por que? Para ser sincero, usando algo mais novo/recente me motiva a estudar e fuçar mais a fundo para conseguir as coisas rodando.

Depois que decidi que usaria o Jekyll, eu precisava pensar sobre meu dominio, eu queria manter meu blog antigo rodando como histórico mas também gostaria de fazer redirecionamento correto para não perder meus pontos de SEO por exemplo, então como manter 2 blogs funcionando de uma maneira inteligente sem quebrar os links antigos? 

Eu pensei que o ideal seria algo que tentasse acessar o novo site e caso não encontrasse, deveria redirecionar para o blog antigo rodando Wordpress, mas como atingir isso somente quando a página não fosse encontrada e de uma maneira boa para SEO (utilizando 301 para os redirects)? 

Depois de algum tempo brincando e lendo a documentação do nginx eu achei uma maneira de rodar um servidor com um proxy e caso o acesso falhe, interceptar o mesmo e redirecionar para outro servidor do nginx.

Então para fazer isso eu tenho um arquivo de configuração do nginx com multiplas configurações, a primeira delas possui a configuração do Wordpress, esse servidor é bem simples e somente trata acessos de páginas PHP com o PHP FPM basicamente usando um subdomínio.

{% highlight nginx %}
server {
   listen 80;
   server_name wordpress.matbra.com;

    location / {
        root   /var/www/wordpress/live;
        index  index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$uri$args;
    }

    location ~ \.php$ {
        root /var/www/wordpress/live;
        fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
{% endhighlight%}

<!--more-->

O meu servidor secundário é basicamente um servidor no nginx que serve arquivos estáticos que foram gerados com o Jekyll.

{% highlight nginx %}
server {
    listen	5000;

    location / {
        root   /var/www/jekyll/live/_site/;
        index  index.html index.htm;
    }
}
{% endhighlight %}

E o último e mais importante servidor é o que é responsável por tentar acessar o servidor Jekyll e caso a página não seja encontrada no mesmo, fazer o redirect para o servidor antigo no Wordpress. Esse é um pouco mias complexo, eu estou criando um proxy com capacidade de interceptar erros usando `proxy_intercept_errors on;` e redirecionando esses casos para um outro servidor com `error_page 404 = @wordpress;` e neste caso não é um proxy transparente, eu faço redirect utilizando response code 301. 

{% highlight nginx %}
server {
   listen 80;
   server_name www.matbra.com matbra.com;
   location / {
        proxy_intercept_errors on;
        error_page 404 = @wordpress;
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:5000;
   }

   location @wordpress {
	rewrite ^/(.*) http://wordpress.matbra.com/$1 permanent;
   }
}
{% endhighlight %}

Então a configuração do meu nginx é uma composição desses 3 servidores. 

O que vocês acham? Vocês tem alguma pergunta? 


Matheus