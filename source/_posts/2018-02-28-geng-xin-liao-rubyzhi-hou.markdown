---
layout: post
title: "更新了ruby之后 - -"
date: 2018-02-28 14:28:48 +0800
comments: true
categories: 
---

偷了一个网络请求库，在pod update的时候发现抛错，试了很多办法最后无奈选择升级ruby..demo<!--more-->


核心方法在于自定义flowLayout 中的这个方法更新布局，
```javascript
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath {

}
```
