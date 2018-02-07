---
layout: post
title: Docker-compose com PHP-FPM, sendmail, nginx, mariadb serving jekyll e wordpress
lang: pt_BR
description: Exemplo de Docker-compose com multiplos containers servindo wordpress e jekyll
tags: linux aws docker docker-compose php-fpm nginx mariadb jekyll wordpress ec2 lightsail rds
comments: true
--- 

Como expliquei recentemente, eu tinha um blog rodando no wordpress e decidi migrar para o Jekyll, mas existia um detalhe, eu não queria perder os links que já apontavam para o meu blog em wordpress, para atingir isso, (Eu fiz o setup de um nginx o qual irá tentar encontrar arquivos estáticos do Jekyll e no caso de falha irá fazer um fallback para o wordpress)[http://www.matbra.com/2016/12/22/nginx-redirect-multiserver.html].

Eu estava rodando os mesmos em uma instancia ec2 com RDS e o preço estava um pouco alto, então decidi mover tudo para uma unica máquina e utilizar o docker para poder mudar facilmente meu website de servidor.

Para atingir isso eu criei um docker-compose com:
- PHP-FPM e sendmail para processar php e enviar e-mails com o sendmail
- Nginx para servir arquivos estáticos e caso eles não sejam encontrados, servir meu antigo blog em wordpress
- MariaDB como meu banco de dados para o wordpress

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
    hostname: test.com.br
  
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

Container PHP-FPM:

Eu estou usando um Dockerfile modificado, o qual vem do php:7.0-fpm e adiciona sendmail e a extensão do mysql. Existe um script de inicialização modificado para rodar os dois serviços na mesma máquina. (Eu sei, eu deveria criar um container especifico para o sendmail)

Neste container eu estou mapeando alguns arquivos php e configurações:
- ./wordpress.matbra.com para /var/www/wordpress.matbra.com meus arquivos do wordpress
- ./php7fpm/sendmail.mc para /usr/share/sendmail/cf/debian/sendmail.mc meu arquivo de configuração para o sendmail
- ./php7fpm/gmail-auth.db para /etc/mail/authinfo/gmail-auth.db meu arquivo de senha para o gmail (Configuring gmail as relay to sendmail)[https://linuxconfig.org/configuring-gmail-as-sendmail-email-relay]

Eu também estou mapeando as portas 9000 para 9000, então o nginx irá se comunicar com o php-fpm nestas portas e criando um link para o mariadb e criando um hostname

Container NGINX:

Eu estou usando um nginx alpine com alguns mapeamentos:
- ./nginx/nginx.conf para /etc/nginx/nginx.conf meu arquivo de configuração para o nginx
- ./nginx/app.vhost para /etc/nginx/conf.d/default.conf meu arquivo de configuração para o site com Jekyll fazendo fallback para o wordpress
- ./logs/nginx para /var/log/nginx o diretório de logs
- ./wordpress.matbra.com/ para /var/www/wordpress.matbra.com o local onde o nginx consegue encontrar meus arquivos do wordpress
- ./jekyll.matbra.com/ para /var/www/jekyll.matbra.com o local onde o nginx consegue encontrar meus arquivos do jekyll

Eu também mapeio as portas 80 para 80 e 433 para 433, crio um link para o php-fpm para que o nginx se comunique com o container do php-fpm

Container MARIADB:

Sem mistérios aqui, uma imagem do mariadb com mapa para os dados e algumas variaveis de ambiente

Já que eu não adicione meus arquivos do website, eu criei um comando `init.sh` para remover o diretório de dados e clonar os mesmos do git. Também tem um comando chamado `update-config.sh` para atualizar as configurações do wordpress (wp-config.php) com os valores corretos.

Com isso eu consigo facilmente criar meu "sistema" em uma nova máquina.

https://github.com/x-warrior/blog-docker

Eu espero que seja útil.
Matheus