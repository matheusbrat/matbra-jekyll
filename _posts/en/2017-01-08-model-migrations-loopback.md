---
layout: post
title: Loopback model migration using postgresql database
lang: en
description: Explanation about how to keep your model syncronized with the database using Loopback
tags: javascript nodejs loopback database
comments: true
--- 

I have been playing with [Loopback](https://strongloop.com/node-js/loopback-framework/), initially I was just declaring models and use in memory, but now I got to a point where I need to have a persistent database. 

I couldn't find how to keep my database synced with my models easily. I'm not sure if I'm not that familiar with Loopback yet, or if their documentation is not clear enough. 

To create a script to sync your models with your database you can create a file under bin/ called `autoupdate.js` and add the following:

```
var path = require('path');

var app = require(path.resolve(__dirname, '../server/server'));
var ds = app.datasources.db;
ds.autoupdate(function(err) {
  if (err) throw err;
  ds.disconnect();
});
```

The code is pretty simple, it will fetch the app from server.js, grab the datasource and run the `autoupate` command. You could use `automigrate`, but this one will clean the database every time, so pay attention on this.

I think this will work for most of datasources, but if it doesn't work for yours, drop me a line. I can try to help :D


Matheus