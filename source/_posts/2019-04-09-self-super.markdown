---
layout: post
title: "self super"
date: 2019-04-09 17:44:00 +0800
comments: true
categories: 
---
重新认识下 self super，及msgSend msgSendSuper<!--more-->
###  先看几行行老生常谈的代码。。。

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
其输出结果如图所示：(其中 runtimeTest 继承NSObject,我们在runtimeTest类中 测试如上代码)
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
从objc/message.h中 objc_msgSend 的参数说明可以看出其第一个参数为消息的接收者，
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/2.png)
而objc_msgSendSuper 参数的第一个参数为一个名为objc_super 的结构体，其构成如下
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/3.png)
从上面的obj1和obj2 的示例里可以看出，obj1的接受者参数为self，obj2在objc_super的结构体中，receiver的值同样是self.
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/4.png)
下面在来看NSObject的class 方法实现，[源码地址](https://opensource.apple.com/source/objc4/objc4-208/runtime/Object.m.auto.html)

