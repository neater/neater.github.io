---
layout: post
title: "用Ruby写Alfred2的Workflow"
date: 2015-04-13 15:25:57 +0800
comments: true
categories: 
---
Mac下的大杀器Alfred2用了好久，Workflow也收集了不少，那为什么不自己写一个呢？当添加一个Workflow时发现，竟然支持这么多的语言，那就用我的最爱Ruby开工吧！

![view frame](../images/languages.png)

第一次就拿ASCII码转换练练手吧，可以将十进制ASCII码转换成字符

![](http://ytliu.info/images/2014-04-14-1.png)

还可以将16进制ASCII转换成字符

![](http://ytliu.info/images/2014-04-14-2.png)

或者把字符转换成ASCII码

![](http://ytliu.info/images/2014-04-14-3.png)

<!--more-->

这里使用了[alfred2-ruby-template](http://zhaocai.github.io/alfred2-ruby-template/)模板，使用过程如下

```bash
$ git clone https://github.com/zhaocai/alfred2-ruby-template.git
$ mv alfred2-ruby-template alfred2-ascii-translator
$ cd alfred2-ascii-translator
```

更新config.yml的`domain`, `id`, `path`字段：

```yml
# bundle_id = "domain.id"
# path is the relative path to the workflow in the project root
---
path: ascii
domain: me.ytliu
id: alfred2-ascii-translator
# If you are using Alfred's advanced Dropbox sync, indicate the path shown in
# Alfred Preferences > Advanced > Syncing:
dropbox: ~/Dropbox/Alfred
```