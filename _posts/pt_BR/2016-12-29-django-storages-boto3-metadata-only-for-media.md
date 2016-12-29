---
layout: post
title: Django Storages com Boto3 e Metadados adicionais somente para as Medias
lang: pt_BR
description: Django Storages com Boto3 e Metadados adicionais somente para as Medias
tags: django-storages boto3 python s3 aws
comments: true
--- 

Eu tenho um projeto pessoal o qual estou usando Django com django-storages para enviar meus arquivos estáticos e medias para a Amazon S3, já que as minhas medias possuem UUID e não são editáveis eu gostaria de ter um tempo de expiração maior para elas, assim eu poderia salvar transferência porém eu não gostaria de ter essa cache maior para os arquivos estáticos que são atualizados com mais frequência.

Maioria dos conteúdos que encontrei na internet se referiam a `AWS_HEADERS` porém o mesmo não funcionou para mim. Parece que o mesmo só funciona para o boto (não para o boto3) e depois de olhar o código do boto3 eu descobri o `AWS_S3_OBJECT_PARAMETERS` o qual funciona para o boto3, mas esta é uma configuração global então eu precisei extender a classe `S3Boto3Storage`.

Então o código que resolveu meu problema foi:

{% highlight python %}
class MediaRootS3Boto3Storage(S3Boto3Storage):
    location = 'media'
    object_parameters = {
        'CacheControl': 'max-age=604800'
    }
{% endhighlight %}

Se você está usando o boto (não o boto3) e gostaria de ter meta dados especiais somente para a sua classe Media, você pode usar:

 {% highlight python %}
class MediaRootS3Boto3Storage(S3BotoStorage):
    location = 'media'
    headers = {
        'CacheControl': 'max-age=604800'
    }
{% endhighlight %}

Você também vai precisar atualizar a sua configuração do django-storages, preste atenção ao nome da classe, no boto 3 é S3Boto**3**Storage porém no boto, o mesmo não possui o 3 depois da palavra Boto

```
DEFAULT_FILE_STORAGE = 'package.module.MediaRootS3Boto3Storage'
``` 

Uma dica simples, porém pode salvar um pouco do seu tempo.

Matheus