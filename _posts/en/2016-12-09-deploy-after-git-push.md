---
layout: post
title: Deploy after push to your own git
lang: en
description: How to deploy your code automatically after a git push
tags: git githooks
comments: true
---	

I have explained [how to push your code to your own git server]({% post_url /en/2016-12-08-pushing-to-your-own-remote-git %}) and after this you may want to execute some especific functions, in my specific case I wanted my code to be builded and to release a new version, so I used `post-receive` hook from my repo.

Oh, it also handle multiple versions keeping the last 3 versions of the release. To do this it uses your DEPLOY_PATH and create a new folder sources on it, which will have your versions and a live folder which is a symlink to the version which is running. 

Vars:
* REPO_PATH = Path to your git folder
* DEPLOY_PATH = Path to your destiny folder
* DEPLOY_BRANCH = Branch you want to deploy

{% highlight bash %}
#!/bin/bash
REPO_PATH=/home/someuser/test.git
DEPLOY_PATH=/var/www/
DEPLOY_BRANCH="master"

echo "REPO_PATH=$REPO_PATH"
echo "DEPLOY_PATH=$DEPLOY_PATH"

while read oldrev newrev refname
do
    branch=$(git rev-parse --symbolic --abbrev-ref $refname)
    if [ $DEPLOY_BRANCH == "$branch" ]; then
        TIMESTAMP=$(date +%Y%m%d%H%M%S)
        VERSION_PATH=$DEPLOY_PATH/sources/$TIMESTAMP
        LIVE_PATH=$DEPLOY_PATH/live
        echo "TIMESTAMP=$TIMESTAMP"
        echo "VERSION_PATH=$VERSION_PATH"
        echo "LIVE_PATH=$LIVE_PATH"

        mkdir -p $VERSION_PATH
        mkdir -p $VERSION_PATH/sources

        git --work-tree=$VERSION_PATH --git-dir=$REPO_PATH checkout -f $DEPLOY_BRANCH
        # Remove git files
        rm -rf $VERSION_PATH/.git
        rm -rf $LIVE_PATH
        ln -s $VERSION_PATH $LIVE_PATH


        # Delete old folder keeping the 3 most recent ones, which aren't the current live one, / (root, security measure, different from your source folder)
        rm -rf $(ls -1dt $(find -L $DEPLOY_PATH/sources/ -maxdepth 1 -type d ! -samefile / ! -samefile $DEPLOY_PATH/sources/ ! -samefile $LIVE_PATH -print) | tail -n+3)
    fi
done
{% endhighlight %}


If you have any question, let me know.
Matheus

