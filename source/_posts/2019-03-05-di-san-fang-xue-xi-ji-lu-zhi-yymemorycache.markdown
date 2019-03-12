---
layout: post
title: "第三方学习记录之YYMemoryCache"
date: 2019-03-05 20:22:07 +0800
comments: true
categories: 
---

YYMemoryCache学习笔记<!--more-->

### 我们可以从类名很直观的看出YYCache的几种持久化方式，本文主要讲解其中的内存缓存（YYMemoryCache）

#### 1.关键类说明

##### YYLinkedMap YYLinkedMapNode
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
从上面的代码的我们可以看出 YYLinkedMap 类 是主要由一个 dic 及两个YYLinkedMapNode 对象构成。dic的作用是提供给我们快速查出缓存对象是否存在，而YYLinkedMapNode 则是一个双向链表，其维护的是所有缓存对象并且按最一次使用的时间依次递减（最近用的在头，在最后的则是最不常用的）。如下图：




#### 2.设计目的

这里面之所以要采用字典（hash）+双向链表的方式是[LRU](https://baike.baidu.com/item/LRU/1269842?fr=aladdin)算法的一种实现方式。如果单单用链表来维护缓存对象，则每次查找（命中）要花费(O)N,如果采用hash其本身是无序的，没办法保证不常用的一直在最后。所以采用hash（查找效率高）+双向链表来实现。插入过程如图所示



