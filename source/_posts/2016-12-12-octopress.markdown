---
layout: post
title: "octopress github"
date: 2016-12-12 18:05:37 +0800
comments: true
categories: 
---

### 搭建基于github的个人博客

#### 一.配置octopress相关（ruby）
    * 确认ruby版本至少是1.9.3，使用ruby --version
    * git clone git://github.com/imathis/octopress.git octopress
    * cd octopress
    * gem install bundler  (//这里可能出现权限问题，在全新的 OS X El Capitan 10.11 上已经使用了 Rootlees , 可以理解为一个更高等级的系统的内核保护. 系统默认将会锁定 /system /sbin /usr 这三个目录. 但是这个保护也是可以关闭的. 解决办法：重启mac，按住command+r，终端输入csrutil disable，重启)
    * bundle install  
    * rake install  这里可能出现sass错误，可以尝试 rake update 解决
[这里](http://octopress.org/docs/setup/)有关octopress资料
#### 一.配置github相关




