---
layout: post
title: Install Cmake 3 on AWS Linux
lang: en
description: Steps executed to install CMake3 on AWS Linux
tags: linux aws cmake g++
comments: true
--- 

If you are trying to build something using CMake and is getting the error: "CMake 3.1 or higher is required.  You are running version 2.8.12.2"

You can manually install this CMake version, to do this, I removed the previous CMake.

```
# yum remove cmake
```

Tested if it was really removed

```
$ cmake 
```

```
-bash: /usr/bin/cmake: No such file or directory
```

Install G++

```
# yum install gcc-c++
```

Download latest version from: [Cmake Download](https://cmake.org/download/)

```
$ wget https://cmake.org/files/v3.10/cmake-3.10.0.tar.gz
```

Extract it:
```
$ tar -xvzf cmake-3.10.0.tar.gz
```

Enter on cmake folder
``` 
$ cd cmake-3.10.0
```

Install it with:
```
# ./bootstrap
# make
# make install
```

Now you should have cmake under /usr/local/bin/cmake

Best regards,
Matheus