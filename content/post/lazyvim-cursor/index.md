---
title: "解决Lazyvim退出后cursor样式变成block的问题"
description: 
date: 2026-06-17T16:37:03+08:00
image: 
math: 
license: 
comments: true
draft: false
categories:
    - Tool-Usage
tags:
    - LazyVim
build:
    list: always    # Change to "never" to hide the page from the list
---

在使用Lazyvim的过程中，我遇到了一个问题：每次退出Lazyvim后，终端的光标样式会变成block的样式，
这使得拥有强迫症的我非常不爽，就STFW找了下解决方法。结果还真给我找到了，下面介绍解决方法：

解决方案引用自[恢复被NeoVim修改的终端光标](https://www.bilibili.com/video/BV1Xx4y1i7Lz/?spm_id_from=333.1391.0.0&vd_source=f5d256744421c08d819592d16ea163bf)下uid为300769512的用户评论

## TL;DR

在.bashrc或.bash_profile或其他shell配置文件中添加如下代码：

```bash
export PROMPT_COMMAND='printf "\e[6 q"' # set cursor to blinking bar
# \x30 change to blinking block
# \x31 change to blinking block also
# \x32 change to steady block
# \x33 change to blinking underline
# \x34 change to steady underline
# \x35 change to blinking bar
# \x36 change to steady bar
```

## 这是什么东西？

PROMPT_COMMAND是一个环境变量，它的值会在每次显示shell提示符之前被执行。通过设置PROMPT_COMMAND，我们可以在每次显示提示符时执行一些命令来改变终端的行为。
这个方法比视频内介绍的利用lazyvim的autocmd来改变光标样式更为直接和有效，因为它直接在shell层面上设置了光标样式，而不依赖于编辑器的行为（你不知道lazyvim还会在别的什么时候改变光标样式）
