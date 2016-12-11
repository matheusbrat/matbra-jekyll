---
layout: post
title: Instalar Nginx, PHP no Amazon Linux
lang: pt_BR
description: Instalando nginx e php com suporte ao MySQL no Amazon Linux
tags: amazon heroku wordpres s3 rds composer
comments: true
---

Estou migrando meu blog e alguns outros serviços para a infraestrutura da Amazon. Eu precisava de uma instância EC2 com suporte a PHP e a conectar no MySQL.

### Passos:

1. `yum update`
2. `yum install nginx`
3. `yum install php70 php70-fpm php70-mysqlnd`
4. Edite /etc/nginx/conf.d/virtual.conf

{% highlight nginx %}
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
}
{% endhighlight %}

5. Edite as seguintes propriedades do: /etc/php-fpm-7.0.d/www.conf
{% highlight nginx %}
user = nginx
group = nginx

listen = /var/run/php-fpm/php-fpm.sock

listen.owner = nginx
listen.group = nginx
listen.mode = 0660
{% endhighlight %}

6. Crie um arquivo php em /var/www/
{% highlight php %}
<?php
phpinfo();
?>
{% endhighlight %}

7. Acesse http://SERVER_IP:3000 

Você vai precisar da porta 3000 aberta nos security group da sua instância ec2.

Se você deseja iniciar os serviços automaticamente:
```
sudo chkconfig nginx on
sudo chkconfig php-fpm on
```

Se você quer reiniciar os seriviços:
```
sudo service nginx restart
sudo service php-fpm restart
```

Matheus