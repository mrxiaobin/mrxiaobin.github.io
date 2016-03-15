---
layout: post
title: Swift中报错Method 'xxx' with Objective-C selector xxx conflicts with getter for xxx with the same Objective-C Selector
date: 2016-3-14
categories: blog
tags: [学习笔记,iOS]
description: 
---

在学习斯坦福课程 [Stanford University: Developing iOS 8 Apps with Swift](https://itunes.com/StanfordSwift)[（中文字幕版)](http://open.163.com/special/opencourse/ios8.html)
遇到的问题,部分代码如下：


```Swift
    func performOperation(operation: (Double, Double) -> Double) {
        if operandStack.count >= 2 {
            displayValue = operation(operandStack.removeLast(), operandStack.removeLast())
            enter()
        }
    }
    
    func performOperation(operation: Double -> Double) {
        if operandStack.count >= 1 {
            displayValue = operation(operandStack.removeLast())
            enter()
        }
    }
```
    
出现了这样的报错:Method with Objective-C selector conflicts with previous declaration with the same Objective-C selector
[stackoverflow](http://stackoverflow.com/questions/29457720/compiler-error-method-with-objective-c-selector-conflicts-with-previous-declara)中同样的解答
### 提到的问题:
 1. The problem is UIViewController is an @objc class. When inheriting from UIViewController, BugViewController is also a @objc class.
 2. ObjC doesn't support method overloading (two methods with the same name) 

### 解决方法有以下三种：
 
```Swift
    func performOperation(operation: (Double, Double) -> Double) {
        if operandStack.count >= 2 {
            displayValue = operation(operandStack.removeLast(), operandStack.removeLast())
            enter()
        }
    }
    @nonobjc 
    func performOperation(operation: Double -> Double) {
        if operandStack.count >= 1 {
            displayValue = operation(operandStack.removeLast())
            enter()
        }
    }
```
    
```Swift
    func performOperation(operation: (Double, Double) -> Double) {
        if operandStack.count >= 2 {
            displayValue = operation(operandStack.removeLast(), operandStack.removeLast())
            enter()
        }
    }
    @objc(methodTow:)
    func performOperation(operation: Double -> Double) {
        if operandStack.count >= 1 {
            displayValue = operation(operandStack.removeLast())
            enter()
        }
    }
```
    
```Swift
    func performOperation(operation: (Double, Double) -> Double) {
        if operandStack.count >= 2 {
            displayValue = operation(operandStack.removeLast(), operandStack.removeLast())
            enter()
        }
    }
    private func performOperation(operation: Double -> Double) {
        if operandStack.count >= 1 {
            displayValue = operation(operandStack.removeLast())
            enter()
        }
    }
```
    