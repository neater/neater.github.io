---
layout: post
title: "ViewController瘦身之 抽取View Logic"
date: 2014-07-24 16:45:58 +0800
comments: true
categories: 
---
面对日益臃肿的ViewController，测试、重构、增加新的需求将越来越困难，怎么破？

通常我们会在ViewController里加载并控制View（通过Xib或者loadView方法），这样VC就会关心View的内部细节，如layout、Animation、如何填充、响应子视图的事件等，使VC更难以理解，而且增加了和View的耦合。

VC主要负责与用户的交互，主要是控制内部View以及用户操作时产生的事件，而不是用来实现View，如果View使用了UITableView或者UICollectionView，那么VC不得不关心View的内部细节，就像一只冲出牢笼的野兽，变得难以驾驭，应该避免这样。

下面是如何设计View的实践：
###为Views创建Class
首先，为Views创建类，它封装了View的所有内部细节，并为VC提供抽象接口，用来提供改变View状态的方法或者获得其内部信息，这样一来，VC实现了与View的解耦，不用关心View的内部实现，只需要访问View提供的接口即可。
```
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
可以看出，这个.h文件没有IBOutlets和subViews的信息，因为他们都是内部细节；如果不这样的话，VC将能访问这样内部细节，设想将来你想使用UITextFields和UILabels，而不是现在的UITableView来显示login和password，修改VC将是无法避免的。

###使用View Model封闭View信息
上面的代码中的JPLoginViewModel封闭了View显示的细节，其中定义了2个字符串，而不在View中这样做，是因为View Model可以作为两者沟通的桥梁。
View Model一般定义在View中，并提供操作的接口，VC创建View Model，并给其赋值，当然也能再获得这样值，再把这个View Model的引用赋给View。通过KVO，VC可以实现对View属性的监控。

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    NSString *userName = @""; //Retreive it from your bussiness logic
    NSString *password = @""; //Retreive it from your bussiness logic
    
    
    JPLoginViewModel *viewModel = [[JPLoginViewModel alloc] init];
    viewModel.userName = userName;
    viewModel.password = password;
    
    self.loginView.viewModel = viewModel;
}

- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    
    [self.loginView.viewModel addObserver:self forKeyPath:@"password" options:kNilOptions context:nil];
}

- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    
    [self removeObserver:self forKeyPath:@"password"];
}
```
这样一来，VC就知道什么时候View什么时候修改了View Model的数据，而不用通过蛋疼的Delegate，同时与View实现了解耦，View Model的这种简单清晰的方式，实现了VC与View之间的数据同步。

###在内部接口中引用Subviews
当你需要引用内部的Subviews，例如UILable，UITableView等视图时，所有这些内部细节对任何其它类来说都应该是不可见的，应该把这些细节定义为View的内部属性。
```
#import "JPLoginView.h"
 
@interface JPLoginView()
@property (strong, nonatomic) IBOutlet UILabel *titleLabel;
@property (strong, nonatomic) IBOutlet UITableView *fieldsTableView;
@property (strong, nonatomic) IBOutlet UIButton *loginButton;
@property (strong, nonatomic) IBOutlet UIProgressView *connectionIssueProgressView;
 
@end
 
@implementation JPLoginView
//...
```
可以看到，将来如果想把UITableView改成UILabels和UITextFields，不用修改VC，改这个类就可以了，删除也是一样。
###使用XIBs定义Layout
定义View的最佳方式是使用XIBs，而不是在VC里面，这可能会让人不解，XIBs自动创建UIView和Layout，VC使用它初始化View。

定义界面时，XIBs可以减少代码量，而且表现得很出色。通过IBOutlets来连接View的内部属性和XIBs创建的Subviews，也就是常说的连线。在Interface Builder中，可以指定View的具体类型，通过这种方式，Framework自动创建具体类型的View。

![fdfs](http://jpellat.com/wp-content/uploads/2014/05/Screenshot-2014-05-18-01.02.28-300x129.png)
![fdfs](http://jpellat.com/wp-content/uploads/2014/05/connect-300x136.png)

###使用Delegate响应View的异步事件
当用户点击登陆按钮时，我们应该让VC马上响应这一事件，并不是说VC应该关心登陆的逻辑实现，下面谈谈如何让VC在这种情况下实现解耦。

Delegate应该定义在View的头文件中，因为是View产生了事件。可以为View定义一个Delegate的属性，而让VC为它赋值，方式可以是通过XIBs，或者在viewDidLoad方法中。
```
@protocol JPLoginViewDelegate;
 
@interface JPLoginView : UIView
 
@property (strong, nonatomic) JPLoginViewModel *viewModel;
@property (weak, nonatomic) id<JPLoginViewDelegate> delegate;
 
//methods
 
@end
 
@protocol JPLoginViewDelegate <NSObject>
 
- (void)didTapLoginAtLoginView:(JPLoginView *)loginView;
 
@end
```
Delegate的所有方法，可以集中在VC中，如果使用Block的话，viewDidLoad会充斥大量的代码，影响可读性，这方面，无疑Delegate做得更好。
也可以使用Delegate验证View内部的文本格式以及数据的有效性，VC可以调用框架的许多API，来验证Model中的相关数据。

###为异步操作使用Block
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

###Datasources置于View中而不是ViewController
众所周之，VC通常实现Datasources，Apple的示例代码也是这么做的，但是仔细想想，当把UITableView的内部View改成UILabel或者UITextfield时，不可避免地要修改VC，无疑增加了耦合。

另外，如果View Model所需要的数据已经由专门的Class提供，为什么还要VC充当这个角色？TableView只需要关心视图的细节。

View所要做的仅仅是实例化分离出来的Datasources，并且将View Model赋值给它。将Cell的创建和配置信息封装起来，增加了可读性；分离Datasoures，方便进行单独测试，比如：number of cells、number of sections。

####结论
有些人可能认为，这些不符合Apple的标准，但Apple Guideline并没有明确禁止把View从View Controller分离出来，也没有说只能在VC中定义IBOutlet，Guideline适用的场景是在比较小的系统中，当应用场景变的越来越大、越来越复杂，那就必须分离逻辑，好处是增加了可读性和可测试性。

当然，增加额外的层会影响性能，但考虑到这样做所带来的好处，比如更易读、更快的查找，这点性能上的损失是绝对值得的。		