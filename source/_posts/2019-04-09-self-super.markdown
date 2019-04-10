---
layout: post
title: "self super"
date: 2019-04-09 17:44:00 +0800
comments: true
categories: 
---
重新认识下 self super，及msgSend msgSendSuper<!--more-->
###  先看几行行老生常谈的代码。。。（万恶之源）

```javascript
   NSLog(@"self class = %@",NSStringFromClass(self.class));
   NSLog(@"super class = %@",NSStringFromClass(super.class));
   NSLog(@"self super class = %@",self.superclass);
   NSLog(@"super super class = %@",super.superclass);
```
利用clang 命令查看其C++实现  ，其中XXXX 为目标文件
```javascript
clang -x objective-c -rewrite-objc -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk XXXX.m  
```

