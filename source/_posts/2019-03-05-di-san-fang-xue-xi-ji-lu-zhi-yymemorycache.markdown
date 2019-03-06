---
layout: post
title: "第三方学习记录之YYMemoryCache"
date: 2019-03-05 20:22:07 +0800
comments: true
categories: 
---

YYMemoryCache学习笔记<!--more-->

### 我们可以从类名很直观的看出YYCache的几种持久化方式，本文主要讲解其中的内存缓存（YYMemoryCache）

#### 关键类说明

#### YYLinkedMap
```javascript
@interface _YYLinkedMap : NSObject {   
    @package
    CFMutableDictionaryRef _dic; // do not set object directly
    NSUInteger _totalCost;   （总的内容开销）
    NSUInteger _totalCount;   （当前缓存总数）
    _YYLinkedMapNode *_head; // MRU, do not change it directl（链表头）
    _YYLinkedMapNode *_tail; // LRU, do not change it directly（链表尾）
    BOOL _releaseOnMainThread;   （是否在主线程释放 _YYLinkedMapNode对象）
    BOOL _releaseAsynchronously; （是否异步释放 _YYLinkedMapNode对象）
}

- (instancetype)init {
   self = [super init];
   _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
   _releaseOnMainThread = NO;
   _releaseAsynchronously = YES;
   return self;
}
```
从上面的代码的我们可以看出 YYLinkedMap 类 是主要由一个 dic 及两个YYLinkedMapNode 对象构成，

