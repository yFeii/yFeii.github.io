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
其输出结果如图所示：(其中<code> runtimeTest</code> 继承<code>NSObject</code>,我们在<code>runtimeTest</code>类中 测试如上代码)

![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/1.png)

利用<code>clang </code>命令查看其C++实现  ，其中XXXX 为目标文件
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
从<code>objc/message.h</code>中 <code>objc_msgSend </code>的参数说明可以看出其第一个参数为消息的接收者，
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/2.png)
而<code>objc_msgSendSuper </code>参数的说明中，我们需要注意一点，<code>objc_msgSendSuper</code>方法是直接从父类的方法列表中找实现，所以当我们使用<code>super</code>调用
方法时，运行时会将<code>[super callFunction]</code>转换为<code>objc_msgSendSuper</code>并且父类的方法中寻找实现，以此来实现调用父类方法，如下图
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/2019041002.png)

<code>objc_msgSendSuper</code>第一个参数为一个名为<code>objc_super </code>的结构体，其构成如下
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/3.png)

从上面的<code>obj1</code>和<code>obj2</code> 的示例里可以看出，<code>obj1</code>的接受者参数为<code>self</code>，<code>obj2</code>在<code>objc_super</code>的结构体中，<code>receiver</code>的值同样是<code>self</code>.
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/4.png)
下面在来看NSObject的<code>class</code>方法实现[源码地址](https://opensource.apple.com/source/objc4/objc4-208/runtime/Object.m.auto.html)
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/2019041001.png)
在<code>class</code>的实现中，我们看到其返回值为<code>isa</code>，同时我们也注意到，在发起消息时传入的接收者都为<code>self</code>(obj1和obj2的示例中，即为runtimeTest类的实例)，
所以在其父类的<code>class</code>方法实现中，此时<code>isa</code> 也都为<code>runtimeTest</code>类，所以我们最终obj1和obj2的打印结果都为runtimeTest,  
但是在obj3和obj4的实例中，<code>superclass</code>方法返回的是<code>((struct objc_class *)self)->super_class</code>。所以结果为<code>runtimeTest</code>的父类即：<code>NSObject</code> 。  
**2019.05.13更新**
>>这里面有一个错误，就是上面截图的class方法  是类方法，并不是实例方法。下面补充实例方法的实现
![Markdown](https://yfeii-blog.oss-cn-hangzhou.aliyuncs.com/img/self_super/1.png)

### 总结
* <code>self</code> 与<code>super</code> 在调用方法时，在运行时的变现有所不同，<code>self</code>为<code>objc_msgSend</code>，<code>super</code>为<code>objc_msgSendSuper</code>。
* <code>objc_msgSend</code> 会优先从当前类中找实现。
* <code>objc_msgSendSuper</code> 直接从父类中寻找实现。
* 之所以上面4个示例的输出结果为runtimeTest，runtimeTest，NSObject，NSObject，是由消息的接收者与父类的方法共同决定的。
