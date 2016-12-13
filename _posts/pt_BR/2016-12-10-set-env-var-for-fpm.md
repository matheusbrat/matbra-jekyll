---
layout: post
title: Definindo variáveis de ambiente para o PHP-FPM
lang: pt_BR
description: Como definir variáveis de ambiente para o PHP-FPM
tags: php fpm
comments: true
---	

Após [instalar o nginx e o PHP]({% post_url /pt_BR/2016-12-07-install-nginx-php-on-amazon-linux %}), eu queria passar variáveis de ambiente para o PHP 7 para que eu não tivesse que salvar as configurações para o meu repositório.

Quando você utiliza variáveis de ambiente o ideal é defini-las sem deixa-las salvas, porém nesse caso eu salvei as mesmas.

Se você deseja adicionar variáveis de ambiente para o seu PHP-FPM, você pode editar `/etc/php-fpm.d/www.conf` (Estou fazendo isso no Amazon Linux e PHP 7.0)

Existe uma configuração no PHP-FPM, `clear_env = no` esta variável define se o PHP vai receber um ambiente limpo ou não. Eu deixei a mesma habilitada (o valor padrão é habilitado) mas adicionei as minhas variáveis de ambiente:

```
env[WP_SECURE_AUTH_KEY] = "valor1"
env[WP_NONCE_KEY] = "valor2"
```

Depois disso eu reiniciei o nginx e o PHP-FPM:


```
sudo service nginx restart
sudo service php-fpm restart
```

Matheus