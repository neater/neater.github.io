---
layout: post
title: 如何过渡到 RxAndroid 1.0"
date: 2016-01-28 14:08:00 +0800
comments: true
categories: 
---

最近有许多人都在问我，操蛋的 [RxAndroid](https://github.com/ReactiveX/RxAndroid) 是怎么了？说真的，它确实是有点乱，这也是它最近被重构的主要原因，详细请看[这里](https://github.com/ReactiveX/RxAndroid/issues/172)，主要是说：
> 我建议从头开始，改善它的模块化问题，提高可重用性和可组合性。

虽然已经完成了这个目标，但我们过渡到新版本时，出现了新的问题：
原来的 API 都跑哪去了，我该如何才能通过编译？

###RxAndroid
`AndroidSchedulers` 的代码全部保留，仅仅改变了一些方法的签名。

###改动

`WidgetObservable` 和 `ViewObservable` 经过完善后，移到了 [RxBinding](https://github.com/JakeWharton/RxBinding)。

`LifecycleObservable` 移到了 [RxLifecycle](https://github.com/trello/RxLifecycle)，不过它被大量的重构了，你可能需要好查看一下它的更新日志。

还有 `ContentObservable.fromSharedPreferencesChanges()` 经过完善后，移到了 [rx-preferences](https://github.com/f2prateek/rx-preferences)。

###孤立
`ContentObservable` 其余部分还没有人来继续这项开源项目。

###移除
`AppObservable` 及其一系列绑定方法全部删除，主要是下面几个原因：

1. 原本想自动取消订阅，但只在 Activity 和 Fragment 暂停时实现了，数据源最终也不会取消订阅
2. 原来想防止 App 在暂停的时候收到通知，但在 `HandlerScheduler` 里出现了不容易察觉的逻辑问题
3. 自动调用 `observeOn(AndroidSchedulers.mainThread())`

换句话说，它非但没有实现当初的设计目标，而且还过于保守，副作用的问题也让人不快。

使用它时，请注意：

1. 通过 `Subscription`（或者使用 RxLifecycle）取消订阅
2. 是否真的需要使用 `observeOn(AndroidSchedulers.mainThread())`

译自 [http://blog.danlew.net/2015/09/01/how-to-upgrade-to-rxandroid-10/](http://blog.danlew.net/2015/09/01/how-to-upgrade-to-rxandroid-10/)
