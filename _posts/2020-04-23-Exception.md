---
layout:     post
title:      捕获异常
subtitle:	 
date:       2020-04-23
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - 捕获异常
---

# 异常捕获

OC有和Java、C++类似的捕获异常的语法。通过使用`NSException`, `NSError`或者自定义使用classes,你可以给你的程序捕获强壮的异常捕获，这个章节主要提供简单的异常捕获语法，更多的详情请参考[Exception Programming Topics.](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Exceptions/Exceptions.html#//apple_ref/doc/uid/10000012i)

##允许异常捕获（Enabling Exception-Handling）

使用GNU Compiler Collection (GCC) 3.3及更高版本，Objective-C为异常处理提供了语言级别的支持。要启用对这些特性的支持，请使用GNU编译器集合(GCC) 3.3版及更高版本的-fobjc-exceptions开关。(注意，这个开关只在OS X v10.3及以后版本中使应用程序可运行，因为在软件的早期版本中没有运行时对异常处理和同步的支持。)

## 捕获异常(Exception Handling)

异常是中断正常程序执行流的一种特殊情况。产生异常(通常称为引发或抛出异常)的原因有很多，有硬件也有软件。示例包括算术错误，如除数为0、下溢或溢出、调用未定义的指令(如试图调用未实现的方法)，以及试图在界限之外访问集合元素。

Objective-C异常支持包含四个编译器指令:`@try`， `@catch`， `@throw`和`@finally`:

* 可能抛出异常的代码包含在`@try{}`块中。
`@catch{}`块包含针对`@try{}`块中抛出的异常的异常处理逻辑。您可以使用多个@catch{}块来捕获不同类型的异常。(有关代码示例，请参见[Catching Different Types of Exception](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocExceptionHandling.html#//apple_ref/doc/uid/TP30001163-CH13-171873)。) 
* 你使用`@throw`指令来抛出一个异常，它本质上是一个Objective-C对象。您通常使用NSException对象，但不需要这样做。
* `@finally{}`块包含无论是否抛出异常都必须执行的代码。
这个例子描述了一个简单的异常处理算法:

```
Cup *cup = [[Cup alloc] init];
 
@try {
    [cup fill];
}
@catch (NSException *exception) {
    NSLog(@"main: Caught %@: %@", [exception name], [exception reason]);
}
@finally {
    [cup release];
}
```


## 捕获不同类型的异常（Catching Different Types of Exception）

要捕获在@try{}块中抛出的异常，请在@try{}块后面使用一个或多个@catch{}块。@catch{}块应该按照从最特定到最不特定的顺序排序。通过这种方式，可以将异常处理裁剪为组，如清单10-1所示。

```
@try {
    ...
}
@catch (CustomException *ce) {   // 1
    ...
}
@catch (NSException *ne) {       // 2
    // Perform processing necessary at this level.
    ...
 
}
@catch (id ue) {
    ...
}
@finally {                       // 3
    // Perform processing necessary whether an exception occurred or not.
    ...
}
```

下面的列表描述了编号的代码行:

1. 捕获最特定的异常类型。
2. 捕获更一般的异常类型。
3. 执行必须始终执行的任何清理处理，不管是否抛出异常

## 抛出异常（Throwing Exceptions）
要抛出异常，您必须使用适当的信息来实例化一个对象，例如异常名称和抛出它的原因

```
NSException *exception = [NSException exceptionWithName: @"HotTeaException"
                                                 reason: @"The tea is too hot"
                                               userInfo: nil];
@throw exception;
```

>重要的：在许多场景下使用异常捕获是非常司空见惯的。举个例子，你可能抛出一个异常信号当你的例程无法正常执行的的时候，比如文件找不到或者传入的数据错误。在OC中异常是非常多的。在流控制中不需要使用`exception`，或者只表示错误。相反你应该使用`return`返回方法或者函数指出遇到什么错误，并且提供问题足够的信息。更多详细的信息请参考[Error Handling Programming Guide.](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ErrorHandlingCocoa/ErrorHandling/ErrorHandling.html#//apple_ref/doc/uid/TP40001806)

在`@catch`的闭包中，你可以重复的抛出异常通过使用`@throw`指令而不提供参数。这种情况下不使用参数使得对你的代码更有可读性

您不限于抛出`NSException`对象。你可以抛出任何Objective-C对象作为异常对象。`NSException`类提供了一些方法来帮助异常处理，但是如果您愿意，您可以实现自己的方法。您还可以子类化`NSException`来实现特殊类型的异常，例如文件系统异常或通信异常。

参考  
[Exception Handling](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocExceptionHandling.html)