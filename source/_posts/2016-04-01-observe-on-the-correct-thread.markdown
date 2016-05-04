---
layout: post
title: "RxJava 中的线程"
date: 2016-04-01 19:29:08 +0800
comments: true
categories: RxJava
---

最近有不少人开始使用 RxJava，但令人困扰的是：我们的业务逻辑链及其操作应该运行在哪个线程中。首先需要区分 `.subsribeOn()` 和 `.observeOn()`:

- `.subsribeOn()` 决定了 `Observable` 的操作所在的线程
- `.observeOn()` 决定了在哪个线程接收 `Observable` 的数据
- 需要注意的是，默认地，`Observable` 调用链运行在调用 `.subscribe()` 的线程中

##例子

###1. Main thread / .subscribe() thread

在 Android 的 Activity 中的 onCreate() 方法中运行下面的代码，将会在 UI 线程中

```java
Observable.just(1,2,3)
  .subscribe();
```

![](https://cdn-images-1.medium.com/max/1600/1*YDdZ06fTiZUHun9yvBh10Q.png)

<!--more-->

###2. subscribeOn()

下面的代码即使在主线程运行，也会被切换到`.subsribeOn()`中使用的线程中

```
Observable.just(1,2,3)
  .subscribeOn(Schedulers.newThread())
  .subscribe();
```
![](https://cdn-images-1.medium.com/max/1600/1*fEmKjaIddA88GbQBPvjwNA.png)

###3. observeOn()

同样地，Observable 的创建会运行在调用`.subscribe()`的线程中，但调用 `.observeOn()`后的代码会运行在其指定的线程中

```
Observable.just(1,2,3)
  .observeOn(Schedulers.newThread())
  .subscribe();
```

![](https://cdn-images-1.medium.com/max/1600/1*dABPYQGANqqiF8BYhDQGwA.png)

###	4. Combined logic

组合一下上面的代码：

```
Observable.just(1,2,3)
  .subscribeOn(Schedulers.newThread())
  .observeOn(Schedulers.newThread())
  .subscribe();
```

![](https://cdn-images-1.medium.com/max/1600/1*_7eJddNwledE5A8tM_7KfA.png)

##提示与陷阱

###1. 运行在主线程中

```
Observable.just(1,2,3)
  .subscribeOn(Schedulers.newThread())
  .subscribe(/** 操作 UI 的逻辑 **//);
```

###	2. 运行在非主线程中

错误：

```
Observable.just(1,2,3)
  .subscribeOn(Schedulers.newThread())
  .observeOn(AndroidSchedulers.mainThread())
  .flatMap(/** 操作非 UI 的逻辑 **//)
  .subscribe();
```

正确：

```
Observable.just(1,2,3)
  .subscribeOn(Schedulers.newThread())
  .flatMap(/** 操作非 UI 的逻辑 **//)
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe();
```

**注意调用 `.observeOn` 的先后顺序**

第2段代码中，`.flatMap()`的代码运行在后台线程中，当然不会阻塞 UI 线程了，这借鉴了`AsyncTask`中在`.doInBackground()`运行业务逻辑，而不是运行在`.onPostExecute()`中

### 3. 首次 .subscribeOn() 优先

```
Observable.just(1,2,3)
  .subscribeOn(thread1)
  .subscribeOn(thread2)
  .subscribe();
```

如果多次调用 `subscribeOn()`，以首次为准

译自 [https://medium.com/@diolor/observe-in-the-correct-thread-1939bb9bb9d2](https://medium.com/@diolor/observe-in-the-correct-thread-1939bb9bb9d2)