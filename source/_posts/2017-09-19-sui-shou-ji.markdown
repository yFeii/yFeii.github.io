---
layout: post
title: "随手记"
date: 2017-09-19 15:15:12 +0800
comments: true
categories: 
---

本文意在收集开发中不经意发现的小惊喜，积少成多，方至千里。<!--more-->

* (autoLayout是个好东西，但是在滚动视图里(UIScrollView等)，childView的约束加在scrollView上会出现各种好玩的问题如：contentSize设置无效，childView宽度或高度设置无效等等各种无法达到frame方式的表现，代码如下：
```javascript
[childView mas_makeConstraints:^(MASConstraintMaker *make) {
make.width.mas_equalTo(scrollView.mas_width);
make.height.mas_equalTo(scrollView.mas_height);
make.top.mas_equalTo(scrollView.mas_top);
make.bottom.mas_equalTo(scrollView.mas_bottom);
}];
```
原因：scrollView的contentSize是由childView的尺寸决定，换句话说也就是，必须确定childView的Size和position(x,y);解决这个问题可以建一个contentView并确定其位置大小，然后childView add 到contentView上)。

* Xcode在运行时是根据buildPhases->Compile Sources里面的从上至下顺序编译的，编译时通过压栈的方式将多个分类压栈，根据后进先出的原则，后编译的会被先调用，当objc_msgSend找到方法并调用之后，就不再继续传递消息，所以形成所谓上的覆盖。并不是后面创建的就一定被调用，得看创建之后其在buildPhases->Compile Sources里面的位置

* navigationBar 设置导航背景如果 图片高度不够64（不能填充statusBar+navBar）时，在iOS10之前不会拉伸填充满高度，在iOS10之后会自动处理（假设原图 375*44，最终效果就是从左上角(0，0)开始铺第一张图到0~44高度,然后从44~64取原图的前20高拼接)

