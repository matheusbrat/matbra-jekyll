---
layout: post
title: Update default git commit author and reset for commit.
lang: en
description: Update default git commit author and reset for commit.
tags: linux git github author reset
comments: true
--- 

If you would like to set your global git author, use:

```
git config --global user.name "Your name"
git config --global user.email "email@example.net"
```

After having it set globally, you can to set your git author per project using:

```
git config user.name "Your name"
git config user.email "email@example.net"
```

And a bonus, If you need to reset the git commit author:

```
git commit --amend --reset-author
```

If you want to do it for multiple commits:

```
git rebase -i <COMMIT_HASH>
```

See you,
Matheus