---
layout: post
title: "更新了ruby之后 - -"
date: 2018-02-28 14:28:48 +0800
comments: true
categories: 
---

偷了一个网络请求库，在pod update的时候发现抛错，试了很多办法最后无奈选择升级ruby.. <!--more-->

年前写了一个[网络请求库](https://github.com/yFeii/RadiusCollectionView)，发布到Cocoapods一切正常却发现 pod search 不到，通过如下方法解决[原文](https://www.jianshu.com/p/b5e5cd053464)

```javascript
  rm ~/Library/Caches/CocoaPods/search_index.json
```
