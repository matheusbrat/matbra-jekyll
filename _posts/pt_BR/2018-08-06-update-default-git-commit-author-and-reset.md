---
layout: post
title: Atualizar o autor dos commits do git e resetar.
lang: pt_BR
description: Atualizar o autor dos commits do git e resetar.
tags: linux git github author reset
comments: true
--- 

Se você deseja atualizar o autor dos seus commits globalmente, utilize:

```
git config --global user.name "Your name"
git config --global user.email "email@example.net"
```

Após setar o mesmo globalmente, você pode setar o mesmo por projeto, então caso você deseje setar o autor dos commits do git por projeto use:

```
git config user.name "Your name"
git config user.email "email@example.net"
```

E um bonus: Se você deseja resetar os autores dos commits.

```
git commit --amend --reset-author
```

Se você desejar fazer para vários commits:

```
git rebase -i <COMMIT_HASH>
```

Até mais,
Matheus