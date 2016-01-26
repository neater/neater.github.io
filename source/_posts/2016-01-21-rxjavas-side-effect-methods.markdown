---
layout: post
title: "RxJava 具有副作用的方法（译）"
date: 2016-01-21 16:00:14 +0800
comments: true
categories: RxJava
---

RxJava 有许多方法可以把流中的数据项转化成需要的数据，它们是 RxJava 的核心，也是最吸引人的部分。与之对应的是，其它方法不更改流中的数据项，我称它们为副作用方法。

###副作用方法
虽然副作用方法不影响流，但它们会在一些特定的事件中被调用，从而响应这些事件。比如：当一个被订阅的流产生错误的时候，可以 doOnError() 会被调用，这时把一些调试代码写在 doOnError() 里，就能知道到底发生了什么错误。

<!--more-->

```java
someObservable
      .doOnError(new Action1() {
         @Override
         public void call(Throwable t) {
            // use this callback to clean up resources,
            // log the event or or report the
            // problem to the user
         }
      })
      //…
```

在 call() 方法的代码会在订阅者的 onError() 方法之前调用。除了异常事件，RxJava 提供了很多回调方法用来响应各种事件：


**副作用方法和事件的关系**

| **方法**        | **回调类型** | **方法（调用时机）** |
| ------------- |-------------|-------------|
| doOnSubscribe()     | 	Action0 | 订阅前
| doOnUnsubscribe()     | 	Action0 | 订阅者通过 subscription 取消订阅前
| doOnNext()     | Action1\<T> | 发送数据项前
| doOnCompleted()     | 	Action0 | 生产者发送所有的数据项前
| doOnError()     | 	Action1\<T> | 产生错误前
| doOnTerminate()     | 	Action0 | 出错或者完成发送之前 
| finallyDo()     | 	Action0 | 出错或者完成发送之后 
| doOnEach()     | 	Action1\<Notification<T>> |每个数据项发送前、完成所有发送前、发生错误前，发送中的通知对象包含事件的类型。
| doOnRequest()     | 	Action1\<Long> |  订阅者请求更多的数据项

范型 T 在 doOnNext() 调用时表示数据的类型，在 doOnError() 调用时则是异常的类型。回调类型则有 Action0 和 Action1，说明这些回调方法没有返回值，没有或者只有一个参数。正因为没有返回值，它们不会在需要改变数据项时调用，它们适合在产生副作用的地方，比如保存数据到本地、清空状态值等等。

需要注意的是，这些副作用方法（doOnNext(), doOnCompleted() and so on）会返回 Observable，可以保持方法的链式调用。

###有什么用
利用它们不更改流中的数据的特性，可以满足下面三种需求：

- doOnNext(): 调试
- doOnError() in flatMap(): 错误处理
- doOnNext(): 保存/缓存数据

来看看详细用法。
###doOnNext(): 调试
在使用 RxJava 的过程中，Observable 有时并不会像预期运行，尤其是刚上手的那段时间，这时特别想知道发生了什么。还有当你通过某些方便的转换方法，是不是想看看每个数据项转换后的结果。

我开始接触 RxJava 的时候只有一些 Java 流的经验，想必你也一样吧。使用内置的方法可以把流中的数据转换成另一种类型，如果转换过程出现问题了呢？

Java 8 Streaming 的 **peek()** 方法可以调试这个问题，当我开始使用 RxJava 时自然想用与之类似的方法，幸运的是有，事实上，它不仅如此！

可以在任何流发送的过程中调用 **doOnNext()** 查看流中的每项数据，代码如下：

```java
Observable someObservable = Observable
            .from(Arrays.asList(new Integer[]{2, 3, 5, 7, 11}))
            .doOnNext(System.out::println)
            .filter(prime -> prime % 2 == 0)
            .doOnNext(System.out::println)
            .count()
            .doOnNext(System.out::println)
            .map(number -> String.format(“Contains %d elements”, number));

Subscription subscription = o.subscribe(
            System.out::println,
            System.out::println,
            () -> System.out.println(“Completed!”));
```

运行结果如下：

```
2
3
3
5
5
7
7
11
11
4
Contains 4 elements
Completed!
```
这种方法可以查看数据的值，当流的数据不符合预期的时候特别有用。

**doOnError()** 和 **doOnCompleted()** 也能用来调试。

如果你在开发 Android App 时使用了 RxJava，可以使用 [Frodo](https://github.com/android10/frodo)（使用了注解）调试 Observables 和 Subscribers。具体用法可以参考 Fernando Ceja 的[博客](http://fernandocejas.com/2015/11/05/debugging-rxjava-on-android/)

尽管 **doOnNext()** 和 **doOnError()** 的使用会增加你的日志，并且会拖慢你的系统。:-)
当需要改变系统状态时，还是适用的，接着来看。

###flatMap() 中调用 doOnError()

因为支持 RxJava，Retrofit 特别适合在网络请求时使用，而在数据请求链中会经常使用到 flatMap() 方法，是因为网络调用容易出错，特别是在移动设备上，但是出错时又不想停止数据调用链，这时唯一可用的就只有 **onError()方法了**。不过别忘了还有 **flatMap()**，在它里面调用 **doOnError**修改 UI 状态，还能进行出错后的数据处理，代码如下：

```java
flatMap(id -> service.getPost()
       .doOnError(t -> {
          // report problem to UI
       })
       .onErrorResumeNext(Observable.empty())
)
```

这种用法特别适合用来查看远程数据对 UI 状态的影响。

###使用 doOnNext() 保存/缓存网络数据
如果在链式调用过程中有网络调用，可以使用 doOnNext() 把获得的数据存入数据库或者缓存起来。只需要几行简单的代码：

```java

// getOrderById is getting a fresh order
// from the net and returns an observable of orders
// Observable<Order> getOrderById(long id) {…}

Observable.from(aListWithIds)
         .flatMap(id -> getOrderById(id)
         .doOnNext(order -> cacheOrder(order))
         // carry on with more processing
```

Daniel Lew 的 [accessing multiple sources](http://blog.danlew.net/2015/06/22/loading-data-from-multiple-sources-with-rxjava/) 里，详细介绍了这种用法。

### 小结

正如上面所说，副作用方法有多种用途，虽然不能影响流中数据，但却能改变周边的环境状态。这让在某个时刻记录数据流变得很简单，更不用说把网络调用获得的数据存入数据库了。

接下来的博客，会进一步介绍 RxJava hook，敬请期待！

译自 [http://www.grokkingandroid.com/rxjavas-side-effect-methods/](http://www.grokkingandroid.com/rxjavas-side-effect-methods/)