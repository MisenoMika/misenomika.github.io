---
title: "通过 --amend 选项使得你的commit-tree更简洁"
description: 
date: 2026-06-17T15:57:01+08:00
image: git-amend.png
math: 
license: 
comments: true
draft: false
tags:
    - Git
categories:
    - Tool-Usage
build:
    list: always    # Change to "never" to hide the page from the list
---
我们可能会遇到一种情况:刚刚commit一个改动,突然想加一些小改动(加一条注释, 修改一个变量名, 追加一个文件等)
如果这些小改动也要commit一次,长久下去不仅commit-tree变得杂乱无比,更重要的是后续如果遇到严重bug时, 许多无太大意义的commit记录会影响我们的代码追溯

那么我们Linus显然也考虑到了这一点,为我们提供了一条不会影响commit-tree的commit选项:

``` bash
$ git commit --amend
```

效果：

- 不会生成新的 commit
- 而是修改最后一个 commit
- 可以改 commit message，也可以不改

如果不需要修改commit message:

``` bash
$ git commit --amend --no-edit
```

如果已经git push到了远程仓库,则需:

``` bash
$ git push [-f|--force]
```

因为再次提交会改变哈希值
