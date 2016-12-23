---
layout: post
title: Nginx redirect on failure 
lang: en
description: Redirect requests to a specific server with nginx
tags: jekyll nginx redirect
comments: true
--- 

As a few of you probably noticed, recently I have decided to [update my really old wordpress blog from PHP4~5 to a most recent one](http://www.matbra.com/2016/12/07/install-nginx-php-on-amazon-linux.html). Leaving a shared host and going to heroku, which later became Amazon EC2.

I had to decide if I would keep Wordpress, or change to a different technology as Jekyll? Or what? I have thought a lot about this and in the end I decided to use Jekyll to be honest, why? Because using something new will motivate me to study, play with something new and work more. 

Have decided to work with Jekyll, I had to think about my domain, because I didn't want to break my old wordpress blog, I want to keep it alive as a record and keep it for SEO points, but how to keep both living together on an awesome way? 

I thought the ideal would be to have something that tries to access the new website and if it is not found it should redirect to the old wordpress website. But how to redirect to the old blog only when a page is not found and complying with the http status code (ie: redirecting with 301).

After some documentation reading on nginx I found you can try to proxy to a server and if it fails redirect to a new one, it seems the ideal solution for now. 

I have a nginx configuration file with multiple servers, first I have a nginx wordpress configuration, this server just adds PHP-FPM to process PHP files basically with my own custom domain.

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

My second server is basically nginx serving static files for the new blog created with Jekyll:

{% highlight nginx %}
server {
    listen	5000;

    location / {
        root   /var/www/jekyll/live/_site/;
        index  index.html index.htm;
    }
}
{% endhighlight %}

So the most important server is the nginx server which will redirect when my jekyll server doesn't find the url being requested. This one is a little bit more, I'm creating a nginx proxy which intercept errors with `proxy_intercept_errors on;` and on error page redirect to my secondary server wordpress using `error_page 404 = @wordpress;` 

If it is redirected to the wordpress page it will rewrite the url to the wordpress server.

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

So my configuration file is a composition from all of this.

What do you guys think? Do you have any question?

Matheus