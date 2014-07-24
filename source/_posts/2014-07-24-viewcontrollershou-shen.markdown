---
layout: post
title: "ViewController瘦身"
date: 2014-07-24 16:45:58 +0800
comments: true
categories: 
---
面对日益臃肿的ViewController，测试、重构、增加新的需求将越来越困难，怎么破？
#### 1.抽取view logic
通常我们会在ViewController里加载并控制View（通过Xib或者loadView方法），这样VC就会关心View的内部细节，如layout、Animation、如何填充、响应子视图的事件等，使VC更难以理解，增加了和View的耦合。


#####为Views创建Class
```objective-c
#import <UIKit/UIKit.h>
@class JPLoginViewModel;
 
@interface JPLoginView : UIView
 
@property (strong, nonatomic) JPLoginViewModel *viewModel;
 
- (void)showAppearanceAnimation;
- (void)showNoConnectionFeedback:(BOOL)shouldShow;
 
@end
 
@interface JPLoginViewModel : NSObject
 
@property (strong, nonatomic) NSString *userName;
@property (strong, nonatomic) NSString *password;
 
@end
```

