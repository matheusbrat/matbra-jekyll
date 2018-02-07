---
layout: post
title: Docker-compose with PHP-FPM, sendmail, nginx, mariadb serving jekyll and wordpress
lang: en
description: Example of Docker-compose with multiple containers serving wordpress and jekyll
tags: linux aws docker docker-compose php-fpm nginx mariadb jekyll wordpress ec2 lightsail rds
comments: true
--- 

As I explained recently, I had a blog running Wordpress and decided to move to Jekyll but there was a catch, I didn't want to loose any link I had to my wordpress blog, to achieve this, (I setup an nginx which will try to find a static file from jekyll and if it is not found it will fallback to Wordpress)[http://www.matbra.com/2016/12/22/nginx-redirect-multiserver.html].

I was running my server on ec2 instance with RDS and it was becoming a little bit expensive, so I decided to move everything to one machine and dockerize my setup so I could easily switch my servers.

To achieve this, I have created a docker-compose with: 
- PHP-FPM and sendmail to process php and sendmail
- Nginx to serve jekyll static files and if they're not found serve my old wordpress blog
- MariaDB as my Database for Wordpress

```
version: '3'
services:
  fpm:
    # image: php:7.0-fpm-alpine
    build: php7fpm
    restart: always
    volumes:
      - ./wordpress.matbra.com/:/var/www/wordpress.matbra.com
      - ./php7fpm/sendmail.mc:/usr/share/sendmail/cf/debian/sendmail.mc
      - ./php7fpm/gmail-auth.db:/etc/mail/authinfo/gmail-auth.db
    ports:
      - "9000:9000"
    links:
      - mariadb 
    hostname: boarders.com.br
  
  nginx:
    image: nginx:1.10.1-alpine
    restart: always
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/app.vhost:/etc/nginx/conf.d/default.conf
      - ./logs/nginx:/var/log/nginx
      - ./wordpress.matbra.com/:/var/www/wordpress.matbra.com
      - ./jekyll.matbra.com/:/var/www/jekyll.matbra.com
    ports:
      - "80:80"
      - "443:443"
    links:
      - fpm

  mariadb:
    image: mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=yourpassword
      - MYSQL_DATABASE=
    volumes:
    -   ./data/db:/var/lib/mysql
```

PHP-FPM container:

I'm using a custom Dockerfile which comes from php:7.0-fpm and add sendmail support and mysql extension. There is a custom starter script which will run sendmail + php-fpm. (I know I should create a specific container for sendmail)

On this container I'm basically mapping some php files and config files:
- ./wordpress.matbra.com to /var/www/wordpress.matbra.com which are my wordpress files
- ./php7fpm/sendmail.mc to /usr/share/sendmail/cf/debian/sendmail.mc which is my configuration file for sendmail
- ./php7fpm/gmail-auth.db to /etc/mail/authinfo/gmail-auth.db which is the password for my gmail (Configuring gmail as relay to sendmail)[https://linuxconfig.org/configuring-gmail-as-sendmail-email-relay]

I'm also mapping the port 9000 to 9000, so I will communicate with PHP-FPM on this ports, creating a link to mariadb and naming my hostname.

NGINX container:

I'm using the regular nginx alpine with some maps:
- ./nginx/nginx.conf to /etc/nginx/nginx.conf which is my nginx configuration
- ./nginx/app.vhost to /etc/nginx/conf.d/default.conf which is my website configuration with Jekyll falling back to wordpress
- ./logs/nginx to /var/log/nginx which will be my log directory
- ./wordpress.matbra.com/ to /var/www/wordpress.matbra.com which is the place where nginx can find wordpress website
- ./jekyll.matbra.com/ to /var/www/jekyll.matbra.com which is the place where nginx can find jekyll website

I'm also mapping ports 80 to 80 and 443 to 443 and create a link to PHP-FPM so nginx can communicate with fpm container.

MARIADB container:

No mistery here, regular mariadb image, with a mapping for data and some environment variables.

Because I'm not adding my website files to the image, I have created a command `init.sh` to remove website directory and clone website from git. There is a command called `update-config.sh` to update wp-config.php file with the correct environment variables.

With this I can easily spin up a new machine with my website structure.

https://github.com/x-warrior/blog-docker

I hope this will be helpful for you.
Matheus