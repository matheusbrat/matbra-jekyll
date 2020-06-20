---
layout: post
title: Installing AvaloniaILSpy on Kali Linux
lang: en
description: Installing AvaloniaILSpy on Kali Linux
tags: linux kali pentest avalonia vb net .net microsoft bash
comments: true
--- 

Hello,

I have been studying pentest and eventually I had to decompile some VB NET (.NET) and decided to give a try on AvaloniaILSpy.

If you ever need to install it on Kali linux 20 you can install its dependencies with:

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


And later to build and run it:

{% highlight bash %}

$ bash build.sh
$ cd artifacts/linux-x64/
$ ./ILSpy
{% endhighlight %}

Hope this helps you,
Matheus

