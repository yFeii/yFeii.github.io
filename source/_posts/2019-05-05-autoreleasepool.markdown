---
layout: post
title: "autoreleasepool"
date: 2019-05-05 16:21:13 +0800
comments: true
categories: 
---
autoreleasepool与runloop，线程（主线程，子线程）之间的关系。<!--more-->
### 前言  
本文主要讲述autoreleasepool，其中runloop与线程的关系不作为重点（可参考的资料很多，当然本文也会稍提），想写demo测试验证的同学可以尝试在MRC下断点查看（在ARC下断点方法的调用栈有很多都被隐藏掉了）,本文主要解决的问题如下：  
1. autoreleasepool的原理
2. autoreleasepool在主线程的释放时机
3. autoreleasepool在子线程的释放时机(子线程默认不开启runloop)
4. 如果在子线程内需要使用 autoreleasepool，如何建立，block内的对象何时释放。
### autoreleasepool
先放一份[autoreleasepool的源码地址](https://opensource.apple.com/source/objc4/objc4-646/runtime/NSObject.mm.auto.html)。新建demo工程。clang main.m文件，打开main.cpp文件查看源码如下
```javascript

int main(int argc, char * argv[]) {
   //可以看见@autoreleasepool 被转换成了__AtAutoreleasePool。
   /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
      return UIApplicationMain(argc, argv, __null, NSStringFromClass(((Class (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("AppDelegate"), sel_registerName("class"))));
   }
}
static struct IMAGE_INFO { unsigned version; unsigned flag; } _OBJC_IMAGE_INFO = { 0, 2 };
```
可以看见@autoreleasepool 被转换成了__AtAutoreleasePool，在当前文件搜索出现下面内容。
```javascript

struct __AtAutoreleasePool {
   __AtAutoreleasePool() {
      atautoreleasepoolobj = objc_autoreleasePoolPush();
   }
   ~__AtAutoreleasePool() {
   objc_autoreleasePoolPop(atautoreleasepoolobj);
   }
   void * atautoreleasepoolobj;
};
```
上面是\_\_AtAutoreleasePool结构体的组成，其中主要的两个方法 <code>objc_autoreleasePoolPush()</code> 和*objc_autoreleasePoolPop()*，这两个方法我们可以在[NSObject.mm的源码](https://opensource.apple.com/source/objc4/objc4-646/runtime/NSObject.mm.auto.html)中找到实现如下：
```javascript
void *objc_autoreleasePoolPush(void){
   if (UseGC) return nil;
   return AutoreleasePoolPage::push();
}

void objc_autoreleasePoolPop(void *ctxt) {
   if (UseGC) return;
   // fixme rdar://9167170
   if (!ctxt) return;
   AutoreleasePoolPage::pop(ctxt);
}
```
