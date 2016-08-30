---
layout: post
title: git添加多个remote同步push
date: 2016-8-30
categories: blog
tags: [学习笔记,git]
description: 
---

有时候会需要在不同的git服务器存放同一个工程，比如，我本来在oschina上有一个代码仓库，现在需要在github上也有一份，而且每次能push的时候push到两个地方

比如，一开始，我的origin是指向oschina的仓库的

```shell
git remote -v
输出：
origin 	git@git.oschina.net:mrxiaobin/StudyDemo.git (fetch)
origin 	git@git.oschina.net:mrxiaobin/StudyDemo.git (push)
```

然后这样配置一下

```shell
git remote set-url --add --push origin git@git.oschina.net:mrxiaobin/StudyDemo.git
git remote set-url --add --push origin git@github.com:mrxiaobin/StudyDemo.git
```

配置完成之后再查看

```shell
git remote -v
输出：
origin 	git@git.oschina.net:mrxiaobin/StudyDemo.git (fetch)
origin 	git@git.oschina.net:mrxiaobin/StudyDemo.git (push)
origin 	git@github.com:mrxiaobin/StudyDemo.git (push)
```

然后用git push origin master命令可同时推送到oschina和github的仓库
