---
layout: post
title: "兵器谱之Keyboard Maestro"
date: 2015-04-29 15:12:45 +0800
comments: true
categories: 
---
Keyboard Maestro是一款类似Alfred的App，但一点也不逊色后者，它对Automator进行了封装，方便用户的使用。


#Firefox快捷设置
使用浏览器时，我个人非常不习惯网页的白色背景，感觉非常刺眼，于是手动调成灰色，但个别网页不能正常显示，这时只能手动来切换，不管是鼠标点击还是键盘操作都非常烦锁。

Duang！有请Keyboard Maestro（简写成KM）出场，先在KM的编辑器里为我使用的Firefox添加一个Group并命名，在Avaliable in the following applications下拉菜单中选中Firefox，这样之后的操作只会在Firefox中生效，减少全局冲突；接着在中间的Macros栏中点击加号按钮添加一个Macro，取名为`Backgroud Color White`
![group](/images/group.png)

KM有许多触发动作的机制，这里我设置一个快捷键`⌃⌘I`，当切换到Firefox中并按下这个快捷捷，就会触发一系列自定义的action。点击`New Action`，弹出选择列表，作为一个键盘党，按照Firefox修改背景颜色的键盘顺序，添加`Interface Control`中的`Type a keystroke`就可以了。问题来了，当我进入`常规`选项卡里进行设置后，再触发前面设置的快捷键，预设的一系列键盘操作就会在`常规`里面跳转。

<!--more-->

看来只能用鼠标了，为了统一基准坐标，我把设置窗口拖动到左上角，与Firfox窗口重合，`Interface Control`的`Move or Click Mouse`支持鼠标的操作，再添加了相关的action后再试，问题又来了，预先的操作有时成功、有时失败，通过一次次的窗口操作，发现了原因：平时我们手动操作的时候，每一步之间都有间隔时间的，而KM自动执行这一系列action之间则没有间隔时间（后来知道KM支持Debug功能，不用这么费力的调试了），幸好在`Control Flow`里找到了`Pause`，再每个action之间添加它，并设置间隔时间为0.2秒，这回终于正常了。
![white](/images/white.gif)

上面手动添加action还是挺费力了，幸好KM支持Macros的录制功能，刚才是把背景色改为白色，那就录个换回灰色的Macro吧。
![white](/images/gray.gif)

最终生成的action列表
![](/images/actions.png)

点击`edit`按钮，可以看到action的文字说明
![](/images/edit.png)

KM内置了上百种功能强大的action，而且也支持自定义action，接下来好好看看官方的文档，写几个自定义的action！

