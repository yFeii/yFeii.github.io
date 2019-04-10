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
id obj1 = self.class;
id obj2 = super.class;
id obj3 = self.superclass;
id obj4 = super.superclass;

NSLog(@"obj1 = %@",obj1);
NSLog(@"obj2 = %@",obj2);
NSLog(@"obj3 = %@",obj3);
NSLog(@"obj4 = %@",obj4);
```
其输出结果如图所示：
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/1.png)

利用clang 命令查看其C++实现  ，其中XXXX 为目标文件
```javascript
clang -x objective-c -rewrite-objc -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk XXXX.m  
```
其编译结果为
```javascript
id obj1 = ((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class"));
id obj2 = ((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("runtimeTest"))}, sel_registerName("class"));
id obj3 = ((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("superclass"));
id obj4 = ((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("runtimeTest"))}, sel_registerName("superclass"));

NSLog((NSString *)&__NSConstantStringImpl__var_folders_ng_8ncwcfkj69n6sh2hxlsfzz300000gn_T_runtimeTest_929dba_mi_0,obj1);
NSLog((NSString *)&__NSConstantStringImpl__var_folders_ng_8ncwcfkj69n6sh2hxlsfzz300000gn_T_runtimeTest_929dba_mi_1,obj2);
NSLog((NSString *)&__NSConstantStringImpl__var_folders_ng_8ncwcfkj69n6sh2hxlsfzz300000gn_T_runtimeTest_929dba_mi_2,obj3);
NSLog((NSString *)&__NSConstantStringImpl__var_folders_ng_8ncwcfkj69n6sh2hxlsfzz300000gn_T_runtimeTest_929dba_mi_3,obj4);
```
我们摘出其关键代码整理如下

