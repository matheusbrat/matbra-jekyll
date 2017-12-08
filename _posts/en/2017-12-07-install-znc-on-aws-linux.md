---
layout: post
title: Install ZNC IRC Bouncer on AWS Linux
lang: en
description: Steps executed to install ZNC IRC Bouncer on AWS Linux
tags: linux aws cmake irc
comments: true
--- 

If you want to install [ZNC IRC Bouncer](https://github.com/znc/znc) you will need CMake, but AWS Linux CMake is too old. (Update your cmake to 3.x)[http://www.matbra.com/2017/12/07/install-cmake-on-aws-linux.html]

Now you will need git to clone the ZNC source code and openssl-devel to have ssl support

```
# yum install git openssl-devel
```

Clone ZNC source code

```
$ git clone https://github.com/znc/znc.git
```

Enter on the source code folder
```
$ cd znc
```

Initialize submodules
```
$ git submodule update --init --recursive
```

Install it with:
```
$ cmake . 
$ make
# make install (run this as root #)
```

Configure it with:
```
$ znc --makeconf
```

Best regards,
Matheus