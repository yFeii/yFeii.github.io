---
layout: post
title: "随手记"
date: 2017-09-19 15:15:12 +0800
comments: true
categories: 
---

本文意在收集开发中不经意发现的小惊喜，积少成多，方至千里。<!--more-->"

2017.9.19
>Xcode在运行时是根据buildPhases->Compile Sources里面的从上至下顺序编译的，编译时通过压栈的方式将多个分类压栈，根据后进先出的原则，后编译的会被先调用，当objc_msgSend找到方法并调用之后，就不再继续传递消息，所以形成所谓上的覆盖。并不是后面创建的就一定被调用，得看创建之后其在buildPhases->Compile Sources里面的位置
