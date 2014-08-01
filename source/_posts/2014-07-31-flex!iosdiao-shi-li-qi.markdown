---
layout: post
title: "FLEX！iOS调试利器"
date: 2014-07-31 15:51:37 +0800
comments: true
categories: 
---
FLEX (Flipboard Explorer)是一系列集成在APP内部的用于iOS调试的工具集，以工具栏的形式显示在应用中，通过它，可以查看并修改几乎所有的APP运行状态。


![dfd](https://camo.githubusercontent.com/9986601c5e4306f7935032465911c0f70596e046/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f62617369632d766965772d6578706c6f726174696f6e2e676966)


## 强大的调试能力
* 查看、修改View的层次关系。
* 查看对象的属性和实例变量。
* 动态修改属性和实例变量。
* 动态调用实例和类方法。
* 动态访问堆上的对象。
* 访问APP的沙盒。
* 除jAPP的所有类外，还能访问已链接的系统框架，包括私有的。
* 快速访问一些常用的对象，比如`[UIApplication sharedApplication]`、App Delegate、Root View Controller等。
* 动态查看、修改`NSUserDefaults`存储的值。

和其它调试工具不同，FLEX完全运行在你的APP中，所以不需要连接到LLDB/Xcode或者远程调试的服务端，在模拟器和真机上都运行的很好。
## 用法
### 简单版本
```
[[FLEXManager sharedManager] showExplorer];
```
### 完整版本
```
#if DEBUG
#import "FLEXManager.h"
#endif

...

- (void)handleSixFingerQuadrupleTap:(UITapGestureRecognizer *)tapRecognizer
{
#if DEBUG
    if (tapRecognizer.state == UIGestureRecognizerStateRecognized) {
        // This could also live in a handler for a keyboard shortcut, debug menu item, etc.
        [[FLEXManager sharedManager] showExplorer];
    }
#endif
}
```
##功能示例
###修改视图
点击`views`，从工具栏下面弹出界面，显示视图的详细信息，并能修改属性值和调用方法。

![Views](https://camo.githubusercontent.com/950a2612b1dc796bc5cc3fd9909ed465166afc5b/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f616476616e6365642d766965772d65646974696e672e676966)	

### 查看对象
FLEX在malloc分配的内存块中查找相关的对象，如下：


![Object in Heap](https://camo.githubusercontent.com/573692941c2901c0fd1ce0f085c101f6b4d3ae3b/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f686561702d62726f777365722e676966)

### 文件浏览
查看APP沙盒内的文件系统，包括文件大小、图片预览、以友好的方式显示json和plist文件。也能将文本和图片文件拷贝和剪贴板中，

![File Browser](https://camo.githubusercontent.com/df6e924a21ecaf8080342d80f384e88f8249c3fe/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f66696c652d62726f777365722e676966)

### 探索系统库资源
深挖框架内部的class，创建相应的实例，并访问其状态。    
  
![System Library Exploration](https://camo.githubusercontent.com/c91fc34a63f05f803cdc0d23d72ae047d0b960bd/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f73797374656d2d6c69627261726965732d62726f777365722e676966)
  

