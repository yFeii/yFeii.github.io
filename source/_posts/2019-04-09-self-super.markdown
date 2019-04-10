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
而objc_msgSendSuper 参数的说明中，我们需要注意一点，objc_msgSendSuper方法是直接从父类的方法列表中找实现，所以当我们使用super调用
方法时，运行时会将[super callFunction]转换为objc_msgSendSuper并且父类的方法中寻找实现，以此来实现调用父类方法，如下图
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/2019041002.png)

objc_msgSendSuper第一个参数为一个名为objc_super 的结构体，其构成如下
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/3.png)

从上面的obj1和obj2 的示例里可以看出，obj1的接受者参数为self，obj2在objc_super的结构体中，receiver的值同样是self.
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/4.png)
下面在来看NSObject的class方法实现[源码地址](https://opensource.apple.com/source/objc4/objc4-208/runtime/Object.m.auto.html)
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/2019041001.png)
在class的实现中，我们看到其返回值为self，同时我们也注意到，在发起消息时传入的接收者都为self(obj1和obj2的示例中，即为runtimeTest类的实例)，
所以在其父类的class方法中，此时self 也为runtimeTest实例，所以我们最终obj1和obj2的打印结果都为runtimeTest,但是在obj3和obj4的实例中，
superclass方法返回的是((struct objc_class *)self)->super_class。所以结果也为runtimeTest的父类即：NSObject
### 总结
* self 与super 在调用方法时，在运行时的变现有所不同，self为objc_msgSend，super为objc_msgSendSuper。
* objc_msgSend 会优先从当前类中找实现，其方法参数与objc_msgSendSuper的区别。
* objc_msgSendSuper 直接从父类中寻找实现，objc_super结构体中参数的含义。（接收者，父类）
* 之所以上面4个示例的输出结果为runtimeTest，runtimeTest，NSObject，NSObject，是由消息的接收者与父类的方法实现有关系的。
