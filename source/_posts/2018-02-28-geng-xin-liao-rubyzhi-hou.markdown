---
layout: post
title: "更新了ruby之后 - -"
date: 2018-02-28 14:28:48 +0800
comments: true
categories: 
---

偷了一个网络请求库，在pod update的时候发现抛错，试了很多办法最后无奈选择升级ruby.. <!--more-->

年前写了一个[网络请求库](https://github.com/yFeii/yFeiiNetwork)，发布到Cocoapods一切正常却发现 pod search 不到，通过如下方法解决[原文](https://www.jianshu.com/p/b5e5cd053464)

```javascript
  rm ~/Library/Caches/CocoaPods/search_index.json
```

私有库建好之后一起准备就绪 开始写demo，执行 pod update, duang~ 抛错，尝试各种办法无果，最后只能更新ruby，更新openssl代替系统老板本的openssl，在安装openssl时候出现以下log

```javascript
> brew install openssl
==> Downloading https://homebrew.bintray.com/bottles/openssl-1.0.2h_1.el_capitan.bottle.tar.gz
Already downloaded: /Users/administrator/Library/Caches/Homebrew/openssl-1.0.2h_1.el_capitan.bottle.tar.gz
==> Pouring openssl-1.0.2h_1.el_capitan.bottle.tar.gz
==> Caveats
A CA file has been bootstrapped using certificates from the system
keychain. To add additional certificates, place .pem files in
/usr/local/etc/openssl/certs

and run
/usr/local/opt/openssl/bin/c_rehash

This formula is keg-only, which means it was not symlinked into /usr/local.

Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries

Generally there are no consequences of this for you. If you build your
own software and it requires this formula, you'll need to add to your
build variables:

LDFLAGS:  -L/usr/local/opt/openssl/lib
CPPFLAGS: -I/usr/local/opt/openssl/include
```
可在这两处([1](https://www.cnblogs.com/afluy/p/5462952.html).[2](http://blog.csdn.net/jiamian_/article/details/55098125))找到答案
```javascript
>pod setup
 cannot load such file -- openssl (LoadError)
```
[参考资料](https://stackoverflow.com/questions/14845481/cannot-load-such-file-openssl-loaderror)
<br>总结</br>
* 涉及操作 /usr/bin 类似目录时，重启系统，当启动的时候我们同时按下cmd+r进入Recovery模式，在終端中输入csrutil disable，后执行的操作可加上 sudo如：mv /xxx/xxx -> sudo mv /xxx/xxx
* 在做ln -s 等目录操作时需注意 目录中的版本号对应到你当前的版本，不要按照查到的资料copy
* 看stackoverflow的评论和github的issue 会有意外收获
* 仔细看终端的安装提示，很多时候抛错都会提示对应操作
* 还是吃了英语不好的亏啊

