---
layout: post
title: "RxJava 在 Android 开发中的初次尝试（下）"
date: 2015-09-23 15:06:06 +0800
comments: true
categories: RxJava
---
这次我们来完成⎾RxJava 在 Android 开发中的初次尝试⏌的下半部分，在上次代码的基础上，用 RxJava 实现网络请求、多线程、AsyncTask。

在点击`Submit`按钮后，发起一次网络请求，可以用 Retrofit，对 RxJava 支持的很好，不过我不打算用他，我想用 RxJava 来模拟网络请求实现进度条的显示功能。

```java
    private void submit(InputValidation inputValidation) {
        request(inputValidation)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .doOnSubscribe(() -> enableWidgets(false))
                .doOnCompleted(() -> enableWidgets(true))
                .doOnError(throwable1 -> enableWidgets(true))
                .subscribe(progressBar::setProgress
                );
    }I
```
不过要注意的是，网络等需要多线程的代码需要调用 `subscribeOn(Schedulers.io())`，不然会报 ANR 错误，还有要对界面进行操作需要调用 `observeOn(AndroidSchedulers.mainThread())`，`doOn*`这一类方法会在更阶段进行回调，我把一些状态设置和显示的代码写在里面

```
    private void enableWidgets(boolean enable) {

        userName.setEnabled(enable);
        password.setEnabled(enable);
        confirmedPassword.setEnabled(enable);

        int visible = enable ? View.INVISIBLE : View.VISIBLE;
        progressBar.setVisibility(visible);

        String text = enable ? "end submit" : "begin submit";
        Snackbar.make(submitButton, text, Snackbar.LENGTH_SHORT).show();
    }
```
模拟网络请求的代码：

```
    private Observable<Integer> request(InputValidation inputValidation) {
        return Observable.create(subscriber -> {
            try {
                for (int i = 1; i < 101; i++) {
                    subscriber.onNext(i);
                    Thread.currentThread().sleep(30);
                }
                subscriber.onCompleted();
            } catch (InterruptedException e) {
                e.printStackTrace();
                subscriber.onError(e);
            }
        });
    }
```
把用户输入的代码移到 InputValidation.java 里，这里不贴代码了。最终效果：

![submit](/images/progress.gif)




