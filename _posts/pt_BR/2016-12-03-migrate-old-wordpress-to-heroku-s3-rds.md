---
layout: post
title: Migrando um antigo Wordpress para o Heroku, Amazon RDS e S3
lang: pt_BR
description: Visão geral de como eu migrei o blog matbra.com para o Heroku usando Amazon RDS e S3.
tags: amazon heroku wordpres s3 rds composer
comments: true
---

Depois de alguns bons longos anos com o blog parado, decidi voltar a escrever e mudar o mesmo para o Heroku. Decidi utilizar o Heroku como servidor, o serviço de banco de dados da Amazon, o RDS e o S3 como servidor de arquivos.

### Passos:

1. Desabilite todas as extensões do Wordpress
2. Faça um backup completo (Wordpress, banco de dados, uploads, etc)
3. Sério faça o backup!
4. Crie um repositório git
5. Adicione o código do Wordpress ao seu git
    1. Se você quer atualizar seu Wordpress, adicione a [ultima versão do Wordpress](https://wordpress.org/download/)
        1. **Não adicione as suas configurações privadas!**
        2. Não adicione o conteúdo da pasta uploads
        3. Adicione seus plugins
        4. Adicione seu tema
    2. Se você quer manter a sua versão, adicione o código do seu blog atual
        1. **Não adicione as suas configurações privadas!**
        2. Não adicione a pasta uploads ao seu repositório
6. Atenção com os arquivos que contém dados privados!!
7. Altere seu wp-config.php
    1. Todas as configurações **privadas** devem usar getenv, essa função é responsável por pegar as informações das variaveis de ambiente no PHP

    {% highlight php %}
    <?php
    define('AUTH_KEY',         getenv('WP_AUTH_KEY'));
    define('SECURE_AUTH_KEY',  getenv('WP_SECURE_AUTH_KEY'));
    define('LOGGED_IN_KEY',    getenv('WP_LOGGED_IN_KEY'));
    define('NONCE_KEY',        getenv('WP_NONCE_KEY'));
    
    define('AUTH_SALT',        getenv('WP_AUTH_SALT'));
    define('SECURE_AUTH_SALT', getenv('WP_SECURE_AUTH_SALT'));
    define('LOGGED_IN_SALT',   getenv('WP_LOGGED_IN_SALT'));
    define('NONCE_SALT',       getenv('WP_NONCE_SALT'));
    
    define('S3_UPLOADS_BUCKET', getenv('AWS_S3_BUCKET'));
    define('S3_UPLOADS_KEY', getenv('AWS_S3_KEY'));
    define('S3_UPLOADS_SECRET', getenv('AWS_S3_SECRET'));
    define('S3_UPLOADS_REGION', getenv('AWS_S3_REGION')); 
    {% endhighlight %}

8. Crie um arquivo composer.json para definir pacotes e versão do PHP 
    1. Exemplo composer.json
    {% highlight json %}
    {
      "require" : {
          "php": ">=7.0.0"
      },
      "require-dev": {
      }
    }
    {% endhighlight %}

9. Execute `composer update` para gerar o composer.lock
10. Altere o seu .htaccess para fazer redirect dos uploads para o S3
    1. Atualize o BUCKET para o nome do seu bucket
    2. A regra da 5 linha é a responsável por fazer o redirect

    ```
    <IfModule mod_rewrite.c>
     RewriteEngine On
     RewriteBase /
     RewriteRule ^index\.php$ - [L]
     RewriteRule ^wp-content/uploads/(.*)$ https://s3-us-west-2.amazonaws.com/BUCKET/uploads/$1 [R=301,L]
     RewriteCond %{REQUEST_FILENAME} !-f
     RewriteCond %{REQUEST_FILENAME} !-d
     RewriteRule . /index.php [L]
     </IfModule>    
     ```

11. Faça um commit no seu git com os arquivos do seu wordpress
12. [Setup da Amazon](https://aws.amazon.com/getting-started/)
    1. Crie um banco de dados no RDS
    2. Importe o backup do seu banco de dados no RDS
    3. Envie seus arquivos para o S3
        1. Lembre-se de criar os arquivos com permissão publíca para que usuários não autenticados consigam acessar
13. [Setup do Heroku](https://devcenter.heroku.com/articles/getting-started-with-php#introduction)
    1. Adicione as variáveis de ambiente no Heroku com os nomes e valores que você utilizou no seu `wp-config.php`, lembre de usar as configurações para o RDS.
    2. [Atualize seu DNS no Heroku](https://devcenter.heroku.com/articles/custom-domains)
    3. Envie seu código para o Heroku usando o repositório git que você criou
14. Acesse o seu site

Se você está atualizando seu Wordpress, existem chances dos plugins que você usava não funcionarem na nova versão, então verifique se os plugins funcionam ;)

Se vocês tiverem dúvidas em algum passo específico eu posso criar um novo post com mais detalhes sobre ele, esse foi uma visão geral de como eu fiz. 


Matheus