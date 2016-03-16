---
layout: post
title: Pushing the same view controller instance more than once is not supported
date: 2016-3-14
categories: blog
tags: [学习笔记,iOS]
description: 
---

## 问题：在调用pushViewController的时候程序崩溃

应用有这样一个功能，点击推送通知可以跳转到对应的一个消息界面，
但这个界面是从另一个Controller Push进来的，所以，问题来了，当我已经在APP中手动打开了这个消息界面时，我再去点击通知栏的消息，这时会再次调用这个push方法，于是程序就会崩溃，
报错：
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: 'Pushing the same view controller instance more than once is not supported 。。。。'

所以在push之前先做个判断：

```ObjectiveC
    if(![self.navigationController.topViewController isKindOfClass:[_pageController class]]) {
        [self.navigationController pushViewController:_pageController animated:YES];
    }
```

---


