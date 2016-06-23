---
layout: post
title: OC中load和initialize方法的使用
date: 2016-5-23
categories: blog
tags: [学习笔记,iOS,Objective-C]
description: 
---


# load

- load在main函数之前调用，不管有没有用到
- 先调用父类的，再调用子类的
- 分类里边实现了load，则先调用原始类的load，后调用分类的load

# initialize

- initialize在main函数之后调用，只有第一次调用这个类的时候才会执行，并且只会执行一次
- initialize在调用的时候会自动调用父类的initialize方法，不需要写super...，如果没有实现initialize方法，也不会调用父类的initialize方法
- 如果分类中也实现了initialize，则会优先加载分类的initialize方法，不会调用原始类的initialize方法

In addition:
A class’s +load method is called after all of its superclasses’ +load methods.
A category +load method is called after the class’s own +load method.