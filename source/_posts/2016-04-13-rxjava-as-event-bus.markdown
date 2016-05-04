---
layout: post
title: "RxJava as event bus"
date: 2016-04-13 08:27:12 +0800
comments: true
categories: 
---

前不久 Square 宣布不在维护 Otto，令人欣喜的是，Android 里还可有 RxJava 能实现同样的功能。

棘手的问题是，如何把基于 Otto 和 Greenbot’s EventBus 的代码迁移到 Rx（有时称为 RxBus)。在讨论使用 RxJava 实现事件总线之前，先来说说事件总线。

###什么是事件总线

很简单，连接两个不同事物的机制，可以有不同的生命周期，在不同的层级，等等。

> 在 `Activity`,`Fragment`,`Service` 还有 `dialog` 之间通讯想必是我们很大的痛点，比如：

1. 在 `Service` 中产生事件，需要在 `Activity` 中改变按钮的颜色
2. `Fragment`需要 `Activity`显示一个 `Dialog`，根据用户之后的操作再改变 `Fragment`中的内容，在这3个源文件中进行回调的操作，光想一下就真够累的，如果再增加几个特性，工作量可想而知了。

事件总线的解决方案是：通过特定的事件总线连接两个类，并发送自定义的数据

在类 A 中产生事件

```java
// EventBus
eventBus.post(new AnswerAvailableEvent(42));

// Otto
bus.post(new AnswerAvailableEvent(42));
```

在类 B 中注册并接收事件

```
// EventBus
eventBus.register(this); // e.g. in onCreate
...
public void onEvent(AnswerAvailableEvent event) {

};

// Otto
@Subscribe 
public void answerAvailable(AnswerAvailableEvent event) {

}
```

### Rx 登场

监听者、Rx、事件总线都使用了响应式模式，只不过 RxJava 做的更好，在我看来，主要是下面几个原因：

1. 更容易追踪代码逻辑
2. 更容易测试
3. 简化线程调用
4. 提供了多种事件流
5. 易操作、易重用、易模块化

###单事件流与多事件流的对比

理解了事件总线和 RxJava 之后，你是不是有自己写个轮子的冲动，我劝你还是打消这个念头吧，先看下面的代码：

```
public class MyRxBus {

  private static MyRxBus instance;

  private PublishSubject<Object> subject = PublishSubject.create();

  public static MyRxBus instanceOf() {
    if (instance == null) {
      instance = new MyRxBus();
    }
    return instance;
  }

  /**
   * Pass any event down to event listeners.
   */
  public void setString(Object object) {
    subject.onNext(object);
  }

  /**
   * Subscribe to this Observable. On event, do something 
   * e.g. replace a fragment
   */
  public Observable<Object> getEvents() {
    return subject;
  }
}
```

还需要引入相应的事件，并进行相应的类型检查。

```
getEvents().subscribe(new Action1<Object>() {
  @Override 
  public void call(Object o) {
    if(o instanceof AnswerAvailableEvent){
      // do something
    }
  }
});
```

上面代码不容易测试，没有做到模块，还引入了不必要的事件类，那么我们引入新的概念

> 模块化的响应式

刚才的代码不易测试，没有模块化，还引入了不必要的事件类，现在再来看看：

###模块化响应式

模块化的响应式理念来自于 Futurice’s repo，说的是在 Activity 和 Fragment 里使用多种模型来处理业务数据和网络访问

比如，UserLocationModel 是提供物理位置的模型类，在 Activity 获得设备坐标然后发给相应的 Fragment

![](../images/user_location_model.png)

相应的代码：

```
public class UserLocationModel {

  private PublishSubject<LatLng> subject = PublishSubject.create();
  
  public void setLocation(LatLng latLng) {
    subject.onNext(latLng);
  }

  public Observable<LatLng> getUserLocation() {
    return subject;
  }
}
```
很好的完成了模块化，将来也更容易进行扩展，将来还可以还细分：

![](../images/modularity.png)

看到这，你会发现也没有那么难嘛，这种模式已经在我的许多项目中进行了充分的测试，在这方面没有比 RxJava 做的更好的了。

###小技巧

- 尽可能使用小巧的模型类便于测试
- 业务逻辑应该在模型中，而不是 `Activities/Fragment/Dialogs`
- Keep the threading logic in the Activities/Fragments/Dialogs because unit testing is a bit harder (but not)
- 使用 `Dagger` 或手工降低模块间的依赖

译自 [https://medium.com/@diolor/rxjava-as-event-bus-the-right-way-10a36bdd49ba](https://medium.com/@diolor/rxjava-as-event-bus-the-right-way-10a36bdd49ba)
