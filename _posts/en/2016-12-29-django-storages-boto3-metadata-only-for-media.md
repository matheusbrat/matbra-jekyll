---
layout: post
title: Django Storages with Boto3 and additional Metadata only for Media 
lang: en
description: Django Storages with Boto3 and additional Metadata only for Media 
tags: django-storages boto3 python s3 aws
comments: true
--- 

I have a personal project which I'm using python with Django and django-storages to upload my static and media files to Amazon S3, because my media files have UUID and they're not editable on my system I wanted to have a long expiration time on it, so I could save some bandwidth but I didn't want this on the static files which are updated more regularly when I'm updating the system. 

Most of resources refer to `AWS_HEADERS` but it didn't work for me. It seems it is only for boto (not boto3) after looking into boto3 source code I discovered `AWS_S3_OBJECT_PARAMETERS` which works for boto3, but this is a system-wide setting, so I had to extend `S3Boto3Storage`.

So the code that solved my problem was:

{% highlight python %}
class MediaRootS3Boto3Storage(S3Boto3Storage):
    location = 'media'
    object_parameters = {
        'CacheControl': 'max-age=604800'
    }
{% endhighlight %}

If you're using boto (not boto3) and you want to have specific parameters only for Media classes you could use

 {% highlight python %}
class MediaRootS3Boto3Storage(S3BotoStorage):
    location = 'media'
    headers = {
        'CacheControl': 'max-age=604800'
    }
{% endhighlight %}

You also need to update your django-storages settings, pay attention to the class name, on boto 3 it is S3Boto**3**Storage on boto it doesn't has the 3 after Boto.

```
DEFAULT_FILE_STORAGE = 'package.module.MediaRootS3Boto3Storage'
``` 

Very simple tip, but it took a while to find out how it works

Matheus