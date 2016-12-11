---
layout: post
title: Pushing to your own remote git
lang: en
description: How to send your code to your own git repository hosted on your own machine
tags: git
comments: true
---	

I'm creating a new server as you can notice and I would like to push directly to my git (hosted on my own server), so I could release a new version with a simple `git push myserver branch`. 

If you want to achieve this as well you can connect to your remote ssh and execute

1. `$ mkdir test.git`
2. `$ cd git`
3. `$ git --bare init` 

You will need to know the full path of your git folder to add to as a remote on your local, to check the full path run `pwd`. Back to your local machine add your remote server.

```
git remote add my_server ssh://user@ip:/replace/with/pwd/test.git
```

After this you can use `git push my_server branch` to push to it.

Matheus

