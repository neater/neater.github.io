---
layout: post
title: "ReactiveCocoa and Swift"
date: 2014-09-24 15:21:40 +0800
comments: true
categories: Reactivecocoa Swift
---

Swift可以和Objective-C相互调用，所以，在用Swift编写的App中，可以直接使用Reactivecocoa。

尽管如此，

### Signals in Swift
通过桥接，ReactiveCocoa的API从Objective-C的块过渡到了Swift的闭包，来看一个例子，下面的Objective-C代码subscribe了UITextfield的`rac_textSignal`，用来查看当前的length。
```
[self.searchTextField.rac_textSignal subscribeNext:^(id x) {
  NSString *text = (NSString *)x;
  NSLog(text);
}];
```
当在这个UITextfield中依次输入0 1 2 3 4 5时，Console中显示
```
0
1
2
3
4
5
```
当然了，为了避免显示转换，可以把上面的Block签名中的参数类型，从`id`改为`NSString`:
```
[self.searchTextField.rac_textSignal subscribeNext:^(NSString *text) {
  NSLog(text);
}];
```
与之对应的Swift代码如下：
```
searchTextField.rac_textSignal().subscribeNext {
  (next:AnyObject!) -> () in
  if let text = next as? String {
    println(countElements(text))
  }
}
```
不幸的是，当Swift和Objective-C进行调用的时候，会经常用到`as`操作符，从而变得更复杂了，我们不能进行显示转换，Swift生硬地插入了必须的类型为`(AnyObject!) -> ()`的函数。


也可以使用`if-let`条件语句进行简化，因为`AnyObject`在当前情况下可以视为字符串。
```
searchTextField.rac_textSignal().subscribeNext {
  (next:AnyObject!) -> () in
  let text = next as String
  println(countElements(text))
}
```
但是，这仍然还是要比Objective-C的版本复杂。
还好有解决这个问题的方案！针对`RACSignal`进行扩展，把传入函数参数类型进行必须的转换:
```
extension RACSignal {  
  func subscribeNextAs<T>(nextClosure:(T) -> ()) -> () {
    self.subscribeNext {
      (next: AnyObject!) -> () in
      let nextAsT = next as T
      nextClosure(nextAsT)
    }
  }
}
```
上面的代码使用泛型进行类型转换，参数T的类型通过实际传入的参数进行推断，这引出了一些有趣的语法
```
searchTextField.rac_textSignal().subscribeNextAs {
  (text:String) -> () in
  println(countElements(text))
}
```
好多了！

### Bindings
ReactiveCocoa有一些宏用来绑定属性到signals，下面的代码，把viewModel的`searchText`属性绑定到一个UITextfield的`rac_textSignal`属性，当UITextfield中的内容更新了，这个signal忽略当前值，把新的值更新到viewModel的searchText
```
RAC(self.viewModel, searchText) = self.searchTextField.rac_textSignal;
```
Swift不支持宏，当然不能使用`RAC`了。
下面的struct和operator替代了RAC macro:
```
// a struct that replaces the RAC macro
struct RAC  {
  var target : NSObject!
  var keyPath : String!
  var nilValue : AnyObject!
  
  init(_ target: NSObject!, _ keyPath: String, nilValue: AnyObject? = nil) {
    self.target = target
    self.keyPath = keyPath
    self.nilValue = nilValue
  }
  
  func assignSignal(signal : RACSignal) {
    signal.setKeyPath(self.keyPath, onObject: self.target, nilValue: self.nilValue)
  }
}

operator infix ~> {}
@infix func ~> (signal: RACSignal, rac: RAC) {
  rac.assignSignal(signal)
}
```

```
searchTextField.rac_textSignal() ~> RAC(viewModel, "searchText")
```
重写`RACObserve`宏显得容易些
```
func RACObserve(target: NSObject!, keyPath: String) -> RACSignal  {
  return target.rac_valuesForKeyPath(keyPath, observer: target)
}
```
### Cool Stuff!
我喜欢把已有的App移植一点一点地移植到Swift，为了减少Flickr的API调用，当photo显示一秒后，再调用相应的API获得相应的元数据(评论、点赞数)
```
// a signal that emits events when visibility changes
let visibleStateChanged = RACObserve(self, "isVisible").skip(1)

// filtered into visible and hidden signals
let visibleSignal = visibleStateChanged.filter { $0.boolValue }
let hiddenSignal = visibleStateChanged.filter { !$0.boolValue }

// a signal that emits when an item has been visible for 1 second
let fetchMetadata = visibleSignal.delay(1).takeUntil(hiddenSignal)

fetchMetadata.subscribeNext {
  // fetch the meta-data
}
```
上面的代码监听viewModel的`isVisible`属性，当photo显示或隐藏时产生信号来发出相应的事件，通过组合的方式，延时一秒后产生一个信号来获得元数据。酷！

### Conclusions
把app从Objectiv-C移植到Swift是很有趣的，有些地方显得有些让人困惑，但我认为，一般说来Swift的这种编程范式将来会变得越来越容易。

译自 [
MVVM, Swift and ReactiveCocoa - It's all good!
](http://www.scottlogic.com/blog/2014/07/24/mvvm-reactivecocoa-swift.html)
