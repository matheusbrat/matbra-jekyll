---
layout: post
title: Install Nginx, PHP on Amazon Linux
lang: en
description: Install nginx and php with MySQL support on Amazon Linux
tags: amazon php nginx
comments: true
---	

I'm migrating my blog and a few other stuff I have running to Amazon infrastructure. I needed an Amazon EC2 instance with PHP support and able to connect to a MySQL.

### Steps:

1. `yum update`
2. `yum install nginx`
3. `yum install php70 php70-fpm php70-mysqlnd`
4. Edit /etc/nginx/conf.d/virtual.conf

	```
	server {
	    listen       3000;
	
	    location / {
	        root   /var/www/;
	        index  index.php index.html index.htm;
	    }
	
	    location ~ \.php$ {
	        root /var/www/;
	        fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
	        fastcgi_index  index.php;
	        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
	        include        fastcgi_params;
	    }
	```

5. Edit the following properties of: /etc/php-fpm-7.0.d/www.conf
	```
	user = nginx
	group = nginx

	listen = /var/run/php-fpm/php-fpm.sock

	listen.owner = nginx
	listen.group = nginx
	listen.mode = 0660
	```

6. Create a php file on /var/www/
	```
	<?php
	phpinfo();
	?>
	```

7. Access http://SERVER_IP:3000 


You will need your security group for your ec2 instance to have port 3000 opened.

If you want to add them to auto start:
```
sudo chkconfig nginx on
sudo chkconfig php-fpm on
```

If you want to restart this services:
```
sudo service nginx restart
sudo service php-fpm restart
```

Matheus