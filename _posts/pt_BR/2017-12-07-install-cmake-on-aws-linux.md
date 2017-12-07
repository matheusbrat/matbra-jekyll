---
layout: post
title: Instalando Cmake 3 no AWS Linux
lang: pt_BR
description: Passos executados para instalar Cmake 3 no Aws Linux
tags: linux aws cmake g++
comments: true
--- 

Se você está tentando compilar algo utilizando o CMake e tem o erro: "CMake 3.1 or higher is required.  You are running version 2.8.12.2"

Você pode manualmente instalar a versão mais recente do CMake, para fazer isso. Eu removi o CMake existente.

```
# yum remove cmake
```

Teste se ele realmente foi removido

```
$ cmake 
```

```
-bash: /usr/bin/cmake: No such file or directory
```

Instale G++

```
# yum install gcc-c++
```

Baixe a ultima versão do: [Cmake Download](https://cmake.org/download/)

```
$ wget https://cmake.org/files/v3.10/cmake-3.10.0.tar.gz
```

Extraia:
```
$ tar -xvzf cmake-3.10.0.tar.gz
```

Entre na pasta
``` 
$ cd cmake-3.10.0
```

Instale com
```
# ./bootstrap
# make
# make install
```

Agora você deve ter o cmake instalado /usr/local/bin/cmake

Abraços,
Matheus