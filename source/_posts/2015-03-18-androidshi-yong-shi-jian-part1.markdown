---
layout: post
title: "Android实用实践Part1: 架构"
date: 2015-03-18 16:48:26 +0800
comments: true
categories: Android
---


本文是这一系列的开篇，主要涉及一些基础环境设置，目的是能开发出及扩展、及维护和易测试的项目，而本系列将会涵盖一些模式和库，它们会或多或少改善让Android开发者的体验。

#场景

本篇使用的Android项目救命，是个电影类别的原型，可以标记是否已经观看，影片的信息可以通过themoviedb提供的API获得，[Apiary](http://docs.themoviedb.apiary.io/)上有相关的文档，这个工程基于MVP模式，遵循了Material Design，如转场、结构、动画、动画等。

Github上有示例代码，并配有视频。

![](http://res.cloudinary.com/dttcwxrjo/image/upload/v1422988952/1_ntqfre.png)

#架构:
<!--more-->
正如前面所说，本项目基于的Model View Presenter是Model View Controller的变种。MVP试图抽取表示层中的逻辑，因为Android的Framework很容易把两者混合在数据层中，如Adapters或者CursorLoaders。

这种架构在修改视图时，不用修改逻辑层和数据层，这有既利于逻辑的重用又方便数据源的修改，比如将它从数据库更改为REST API。

##概观

整个结构分为下面3层：

- **表示层**
- **模型层**
- **领域层**

![](http://res.cloudinary.com/dttcwxrjo/image/upload/v1422988949/0B62SZ3WRM2R2eGczcWh3MERkRGc-e1422883852292_hfs4r6.png)

###表示层

表示层负责显示图形界面并为提供要显示的内容。

###模型层

表示层通常和领域逻辑联系，而模型层只负责提供数据，它可以连接数据库、REST API或者其它形式的持久化方式。

该层也实现了应用的实体，比如用来表示一部电影，一个类别等等。

###领域层

领域层完全独立于表示层，应用的业务逻辑都归类在这里。

##实现

下图显示了一个Android项目的目录结构（虚拟），模型层和领域层置于两个模块（module）中，而表示层出现在了app模块中，其它的模块包含了一些公用库和工具。
ps：app模块是Android工程模块。

![](http://res.cloudinary.com/dttcwxrjo/image/upload/v1422988944/0B62SZ3WRM2R2elJfT0JiZnlTcE01_ospwxi.png)

###领域模块

领域模块包含了应用的业务逻辑，比如用户用例和一些实现。并且完全独立于Android的框架。它依赖于模型层和公共模块。
比如：从影片各个分类的排名榜中，找出是受欢迎的类别；对模型层提供的信息进行统计。

```java
dependencies {
    compile project (':common')
    compile project (':model')
}
```

###模型模块

负责管理数据（增、删、改、查）。在目前的1.0版本中，只有一些电影信息的API。它还实现了实体，比如用TvMovie类表示一部电影。现在它只依赖common模块，用来发起API调用，我还使用了Retrofit（Square开发），随后的文章中我会再谈谈Retrofit。

```
dependencies {
    compile project(':common')
    compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```

###表示层模块

其实就是一个Andorid应用本身了，比如一些资源、逻辑等。

它常常和业务逻辑交互，比如显示在某个时间点上显示所有电影的排期、获得某部影片的信息。
Presenter和视图也出现在这里。
每个Activity, Fragment, Dialog都实现了在MVPView接口中，声明了电影相关的显示、隐藏、绘制方法，任何与电影相关的表示层组件（Activity, Fragment, Dialog），都必须实现该接口。

下图示例,PopularMoviesView视图接口,声明了用来显示电影列表的一些方法，而在MoviesActivity中实现了这些方法。

```
public interface PopularMoviesView extends MVPView {

    void showMovies (List<TvMovie> movieList);

    void showLoading ();

    void hideLoading ();

    void showError (String error);

    void hideError ();
}
```

MVP模式目的是让视图尽可能的简单，让presenter决定视图的行为。

```
public class MoviesActivity extends ActionBarActivity implements
    PopularMoviesView, ... {

    ...
    private PopularShowsPresenter popularShowsPresenter;
    private RecyclerView popularMoviesRecycler;
    private ProgressBar loadingProgressBar;
    private MoviesAdapter moviesAdapter;
    private TextView errorTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        ...
        popularShowsPresenter = new PopularShowsPresenterImpl(this);
        popularShowsPresenter.onCreate();
    }

    @Override
    protected void onStop() {

        super.onStop();
        popularShowsPresenter.onStop();
    }

    @Override
    public Context getContext() {

        return this;
    }

    @Override
    public void showMovies(List<TvMovie> movieList) {

        moviesAdapter = new MoviesAdapter(movieList);
        popularMoviesRecycler.setAdapter(moviesAdapter);
    }

    @Override
    public void showLoading() {

        loadingProgressBar.setVisibility(View.VISIBLE);
    }

    @Override
    public void hideLoading() {

        loadingProgressBar.setVisibility(View.GONE);
    }

    @Override
    public void showError(String error) {

        errorTextView.setVisibility(View.VISIBLE);
        errorTextView.setText(error);
    }

    @Override
    public void hideError() {

        errorTextView.setVisibility(View.GONE);
    }

    ...
}
```
下面的示例中，都是由presenters来执行，负责接收事件并管理视图的行为。

```
public class PopularShowsPresenterImpl implements PopularShowsPresenter {

    private final PopularMoviesView popularMoviesView;

    public PopularShowsPresenterImpl(PopularMoviesView popularMoviesView) {

        this.popularMoviesView = popularMoviesView;
    }

    @Override
    public void onCreate() {

        ...
        popularMoviesView.showLoading();

        Usecase getPopularShows = new GetMoviesUsecaseController(GetMoviesUsecase.TV_MOVIES);
        getPopularShows.execute();
    }

    @Override
    public void onStop() {

        ...
    }


    @Override
    public void onPopularMoviesReceived(PopularMoviesApiResponse popularMovies) {

        popularMoviesView.hideLoading();
        popularMoviesView.showMovies(popularMovies.getResults());
    }
}
```

###通讯


这里也选用了**消息队列**，它既让广播事件变得好用又能方便组件间的通讯，从上面的例子就能看得出来。
事件会发往一个队列，所有订阅这一事件的类都会接收到这一事件。这一系统降低了模块间的耦合。

我采用了**Square**开发的**Otto**库来实现这个系统

我声明了2个队列，一个用来和REST API通讯，另一个则向presentation层发送事件。

`REST_BUS`在另一个线程中处理事件，而`UI_BUS`则在默认线程中发送事件，也就是主线程。

```java
public class BusProvider {

    private static final Bus REST_BUS = new Bus(ThreadEnforcer.ANY);
    private static final Bus UI_BUS = new Bus();

    private BusProvider() {};

    public static Bus getRestBusInstance() {

        return REST_BUS;
    }

    public static Bus getUIBusInstance () {

        return UI_BUS;
    }
}
```
BusProvider类放置在common模块中，因为所有模块都要和它进行数据交换。

```
dependencies {
    compile 'com.squareup:otto:1.3.5'
}
```

最后来看一个例子，用户打开应用查看最受欢迎的影片列表。

在`onCreate`方法中，presenter**subscribe**了`UI_BUS`发出的事件，并且在`onStop`被调用时，进行对应的unsubscribe操作，然后presenter调用了GetMoviesUseCase。

```
    @Override
    public void onCreate() {

        BusProvider.getUIBusInstance().register(this);

        Usecase getPopularShows = new GetMoviesUsecaseController(GetMoviesUsecase.TV_MOVIES);
        getPopularShows.execute();
    }

    ...

    @Override
    public void onStop() {

        BusProvider.getUIBusInstance().unregister(this);
    }
}
```
要接收事件，presenter必须实现相应的方法，并且遵循下面的约定：

- 方法中的参数类型要和发送时事件的类型相同
- 使用@Subscribe注解

```
    @Subscribe
    @Override
    public void onPopularMoviesReceived(PopularMoviesApiResponse popularMovies) {

        popularMoviesView.hideLoading();
        popularMoviesView.showMovies(popularMovies.getResults());
}
```

###相关资源:

- [Architecting Android…The clean way?](http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/) - Fernando Cejas
- [Effective Android UI](https://github.com/pedrovgs/EffectiveAndroidUI) - Pedro Vicente Gómez Sanchez
- [Reactive programming and message buses for mobile](http://csaba.palfi.me/reactive-and-buses-for-mobile/) - Csaba Palfi
- [The clean architecturel](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) - Uncle Bob
- [MVP Android](http://antonioleiva.com/mvp-android/) - Antonio Leiva
 
    
译自：[A useful stack on android #1, architecture](http://saulmm.github.io/2015/02/02/A%20useful%20stack%20on%20android%20%231,%20architecture/)
    

    

    