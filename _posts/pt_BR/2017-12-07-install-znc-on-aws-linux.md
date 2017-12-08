---
layout: post
title: Instale ZNC IRC Bouncer on AWS Linux
lang: pt_BR
description: Passos executados para instalar o ZNC IRC Bouncer no AWS Linux
tags: linux aws cmake irc
comments: true
--- 


Se você quer instalar [ZNC IRC Bouncer](https://github.com/znc/znc) você vai precisar do CMake, porém a versão do CMake do AWS Linux é desatualizada. (Atualize seu cmake para 3.x)[http://www.matbra.com/2017/12/07/install-cmake-on-aws-linux.html]

Agora você precisa do git para clonar o código fonte do ZNC e também o openssl-devel para ter suporte ssl.

```
# yum install git openssl-devel
```

Clone o código fonte

```
$ git clone https://github.com/znc/znc.git
```

Entre na pasta
```
$ cd znc
```

Inicializa os submodulos
```
$ git submodule update --init --recursive
```

Instale
```
$ cmake . 
$ make
# make install (run this as root #)
```

Configure-o:
```
$ znc --makeconf
```

Atenciosamente,
Matheus