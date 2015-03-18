---
layout: post
title: "iOS中的Hit-Testin"
date: 2015-03-11 16:15:49 +0800
comments: true
categories: 
---
Hit-Testing决定了一个触控点是否和屏幕上的视图相关，iOS使用Hit-testing找到最前端的并且该视图能够接收用户触碰产生的事件。这背后的实现采用了reverse pre-order depth-first遍历算法。

解释Hit-Testing原理之前，先来看看它产生的时机，下面的示意图说明了一次touch从手指触碰屏幕到离开的过程
![](http://smnh.me/images/hit-test-touch-event-flow.png)

如图所示，每次触屏都会产生hit-testing，并且在相应的视图或者手势识别器接收`UIEvent`事件之前产生的。

```
注意：由于未知的原因，hit-testing接连产生多产，而确定的hit-test view 保持不变。
```

hit-testing完成后，找到了触碰点在其上面并处在最前端的视图。该视图会接收所有阶段（开始、移动、结束或者取消）的UITouch事件，当然它的手势识别器和子视图也可以和这些事件产生关联，然后，hit-test就开始接收一系列的UItouch事件了。

有一点要注意，即使手指从hit-test视图移到移动到另一个视图上，hit-test视图仍然会继续收到touch事件，直到该组touch结束

> “The touch object is associated with its hit-test view for its lifetime, even if the touch later moves outside the view.”
> 
> [Event Handling Guide for iOS, iOS Developer Library](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW4)

As mentioned earlier the hit-testing uses depth-first traversal in reverse pre-order (first visiting the root node and then traversing its subtrees from higher to lower indexes). This kind of traversal allows reducing the number of traversal iterations and stopping the search process once the first deepest descendant view that contains the touch-point is found. This is possible since a subview is always rendered in front of its superview and sibling view is always rendered in front of its sibling views with a lower index into the subviews array. Such that, when multiple overlapping views contain specific point, the deepest view in the rightmost subtree will be the frontmost view.

>“Visually, the content of a subview obscures all or part of the content of its parent view. Each superview stores its subviews in an ordered array and the order in that array also affects the visibility of each subview. If two sibling subviews overlap each other, the one that was added last (or was moved to the end of the subview array) appears on top of the other.”
>
>[View Programming Guide for iOS, iOS Developer Library](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW24)
       


下图显示了一个视图继承树并展示与其对应的真机模拟图。从左至右的的排列方式显示出了子视图的排列关系（右边的视图离用户越近）。

![view hierarchy tree](http://smnh.me/images/hit-test-view-hierarchy.png)

As it can be seen, “View A” and “View B” as well as their children, “View A.2” and “View B.1”, are overlapping. But since “View B” has a subview index higher than that of “View A”, “View B” and its subviews are rendered above “View A” and its subviews. Therefore, “View B.1” should be returned by hit-testing when user’s finger touches “View B.1” in the area where it overlaps with “View A.2”.

By applying depth-first traversal in reverse pre-order allows stopping the traversal once the first deepest descendant view that contains the touch-point is found:

![](http://smnh.me/images/hit-test-depth-first-traversal.png)
