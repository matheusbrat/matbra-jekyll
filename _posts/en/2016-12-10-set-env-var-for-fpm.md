---
layout: post
title: Set env var to PHP-FPM
lang: en
description: How to set arbitrary environment vars when running php-fpm
tags: php fpm
comments: true
---	

After [installing nginx and php]({% post_url /en/2016-12-07-install-nginx-php-on-amazon-linux %}), I wanted to use environment vars inside PHP 7 so I don't need to save configuration to my repo. 

Usually when using environment vars the ideal is to set it without having it saved in a file but on this case it was easier to. 

If you want to add environment variables to your PHP-FPM you can edit `/etc/php-fpm.d/www.conf` (I'm doing it on Amazon Linux and PHP 7.0)

There is a flag `clear_env = no` where you're able to set if php-fpm will receive a clean environment or not. I decided to leave it as the default value and but setting my vars as

```
env[WP_SECURE_AUTH_KEY] = "some-value"
env[WP_NONCE_KEY] = "nonce-key"
```

After this I restarted my nginx and php-fpm.

```
sudo service nginx restart
sudo service php-fpm restart
```

Matheus