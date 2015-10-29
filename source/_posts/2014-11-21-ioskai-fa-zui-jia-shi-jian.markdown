---
layout: post
title: "iOS开发最佳实践"
date: 2014-11-21 11:49:45 +0800
comments: true
categories: 
---

和软件一样，这篇文章离不开大家的贡献，哪怕打开一个Issue或者发一个PR。


如果对Android感兴趣，请看这里 [Android开发最佳实践](https://github.com/futurice/android-best-practices)

##起因
遗憾地是，iOS开发的门槛挺高的，Swift和Objective-C都没有在iOS/Mac以外的系统广泛使用，而且有自己的命名方式，想让代码跑在苹果的设备上并不容易（译者注：各种编译和开发环境问题），不论你刚接触Cocoa还是对iOS开发最佳实践感兴趣，都能在这里找到答案。下面列出的只是一些建议，如果你有更好的实践可以进行相应的替换。

##开始
###Xcode
Xcode是绝大多数iOS开发者选择的IDE，并且是苹果官方唯一支持的。当然也有其它的，AppCode可以说其中最著名的，不过如果你还不是一个iOS开发的老手，还是使用Xcode吧，不管怎么样，它在可用性方面还是不错的。

可以在Mac App Store下载最新的SDK和模拟器，在Preferences > Downloads中可以下载更多的文档。

###项目设置

刚开始开发iOS项目有个很常见的问题，纯代码?Storybard?还是XIB，用哪一个来开发UI，Both are known to occasionally result in working software。但是，还要从下面几个方面来考虑：

####纯代码？
- Xcode5中，通用型App使用不同的Storyboard区分iPhone和iPad，这导致了许多不必要的重复，Xcode6引入了Size Classes来解决这个问题。
- Xcode5中，Storyboard不支持自定义字体和UI元素， but will have a generic appearance instead，Xcode6再次解决了它。
- Storyboard使用了复杂的XML，在版本控制中，更容易产生冲突。

####Storyboards?
- or the less technically inclined, Storyboards在操作方式上做的最好，比如，直接修改颜色或者布局约束，当然这需要一些时间来学习。

###Ignores
把项目加入版本控制的时候，最好也加入.gitignore文件，这样不会把不需要的文件（用户自己的配置和临时文件等）加入。幸运地是，GitHub已经为我们提供了[Objective-C](https://github.com/github/gitignore/blob/master/Objective-C.gitignore)和[Swift](https://github.com/github/gitignore/blob/master/Swift.gitignore)的版本(译者注：还有个好用的网站，[gitignore.io](https://www.gitignore.io/))

<!--more-->

###CocoaPods
在管理第三方库方面，CocoaPods无疑首当其冲，安装的命令：
```
sudo gem install cocoapods
```
进入项目主目录，运行
```
pod init
```
这会生成Podfile文件，统一进行管理依赖，再运行
```
pod install
```
主工程和这些第三方库组成的库会放置在Workspace中，这以后，要通过.xcworkspace而不是.xcproject来打开项目，否则会产生编译错误。

###工程结构
随着源文件日益增多，项目目录越发凌乱，有必要将它们按用途分类，下面是一个实例：
```
├─ Models
├─ Views
├─ Controllers
├─ Stores
├─ Helpers
```
First, create them as groups (little yellow "folders") within the group with your project's name in Xcode's Project Navigator. Then, for each of the groups, link them to an actual directory in your project path by opening their File Inspector on the right, hitting the little gray folder icon, and creating a new subfolder with the name of the group in your project directory.

##网络
###传统方式：使用自定义回调Block
```objective-c
// GigStore.h

typedef void (^FetchGigsBlock)(NSArray *gigs, NSError *error);

- (void)fetchGigsForArtist:(Artist *)artist completion:(FetchGigsBlock)completion


// GigsViewController.m

[[GigStore sharedStore] fetchGigsForArtist:artist completion:^(NSArray *gigs, NSError *error) {
    if (!error) {
        // Do something with gigs
    }
    else {
        // :(
    }
];
```

This works, but can quickly lead to callback hell if you need to chain multiple requests. In that case, have a look at ReactiveCocoa (RAC). It's a versatile and multi-purpose library that can change the way people write entire apps, but you can also use it sparingly where it fits the task.



There are good introductions to the concept of RAC (and FRP in general) on Teehan+Lax and NSHipster.
###Reactive方式: 使用RAC signals
```
// GigStore.h

- (RACSignal *)gigsForArtist:(Artist *)artist;


// GigsViewController.m

[[GigStore sharedStore] gigsForArtist:artist]
    subscribeNext:^(NSArray *gigs) {
        // Do something with gigs
    } error:^(NSError *error) {
        // :(
    }
];
```
通过将gig的signal和其它signal组合，在显示之前，我们可以转换或者过滤掉gips。

###结构化
Pragma marks能够很好地将方法进行分组归类，尤其在View controller文件中，下面是一些常用的分组结构：
	
```
#import "SomeModel.h"
#import "SomeView.h"
#import "SomeController.h"
#import "SomeStore.h"
#import "SomeHelper.h"
#import <SomeExternalLibrary/SomeExternalLibraryHeader.h>

static NSString * const XYZFooStringConstant = @"FoobarConstant";
static CGFloat const XYZFooFloatConstant = 1234.5;

@interface XYZFooViewController () <XYZBarDelegate>

@property (nonatomic, copy)

@end

@implementation XYZFooViewController

#pragma mark - Lifecycle

- (instancetype)initWithFoo:(Foo *)foo;
- (void)dealloc;

#pragma mark - View Lifecycle

- (void)viewDidLoad;
- (void)viewWillAppear:(BOOL)animated;

#pragma mark - Auto Layout

- (void)makeViewConstraints;

#pragma mark - Public Interface

- (void)startFooing;
- (void)stopFooing;

#pragma mark - User Interaction

- (void)foobarButtonTapped;

#pragma mark - XYZFoobarDelegate

- (void)foobar:(Foobar *)foobar didSomethingWithFoo:(Foo *)foo;

#pragma mark - Internal Helpers

- (NSString *)displayNameForFoo:(Foo *)foo;

@end
```
最重要的是在工程中的所有文件保持上面的一致性。
##部署

iOS上部署并不方便，话虽如此, 了解下面几点会很有帮助的。

每当你想将开发的App安装的真实设备而非模拟器时，都需要用苹果颁发的证书对其签名。每个证书都对应着一对公钥/私钥，私钥存储在Mac中的钥匙链中，证书有2种形式：

- 开发证书：团队中的每个开发者都有属于自己的，根据需要生成。在开发调试时需要这种证书，最好由我们手动进行选择设置，而不是让Xcode中的"Fix issue" 来帮我们解决问题。
- 发布证书：可以有多个，但最好每个组织保持一个，通过某种方式共享私钥，这种证书用于提前到App Store或者企业内部的发布系统。

除了证书，还有Provisioning配置文件，用于连接设备和证书，也有开发和发布2种形式：
- 开发provisioning：包含所有被受权安装的设备列表，该文件也能对应多个开发证书，意味着多个开发者可以使用它，可以将它用于专有App，但最常用的用途是将它以wildcard的形式共享于多个App，当然App ID必须以通配符*结尾。
- 分发provisioning：根据用途有3种发布方式，每种provisioning都对应一个发布证书，证书如果过期，它也将无效。
	- Ad-Hoc: 如同开发provisioning，它有个可安装设备列表的白名单，这种provisioning常用于Beta测试（TestFlight）。由于TestFlight被苹果收购，2014年底这种方式可能会有所变化。
	- App Store: 该provisioning没有设备列表，任何人都能在App Store上安装它，所有在App Store上发布的应用都要用到它。
	- Enterprise: 它也没有设备白名单，任何访问企业内部发布网站的人都能下载安装它，该provisioning仅适用于企业开发账号。


要想同步所有证书和provisioning，需要在Xcode中打开Accounts标签，添加开发者账号，然后双击Team名称，底部将会出现刷新按钮，但有时需要重启Xcde才会生效。

##More Ideas
- [https://github.com/vsouza/awesome-ios](https://github.com/vsouza/awesome-ios)
- Update for Xcode 6
	- No automatic precompiled header
- Pod usage: pod install vs pod update
- iTunes Connect etc.
- 3x assets, iPhone 6 screen sizes explained
- Add @interface and constants to VC Structure
- Add list of suggested compiler warnings
- Ask IT about automated Jenkins build machine
- Add section on Testing
- Add section on Debugging, e.g. exception breakpoints
- Add "proven don'ts

译自：[iOS Good Practices](https://github.com/futurice/ios-good-practices)