---
layout: post
title: "radiusCollectionView"
date: 2018-02-06 15:17:38 +0800
comments: true
categories: 
---

无意间发现一个布局好玩的app，手痒写了个简单的demo<!--more-->

核心方法在于自定义flowLayout 中的这个方法
```javascript
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath {

}
```
更新布局，要点：勾股定理😄
