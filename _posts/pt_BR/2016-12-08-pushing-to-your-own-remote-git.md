---
layout: post
title: Enviando seus códigos para o seu próprio git
lang: pt_BR
description:  Como enviar seus códigos para o seu próprio git
tags: git
comments: true
---	

Eu estou mudando a minha infraestrutura como vocês devem ter percebido, neste momento eu gostaria de enviar meus códigos para o meu próprio servidor para poder fazer deploy com um simples `git push my_server branch`.

Então se você quer o mesmo tipo de comportamento você precisa conectar a sua máquina que vai hostear seu git com ssh e executar:

1. `$ mkdir test.git`
2. `$ cd git`
3. `$ git --bare init` 

Você vai precisar saber o caminho completo para a pasta recém criada. Para verificar o diretório que você se encontra utilize `pwd`. Voltando a sua máquina local, adicione o seu servidor como uma origin no seu git. 

```
git remote add my_server ssh://user@ip:/replace/with/pwd/test.git
```

Após isso você pode enviar os códigos do seu repositório utilizando `git push my_server branch`.

Matheus
