---
layout: post
title: iOS多线程学习笔记
date: 2016-5-23
categories: blog
tags: [学习笔记,iOS]
description: 
---


# NSThread

## 1 创建线程
首先看Xcode文档，有这样一句

```
Prior to OS X v10.5, the only way to start a new thread is to use the detachNewThreadSelector:toTarget:withObject: method. In OS X v10.5 and later, you can create instances of NSThread and start them at a later time using the start method.
```
得出创建线程并启动的方法

### 1.1 创建并启动线程

```objectivec
+ (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(nullable id)argument;
```
可以创建线程并启动线程

### 1.2 先创建线程，再启动线程

```objectivec
- (instancetype)initWithTarget:(id)target selector:(SEL)selector object:(nullable id)argument NS_AVAILABLE(10_5, 2_0);
- (void)start NS_AVAILABLE(10_5, 2_0);
```
先初始化新的线程，再用start开启线程

```objectivec
NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(thread1) object:nil];
NSLog(@"hahah");
[thread start];
```

## 2 停止线程

### 2.1 延时

```objectivec
+ (void)sleepUntilDate:(NSDate *)aDate
//Blocks the current thread until the time specified.
//aTime:The time at which to resume processing.
//Discussion:No run loop processing occurs while the thread is blocked.
```

```objectivec
+ (void)sleepForTimeInterval:(NSTimeInterval)ti
```

### 2.2 退出线程 

```objectivec
+ (void)exit
//Terminates the current thread.对象方法，只能写在线程内部，退出当前线程
```
```objectivec
- (void)cancel
//对象方法，可在线程函数外部使用，退出某个线程，NSOperation中也有用到这个
```

## 3 NSThread的其他一些常用属性和方法

```objectivec
+ (BOOL)isMultiThreaded
+ (BOOL)isMainThread
+ (NSThread *)mainThread
+ (NSThread *)currentThread
@property(readonly) BOOL isMainThread
@property(readonly, getter=isExecuting) BOOL executing
@property(readonly, getter=isFinished) BOOL finished
@property(readonly, getter=isCancelled) BOOL cancelled
```

## 4 还有几个通知方法

```objectivec
NSDidBecomeSingleThreadedNotification
NSThreadWillExitNotification
NSWillBecomeMultiThreadedNotification
```

# NSOperation && NSOperationQueue

## 1 Managing Operations in the Queue

```objectivec
- (void)addOperation:(NSOperation *)operation
- (void)addOperations:(NSArray<NSOperation *> *)ops
    waitUntilFinished:(BOOL)wait
- (void)addOperationWithBlock:(void (^)(void))block
- (void)cancelAllOperations
- (void)waitUntilAllOperationsAreFinished
```

# GCD 

## 1 Creating and Managing Queues

```objectivec
//1.获取主队列
dispatch_queue_t dispatch_get_main_queue(void);
//2.获取全局并行队列
dispatch_queue_t dispatch_get_global_queue( long identifier, unsigned long flags);
//其中参数identifier： dispatch_queue_priority_t //优先级
#define DISPATCH_QUEUE_PRIORITY_HIGH        2
#define DISPATCH_QUEUE_PRIORITY_DEFAULT     0
#define DISPATCH_QUEUE_PRIORITY_LOW         (-2)
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND  INT16_MIN
3.创建队列
dispatch_queue_t dispatch_queue_create( const char *label dispatch_queue_attr_t attr);
//label:一个标识，比如传入 @"下载队列"
//attr:DISPATCH_QUEUE_SERIAL (or NULL): to create a serial queue  //创刊串行队列
//     DISPATCH_QUEUE_CONCURRENT: to create a concurrent queue.  //创建并行队列
```

## 2 Queuing Tasks for Dispatch

```objectivec
void dispatch_async( dispatch_queue_t queue, dispatch_block_t block);//异步
void dispatch_sync( dispatch_queue_t queue, dispatch_block_t block);//同步
```
注意：用dispatch_sync方式时，第一个参数queue不能传递 dispatch_get_main_queue()，不然会造成整个应用卡死
