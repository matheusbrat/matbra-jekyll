---
layout: post
title: Loopback migrando seus models com o postgresql
lang: pt_BR
description: Explicação de como manter seus modelos atualizados com o banco de dados utilizando Loopback
tags: javascript nodejs loopback database
comments: true
--- 

Eu estou brincando com o [Loopback](https://strongloop.com/node-js/loopback-framework/), um framework javascript. Inicialmente eu só estava criando modelos e utilizando o explorer para verificar os resultados do mesmo, porém agora eu atingi um ponto que preciso persistir os dados.

Eu não consegui encontrar como manter meu banco de dados e meus modelos sincronizados facilmente, não tenho certeza se eu que não estou tão familiar com o Loopback ainda, ou se a documentação esta realmente um pouco confusa. 

Então para criar um script que mantenha seus modelos sincronizados você pode criar um arquivo na pasta bin e nomea-lo `autoupdate.js` e adicionar o seguinte:

```
var path = require('path');

var app = require(path.resolve(__dirname, '../server/server'));
var ds = app.datasources.db;
ds.autoupdate(function(err) {
  if (err) throw err;
  ds.disconnect();
});
```

O código é bem simples, ele vai importar seu app do server.js, adicionar o datasource e rodar o comando `autoupdate`. Você poderia usar `automigrate` mas este limpa o seu database todas as vezes, então se você quer manter seus dados, essa não é a maneira apropriada. 

Eu acredito que isso vai funcionar para outros bancos de dados, como MySQL. Se não funcionar, me avise, eu posso tentar ajudar.

Matheus