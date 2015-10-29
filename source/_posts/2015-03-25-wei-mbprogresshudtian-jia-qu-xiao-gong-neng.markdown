---
layout: post
title: "为MBProgressHUD添加取消功能"
date: 2015-03-25 16:48:25 +0800
comments: true
categories: 
---

MBProgressHUD是一个显示HUD窗口的第三方类库，用于在执行一些后台任务时，在程序中显示一个表示进度的loading视图和两个可选的文本提示的HUD窗口。

在现实的需求中，可能存在这种情况：比如一个网络操作，在发送请求等待响应的过程中，我们会显示一个HUD窗口以显示一个loading框。但如果我们想在等待响应的过程中，在当前视图中取消这个网络请求，就没有相应的处理方式，MBProgressHUD没有为我们提供这样的交互操作。

简单看了下MBProgressHUD的源码，发现使用了NSOperationQueue来调度任务，便有了下面代码。

```objective-c

- requestData
{
   self.manager = [AFHTTPRequestOperationManager manager];
	UITapGestureRecognizer *tapRecognizer = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(cancelRequest)];
}

- (void)cancelRequest
{
    [self.manager.operationQueue cancelAllOperations];
}

```
