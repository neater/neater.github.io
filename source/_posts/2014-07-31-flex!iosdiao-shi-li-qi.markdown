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
```objective-c
[[FLEXManager sharedManager] showExplorer];
```
### 完整版本
```objective-c
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
  

### 编辑NSUserDefaults

FLEX允许修改NSUserDefaults的strings, numbers, arrays, and dictionaries，及其它们的组合，输入格式为json；如果使用其它数据作为key，如NSDate，则是只读的。

![NSUserDefaults](https://camo.githubusercontent.com/c9b72bf288f79993fbbc46cd4c0c37504fd8e11b/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f6e737573657264656661756c74732d656469746f722e676966)

### 探索其它APP
代码注入留给大家去摸索了

![Injection](https://camo.githubusercontent.com/de456cb9f822094e49b40692cd9720c8e99905d7/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f666c65782d726561646d652d726576657273652d312e706e67)![Injection](https://camo.githubusercontent.com/44624ad09a907893fc95bf35283bc12439ae9d93/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f666c65782d726561646d652d726576657273652d322e706e67)

## 避免将FLEX编译到正式版本
尽管FLEX很适合在开发调试的时候使用，但它不应该让最终用户看到。在Xcode -> Project -> Build Setting，点`+`选择`Add User-Defined Setting`。

![Excluding FLEX](https://camo.githubusercontent.com/5b1cbb5cb14496ee12a8a8aeacc2c155a735a1c1/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f666c65782d726561646d652d6578636c7564652d312e706e67)

命名为`EXCLUDED_SOURCE_FILE_NAMES`，在`Release`配置中，填入`FLEX*`，将不会编译所有以FLEX开头的文件，`Debug`配置里空白就行了。

![](https://camo.githubusercontent.com/843997bca76f737561e1084293e9dfd90cda4d97/687474703a2f2f656e67696e656572696e672e666c6970626f6172642e636f6d2f6173736574732f666c65782f666c65782d726561646d652d6578636c7564652d322e706e67)

在所有集成FLEX代码的地方，确保将其嵌入到`#if DEBUG`中，更多相关信息，请参考官方示例。

## 注意
* NSUserDefaults中，如果值的类型为`id`，FLEX将把输入的string转为json，目的是使用strings, numbers, arrays, and dictionaries的组合。如果值为string，则一定用字符串引号包裹起来，显示使用 `NSString` 的属性和实例变量，则不用引号。
* 在使用FLEX时，你可能会取消异常断点。FLEX会拨出一些异常，当接收了某种不能处理的输入（如`NSGetSizeAndAlignment()`）, 为了防止程序崩溃，FLEX会捕获并将其抛出，但是这样会激活你的断点。

译自 [FLEX README.md](https://github.com/Flipboard/FLEX/)
