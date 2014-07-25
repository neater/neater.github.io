---
layout: post
title: "ViewController瘦身之 抽取View Logic"
date: 2014-07-24 16:45:58 +0800
comments: true
categories: 
---
面对日益臃肿的ViewController，测试、重构、增加新的需求将越来越困难，怎么破？
#### 1.抽取view logic
通常我们会在ViewController里加载并控制View（通过Xib或者loadView方法），这样VC就会关心View的内部细节，如layout、Animation、如何填充、响应子视图的事件等，使VC更难以理解，而且增加了和View的耦合。


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
####为异步操作使用Block
View有许多操作都是异步的，比如显示动画、显示需要客户确认，这时应该使用Block，这样可以增加可读性，使代码更紧凑。

应该将不同的操作分离成Block。

```
typedef BOOL(^JPLoginViewConfirmationCompletion)();

@interface JPLoginView : UIView
//...
- (void)showUserConfirmationWithCompletionBlock:(JPLoginViewConfirmationCompletion)completion;

@end
```
这种策略有利于测试，可以创建一对测试View，仅仅调用block，而不用为两个View分别模拟测试数据。

####Datasources置于View中而不是ViewController
众所周之，VC通常实现Datasources，Apple的示例代码也是这么做的，但是仔细想想，当把UITableView的内部View改成UILabel或者UITextfield时，不可避免地要修改VC，无疑增加了耦合。

另外，如果View Model所需要的数据已经由专门的Class提供，为什么还要VC充当这个角色？TableView只需要关心视图的细节。

View所要做的仅仅是实例化分离出来的Datasources，并且将View Model赋值给它。将Cell的创建和配置信息封装起来，增加了可读性；分离Datasoures，方便进行单独测试，比如：number of cells、number of sections。

####结论
有些人可能认为，这些不符合Apple的标准，但Apple Guideline并没有明确禁止把View从View Controller分离出来，也没有说只能在VC中定义IBOutlet，Guideline适用的场景是在比较小的系统中，当应用场景变的越来越大、越来越复杂，那就必须分离逻辑，好处是增加了可读性和可测试性。

当然，增加额外的层会影响性能，但考虑到这样做所带来的好处，比如更易读、更快的查找，这点性能上的损失是绝对值得的