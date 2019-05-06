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
可以看见<code>@autoreleasepool</code> 被转换成了<code>__AtAutoreleasePool</code>，在当前文件搜索出现下面内容。
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
上面是<code>\_\_AtAutoreleasePool</code>结构体的组成，其中主要的两个方法 <code>objc_autoreleasePoolPush()</code> 和<code>objc_autoreleasePoolPop()</code>，这两个方法我们可以在[NSObject.mm的源码](https://opensource.apple.com/source/objc4/objc4-646/runtime/NSObject.mm.auto.html)中找到实现如下：
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
关键的来啦：通过上面的代码我们已经知道<code>@autoreleasepool</code> 的背后其实一直在默默工作的是<code>AutoreleasePoolPage</code>，因为在很多老的blog中没有体现出如何找到<code>AutoreleasePoolPage</code>，所以关于autoreleasepool的源码介绍就到这里，这里有一篇[参考资料](https://www.jianshu.com/p/b875065074f2)。这里面会讲述<code>AutoreleasePoolPage</code>的数据结构，及维护原理。

### autoreleasepool在主线程的释放时机   
在demo中加入新的断点。如果在主线程，直接执行lldb命令：<code>po [NSRunLoop currentRunLoop]</code> 如果在子线程则需要MainRunLoop。  
![断点结果](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/autoreleasepool/1.png)  
图片如果看不清的同学可以浏览器网页缩放下。  
我们可以看到主线程的<code>runLoop</code> 中有两个<code>Observer</code>，并且 <code>activities</code> 分别为“0xa0”和“0x1”这是两个十六进制的数，转为二进制分别为“1”和“10100000”，对应<code>CFRunLoopActivity</code>的类型如下：  

```javascript
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
   
   kCFRunLoopEntry = (1UL << 0),               //0x1
   kCFRunLoopBeforeTimers = (1UL << 1),
   kCFRunLoopBeforeSources = (1UL << 2),
   kCFRunLoopBeforeWaiting = (1UL << 5),       //0xa0 = kCFRunLoopBeforeWaiting | kCFRunLoopExit
   kCFRunLoopAfterWaiting = (1UL << 6),
   kCFRunLoopExit = (1UL << 7),                //0xa0 = kCFRunLoopBeforeWaiting | kCFRunLoopExit
   kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```
其中“0x1”即<code>kCFRunLoopEntry</code>，监听<code>RunLoop</code>对象进入循环的事件，“0xa0”即<code>kCFRunLoopBeforeWaiting | kCFRunLoopExit</code>，监听<code>RunLoop</code>即将进入休眠和<code>RunLoop</code>对象退出循环的事件。

程序运行后产生的两个<code>CFRunLoopObserver</code>一个监听<code>RunLoop</code>对象进入循环的事件，执行回调函数<code>_wrapRunLoopWithAutoreleasePoolHandler</code>，并且优先级<code>order</code>为<code>-2147483647</code>即32位整数的最小值，保证了它的优先级最高。在回调内会调用<code>_objc_autoreleasePoolPush</code>函数来创建<code>AutoreleasePool</code>，由于它的优先级最高，所以能够保证自动释放池在其他回调执行前得到创建。

另一个监听器监听<code>RunLoop</code>对象进入休眠和退出循环的事件，回调函数同样是<code>_wrapRunLoopWithAutoreleasePoolHandler</code>，而优先级为<code>2147483647</code>即32位整数的最大值，保证它的优先级最低。对于监听进入休眠状态时回调函数内首先会调用<code>_objc_autoreleasePoolPop</code>函数来释放<code>AutoreleasePool</code>然后使用<code>_objc_autoreleasePoolPush</code>函数重新创建一个自动释放池。优先级最低保证了释放操作是在其他所有回调执行之后发生。  

### <code>autoreleasepool</code>在子线程的释放时机  
问题分解：  
1. 子线程中是否存在<code>AutoreleasePool</code>？  
2. 如果存在，那么里面的<code>autorelease</code>对象如何释放呢？  
3. 如果不存在，如何建立一个<code>autoreleasepool</code>？  
4. 自己建立的<code>autoreleasepool</code>何时释放？  

先来回答第一个问题，这里有一份[苹果的开发文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html#//apple_ref/doc/uid/20000047-1041876)这里明确的指出了 
> Each thread in a Cocoa application maintains its own stack of autorelease pool blocks.  

其实这篇文档基本是回答了上面的所有问题，子线程是存在<code>AutoreleasePool</code>的，在线程退出的时候会去做释放<code>autorelease</code>对象，这也证明了问什么在子线程的<code>for</code>循环中释放会不及时，因为<code>for</code>里面产生的<code>autorelease</code>对象是要等到线程退出的时候再去释放的。下面举个例子
```javascript

dispatch_async(dispatch_get_global_queue(0, 0), ^{

    for (int i = 0; i<1000000000; i++) {
   //@autoreleasepool {  
      AutoReleaseModel *model = [[AutoReleaseModel alloc] init];
      NSString *str = [NSString stringWithFormat:@"hi + %d", i];
      NSLog(@"str = %@", str);
   //}
   }
});
```
我们运行下面代码会发现，内存在逐步增加。
![如图](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/autoreleasepool/2.png)
当打开注释掉的<code>@autoreleasepool</code>时，内存如下
![如图](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/autoreleasepool/3.png)
所以子线程是可以建立自己的<code>AutoreleasePool</code>的，并且自己建立的<code>autoreleasepool</code>里的面对象在出了block就会释放，这里可以参考 <code>NSAutoreleasePool</code>，我们可打开MRC，在<code>AutoReleaseModel</code>的<code>dealloc</code>出加入断点，执行如下代码：   

```javascript

dispatch_async(dispatch_get_global_queue(0, 0), ^{

   for (int i = 0; i<1000000000; i++) {

      NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
      NSString *str = [NSString stringWithFormat:@"hi + %d", i];
      NSLog(@"str = %@", str);
      AutoReleaseModel *model = [[AutoReleaseModel alloc] init];
      [model autorelease];
      [pool drain];
}
});

```
结果如下：
![如图](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/autoreleasepool/4.png)
<code>@AutoreleasePool</code>的括号其实分别就是<code>@NSAutoreleasePool</code> 的初始化和<code>drain</code>方法。当出了<code>}</code>，就相当与调用<code>drain</code>，从而调用<code>AutoreleasePoolPage::pop</code>，向栈中的对象发送 release 消息  
这里要补充一点 在ARC中，所有对象在<code>alloc</code>，<code>init</code>之后在编译后，都会被编译器加上<code>autorelease</code>如下

```javascript
//原代码
AutoReleaseModel *model = [[AutoReleaseModel alloc] init];

//编译之后
//AutoReleaseModel *model = [[AutoReleaseModel alloc] init];
//[model autorelease]
```
而<code>autorelease</code>的释放只能依赖<code>@AutoreleasePool</code>，也就是说<code>autorelease</code>对象会被加入<code>@AutoreleasePool</code>，其实现如下

```javascript

- [NSObject autorelease]
   └── id objc_object::rootAutorelease()
       └── id objc_object::rootAutorelease2()
            └── static id AutoreleasePoolPage::autorelease(id obj)
                 └── static id AutoreleasePoolPage::autoreleaseFast(id obj)
                 ├── id *add(id obj)
                 ├── static id *autoreleaseFullPage(id obj, AutoreleasePoolPage *page)
                 │   ├── AutoreleasePoolPage(AutoreleasePoolPage *newParent)
                 │   └── id *add(id obj)
                 └── static id *autoreleaseNoPage(id obj)
                     ├── AutoreleasePoolPage(AutoreleasePoolPage *newParent)
                     └── id *add(id obj)
```
### 结论
1. 在不自己建立<code>@AutoreleasePool</code>的情况下，主线程是通过监听<code>CFRunLoopObserver</code>来决定释放<code>autorelease</code>对象的时机的，而子线程则是在退出线程的时候来释放<code>autorelease</code>对象。  
2. 如果自己手动建立 <code>@AutoreleasePool</code>，那么<code>autorelease</code>对象的声明周期仅在block块内，不论在主线程还是子线程都是如此。
