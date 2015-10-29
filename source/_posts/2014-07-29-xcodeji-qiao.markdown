---
layout: post
title: "Xcode技巧"
date: 2014-07-29 14:05:32 +0800
comments: true
categories: 
---
####Edit all in Scope
适合进行大批量的修改变量和方法；选定一个想要修改的字符串，然后选择**Edit－Edit all in Scope**，然后在你输入的时候，所有该字符出现的地方都进行同步更改。

在文件上执行‘Command + Option + Shift + Left-click’操作，该组合键可展示一个小尺寸的弹出视图，可以查看你想要打开它的地方，比如辅助编辑器、标签或者窗口等。

CTRL + 1，该快捷键可打开'Show Related Items‘弹出菜单’。倘若你已经将光标放在了任何方法中，并点击‘CTRL + 1 ’就可以很方便地通过弹出的视图访问该方法的所有调用者和被调用者。

Command + Shift + J快捷操作，可展示当前你在工程导航器中打开的文件

文档和参考: Command + Shift + 0

在辅助编辑器中打开文件:在项目导航器中选中文件执行Option+左键点击操作


option+点击Project Navigator中选中的文件：在辅助编辑窗口中打开选中文件。

option+command+点击Editor中选中的符号：在辅助编辑窗口中打开符号定义（jump to definition in assistant editor）。

option+control+command+↑/↓：在辅助窗口中打开对应的头文件（*.h）/实现文件（*.m,*.mm,*.cc）。

若在按下option的同时按下shift通常会出现一个导航窗格，可选择在new window/tab/assistant-editor显示打开。

###真机截屏

```
debug -> viewDebugging -> Take sceencast of Active Devics,
即使当前程序没有运行，也可以直接截取手机上的图片直接到桌面。
```
另外Show View Frams和Show Alignment Rectangles，也可显示视图边框和对齐矩形，方便UI调试。如下图

![view frame](/images/viewframe.png)

__unused关键字，用来suppress 那种讨厌的unused parameter warning.

`__unused int i;`

NSLog(@"%@",@(array.count));