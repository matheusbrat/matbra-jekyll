---
layout: post
title: Installing AvaloniaILSpy on Kali Linux
lang: en
description: Installing AvaloniaILSpy on Kali Linux
tags: linux kali pentest avalonia vb net .net microsoft bash hackthebox
comments: true
--- 

Oi,

Eu tenho estudado pentest (teste de intrusão) e eventualmente eu precisei decompilar um programa VBNET (.NET) e decidi testar o AvaloniaILSpy.

Se você precisar instalar ele no Kali linux 20, você pode instalar as dependencias da seguinte forma:

{% highlight bash %}
sudo apt-get update
sudo apt-get upgrade

wget https://packages.microsoft.com/config/ubuntu/19.10/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1

sudo apt-get install mono-devel

git clone https://github.com/icsharpcode/AvaloniaILSpy.git
cd AvaloniaILSpy/
git submodule update --init --recursive
{% endhighlight %}

E para compilar e rodar o mesmo:

{% highlight bash %}
$ bash build.sh
$ cd artifacts/linux-x64/
$ ./ILSpy
{% endhighlight %}

Espero que seja de ajuda,
Matheus