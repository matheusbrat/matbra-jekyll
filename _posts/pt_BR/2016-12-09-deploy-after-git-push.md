---
layout: post
title: Deploy depois de dar push com o seu git
lang: pt_BR
description: Como fazer deploy do seu repositório automaticamente depois de um push para o seu git
tags: git githooks
comments: true
---	

Eu expliquei [como enviar seu código para o seu próprio servidor]({% post_url /pt_BR/2016-12-08-pushing-to-your-own-remote-git %}) e depois disso você talvez queira executar algumas ações específicas, no meu caso eu gostaria que meu blog fosse atualizado quando eu enviasse uma nova versão para o git então eu usei um `post-receive hook.

Este script vai manter as 3 últimas versões do seu código, então se algo der errado, você pode fazer rollback mudando o link. Para fazer isso o script utiliza a variável DEPLOY_PATH e cria uma nova pasta `sources` nela, a qual vai ter as versões do seu site. A versão ativa é basicamente um link simbolico (symlink) da pasta live para a pasta `sources`

Variáveis:
* REPO_PATH = Caminho para o seu repositório local
* DEPLOY_PATH = Caminho para a pasta de release
* DEPLOY_BRANCH = Branch que você quer lançar

{% highlight bash %}
#!/bin/bash
REPO_PATH=/home/someuser/test.git
DEPLOY_PATH=/var/www/
DEPLOY_BRANCH="master"

echo "REPO_PATH=$REPO_PATH"
echo "DEPLOY_PATH=$DEPLOY_PATH"

while read oldrev newrev refname
do
    branch=$(git rev-parse --symbolic --abbrev-ref $refname)
    if [ $DEPLOY_BRANCH == "$branch" ]; then
        TIMESTAMP=$(date +%Y%m%d%H%M%S)
        VERSION_PATH=$DEPLOY_PATH/sources/$TIMESTAMP
        LIVE_PATH=$DEPLOY_PATH/live
        echo "TIMESTAMP=$TIMESTAMP"
        echo "VERSION_PATH=$VERSION_PATH"
        echo "LIVE_PATH=$LIVE_PATH"

        mkdir -p $VERSION_PATH
        mkdir -p $VERSION_PATH/sources

        git clone $REPO_PATH $VERSION_PATH
        git checkout $DEPLOY_BRANCH
        # Remove git files
        rm -rf $VERSION_PATH/.git
        rm -rf $LIVE_PATH
        ln -s $VERSION_PATH $LIVE_PATH


        # Delete old folder keeping the 3 most recent ones, which aren't the current live one, / (root, security measure, different from your source folder)
        rm -rf $(ls -1dt $(find -L $DEPLOY_PATH/sources/ -maxdepth 1 -type d ! -samefile / ! -samefile $DEPLOY_PATH/sources/ ! -samefile $LIVE_PATH -print) | tail -n+3)
    fi
done
{% endhighlight %}


Se você ainda tiver alguma questão me avise,
Matheus

