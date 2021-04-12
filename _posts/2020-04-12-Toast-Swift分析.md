---
layout:     post
title:     Toast-Swift源码分析
subtitle:      
date:       2021-04-12
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog:      true
tags:
    - 视图
---

Toast-Swift的功能点

- 点击消失
- 将toast加入到队列Queue中
- 在顶部将toast显示3秒
- 显示一个标题的toast
- 显示一个图片的toast
- 显示标题、图片并且带回调的toast
- 显示自定义样式的toast
- 显示自定义视图的toast
- 显示自定义坐标的toast
- 显示加载的toast
- 隐藏toast
- 隐藏所有的toast
#### 架构(类图)


Toast一共有三个类

- Toast (一个是对UIView的扩展)
- ToastStyle(一个样式类)
- ToastManager(一个是管理类)

![97443F74-7BFF-41C8-BBB4-AE4F32CA0D0A.png](https://cdn.nlark.com/yuque/0/2020/png/368112/1602482232700-20f3f5a9-8ebb-43f4-911c-77b6ebb16940.png#align=left&display=inline&height=457&margin=%5Bobject%20Object%5D&name=97443F74-7BFF-41C8-BBB4-AE4F32CA0D0A.png&originHeight=457&originWidth=721&size=90803&status=done&style=none&width=721)
#### 核心逻辑（调用栈）


先来看一下如何使用 Toast 弹出一个提示
```
var style = ToastStyle()
style.messageFont = UIFont(name: "Zapfino", size: 14.0)!
style.messageColor = UIColor.red
style.messageAlignment = .center
style.backgroundColor = UIColor.yellow
self.navigationController?.view.makeToast("This is a piece of toast with a custom style", duration: 3.0, position: .bottom, style: style)
```
在看一下函数调用栈
![0136A846-2926-43CC-BA36-D161BBEC13D9.png](https://cdn.nlark.com/yuque/0/2020/png/368112/1602483046341-5c5eb8a5-2f5e-49ae-9fb5-07a3af994ecd.png#align=left&display=inline&height=91&margin=%5Bobject%20Object%5D&name=0136A846-2926-43CC-BA36-D161BBEC13D9.png&originHeight=91&originWidth=590&size=33946&status=done&style=none&width=590)
最终会调用UIView的showToast的方法
#### Toast（类分析）


**Toast**
**1、队列管理**


首先给UIView分类添加了一个queue的属性
```
private var queue: NSMutableArray {
        get {
            if let queue = objc_getAssociatedObject(self, &ToastKeys.queue) as? NSMutableArray {
                return queue
            } else {
                let queue = NSMutableArray()
                objc_setAssociatedObject(self, &ToastKeys.queue, queue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
                return queue
            }
        }
    }
```


在showToast的时候判断ToastManager.shared.isQueueEnabled属性是否打开以以及活跃的Toast的数量时候大于零，当同时都满足的时候，将位置和时间设置给对应的Toast，并将Toast加入都queue当中
```
func showToast(_ toast: UIView, duration: TimeInterval = ToastManager.shared.duration, point: CGPoint, completion: ((_ didTap: Bool) -> Void)? = nil) {
    objc_setAssociatedObject(toast, &ToastKeys.completion, ToastCompletionWrapper(completion), .OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    
    if ToastManager.shared.isQueueEnabled, activeToasts.count > 0 {
        objc_setAssociatedObject(toast, &ToastKeys.duration, NSNumber(value: duration), .OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        objc_setAssociatedObject(toast, &ToastKeys.point, NSValue(cgPoint: point), .OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        
        queue.add(toast)
    } else {
        showToast(toast, duration: duration, point: point)
    }
}
```


当上一个Toast隐藏的时候，从队列里取出数组里的第一个toast弹窗出来，并将其从数组里移除
```
private func hideToast(_ toast: UIView, fromTap: Bool) {
    if let timer = objc_getAssociatedObject(toast, &ToastKeys.timer) as? Timer {
        timer.invalidate()
    }
    
    UIView.animate(withDuration: ToastManager.shared.style.fadeDuration, delay: 0.0, options: [.curveEaseIn, .beginFromCurrentState], animations: {
        toast.alpha = 0.0
    }) { _ in
        toast.removeFromSuperview()
        self.activeToasts.remove(toast)
        
        if let wrapper = objc_getAssociatedObject(toast, &ToastKeys.completion) as? ToastCompletionWrapper, let completion = wrapper.completion {
            completion(fromTap)
        }
        
        if let nextToast = self.queue.firstObject as? UIView, let duration = objc_getAssociatedObject(nextToast, &ToastKeys.duration) as? NSNumber, let point = objc_getAssociatedObject(nextToast, &ToastKeys.point) as? NSValue {
            self.queue.removeObject(at: 0)
            self.showToast(nextToast, duration: duration.doubleValue, point: point.cgPointValue)
        }
    }
}
```


**2、点击消失**
点击Toast的时候会让自己消失，实际上是在自己上面添加一个点击手势
```
if ToastManager.shared.isTapToDismissEnabled {
    let recognizer = UITapGestureRecognizer(target: self, action: #selector(UIView.handleToastTapped(_:)))
    toast.addGestureRecognizer(recognizer)
    toast.isUserInteractionEnabled = true
    toast.isExclusiveTouch = true
}
```


**ToastStyle**
主要是负责对如下属性封装
```
/**
 `ToastStyle` instances define the look and feel for toast views created via the
 `makeToast` methods as well for toast views created directly with
 `toastViewForMessage(message:title:image:style:)`.

 @warning `ToastStyle` offers relatively simple styling options for the default
 toast view. If you require a toast view with more complex UI, it probably makes more
 sense to create your own custom UIView subclass and present it with the `showToast`
 methods.
*/
public struct ToastStyle {

    public init()

    /**
     The background color. Default is `.black` at 80% opacity.
    */
    public var backgroundColor: UIColor

    /**
     The title color. Default is `UIColor.whiteColor()`.
    */
    public var titleColor: UIColor

    /**
     The message color. Default is `.white`.
    */
    public var messageColor: UIColor

    /**
     A percentage value from 0.0 to 1.0, representing the maximum width of the toast
     view relative to it's superview. Default is 0.8 (80% of the superview's width).
    */
    public var maxWidthPercentage: CGFloat { get set }

    /**
     A percentage value from 0.0 to 1.0, representing the maximum height of the toast
     view relative to it's superview. Default is 0.8 (80% of the superview's height).
    */
    public var maxHeightPercentage: CGFloat { get set }

    /**
     The spacing from the horizontal edge of the toast view to the content. When an image
     is present, this is also used as the padding between the image and the text.
     Default is 10.0.
     
    */
    public var horizontalPadding: CGFloat

    /**
     The spacing from the vertical edge of the toast view to the content. When a title
     is present, this is also used as the padding between the title and the message.
     Default is 10.0. On iOS11+, this value is added added to the `safeAreaInset.top`
     and `safeAreaInsets.bottom`.
    */
    public var verticalPadding: CGFloat

    /**
     The corner radius. Default is 10.0.
    */
    public var cornerRadius: CGFloat

    /**
     The title font. Default is `.boldSystemFont(16.0)`.
    */
    public var titleFont: UIFont

    /**
     The message font. Default is `.systemFont(ofSize: 16.0)`.
    */
    public var messageFont: UIFont

    /**
     The title text alignment. Default is `NSTextAlignment.Left`.
    */
    public var titleAlignment: NSTextAlignment

    /**
     The message text alignment. Default is `NSTextAlignment.Left`.
    */
    public var messageAlignment: NSTextAlignment

    /**
     The maximum number of lines for the title. The default is 0 (no limit).
    */
    public var titleNumberOfLines: Int

    /**
     The maximum number of lines for the message. The default is 0 (no limit).
    */
    public var messageNumberOfLines: Int

    /**
     Enable or disable a shadow on the toast view. Default is `false`.
    */
    public var displayShadow: Bool

    /**
     The shadow color. Default is `.black`.
     */
    public var shadowColor: UIColor

    /**
     A value from 0.0 to 1.0, representing the opacity of the shadow.
     Default is 0.8 (80% opacity).
    */
    public var shadowOpacity: Float { get set }

    /**
     The shadow radius. Default is 6.0.
    */
    public var shadowRadius: CGFloat

    /**
     The shadow offset. The default is 4 x 4.
    */
    public var shadowOffset: CGSize

    /**
     The image size. The default is 80 x 80.
    */
    public var imageSize: CGSize

    /**
     The size of the toast activity view when `makeToastActivity(position:)` is called.
     Default is 100 x 100.
    */
    public var activitySize: CGSize

    /**
     The fade in/out animation duration. Default is 0.2.
     */
    public var fadeDuration: TimeInterval

    /**
     Activity indicator color. Default is `.white`.
     */
    public var activityIndicatorColor: UIColor

    /**
     Activity background color. Default is `.black` at 80% opacity.
     */
    public var activityBackgroundColor: UIColor
}
```


**ToastManager**
ToastManager是一个单利，管理当前样式，是否打开队列，是否点击点击消失，默认样式、延迟时间和位置等
```
// MARK: - Toast Manager

/**
 `ToastManager` provides general configuration options for all toast
 notifications. Backed by a singleton instance.
*/
public class ToastManager {
    
    /**
     The `ToastManager` singleton instance.
     
     */
    public static let shared = ToastManager()
    
    /**
     The shared style. Used whenever toastViewForMessage(message:title:image:style:) is called
     with with a nil style.
     
     */
    public var style = ToastStyle()
    
    /**
     Enables or disables tap to dismiss on toast views. Default is `true`.
     
     */
    public var isTapToDismissEnabled = true
    
    /**
     Enables or disables queueing behavior for toast views. When `true`,
     toast views will appear one after the other. When `false`, multiple toast
     views will appear at the same time (potentially overlapping depending
     on their positions). This has no effect on the toast activity view,
     which operates independently of normal toast views. Default is `false`.
     
     */
    public var isQueueEnabled = false
    
    /**
     The default duration. Used for the `makeToast` and
     `showToast` methods that don't require an explicit duration.
     Default is 3.0.
     
     */
    public var duration: TimeInterval = 3.0
    
    /**
     Sets the default position. Used for the `makeToast` and
     `showToast` methods that don't require an explicit position.
     Default is `ToastPosition.Bottom`.
     
     */
    public var position: ToastPosition = .bottom
    
}
```


#### 总结（价值）
类设计问题
可以借鉴Toast来设计，使用分类扩展方法，使得调方便，但同时增加属性时候，会用到很多runtime的东西
使用Manager来管理一些属性和默认样式(style、时间、位置等)
将样式style抽象成类
多个弹窗的顺序弹出问题
使用NSMutableArray来做为一个Queue来管理toast的弹窗顺序，其中单利的属性isTapToDismissEnabled        等没有，多线程操作的问题，所以不需要线程锁
点击消失问题
给弹窗视图添加手势




作者
zhuhao@ksjgs.com
