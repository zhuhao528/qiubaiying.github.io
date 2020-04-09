---
layout:     post
title:      核心动画
subtitle:	 
date:       2020-04-08
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - Core Animation
---

## 核心动画

### 动画归类

`CAAnimation`  核心动画的基础类  
`CAPropertyAnimation` CAAnimation 的子类，用于创建操作层属性值的动画的对象  
`CABasicAnimation` 单关键帧动画功能的对象  
`CAKeyframeAnimation` 提供关键帧动画功能的对象  
`CASpringAnimation` 将弹簧力应用到层属性上的动画的对象  
`CATransition` layer层之间转换的动画的对象  
`CAValueFunction` 灵活转换的动画的对象

### 组动画

`CAAnimationGroup` 允许多个动画被分组并同时运行的对象  
`CATransaction` 一种将多个层树操作分组为呈现树的原子更新的机制

## 核心动画编程指南

### 核心动画基础

#### 图层为绘图和动画提供了基础

`layer`对象是组织在三维空间中的二维表面，是你使用核心动画所做的一切事情的核心。与视图一样，`layer`管理关于其表面的几何、内容和视觉属性的信息。与视图不同，层不定义它们自己的外观。一个层仅仅管理位图周围的状态信息。位图本身可以是视图本身绘制的结果，也可以是您指定的固定图像的结果。因此，您在应用程序中使用的主要层被认为是模型对象，因为它们主要管理数据。记住这个概念很重要，因为它影响动画的行为

##### 基于分层的绘图模型

大多数层在你的应用程序中不做任何实际的绘制。相反，一个层捕获你的应用程序提供的内容，并将其以位图`bitmap`的形式缓存，这有时被称为备份存储。当您随后更改该层的属性时，您所做的全部工作就是更改与该层对象关联的状态信息。当更改触发动画时，Core animation将层的位图和状态信息传递给图形硬件，图形硬件使用新的信息来完成位图`bitmap`的绘制工作。在硬件中操作位图可以得到比在软件中更快的动画效果

##### 层的动画

`layer`的数据和状态信息从屏幕显示中分离出来。这种解耦为核心动画提供了一种方式，使其自身进行干预，并将旧状态值转换为新状态值。例如，改变一个层的位置属性会导致Core Animation将层从当前位置移动到新指定的位置。对其他属性的类似更改会导致适当的动画。图1-2演示了几种可以在层上执行的动画类型。有关触发动画的图层属性列表，请参阅[Animatable Properties](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html#//apple_ref/doc/uid/TP40004514-CH11-SW1)
	
Figure 1-2  Examples of animations you can perform on layers
	
![](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/basics_animation_types_2x.png)

#### 层对象定义它们自己的几何形状

层的工作之一是管理其内容的可视几何图形。可视几何图形包含关于内容边界、其在屏幕上的位置以及该层是否已以任何方式旋转、缩放或转换的信息。与视图一样，层具有框架和边界矩形，您可以使用它们来定位层及其内容。层还具有视图不具有的其他属性，比如`锚点`，它定义了操作发生的点。指定层几何的某些方面的方式也不同于为视图指定信息的方式。

#### 锚点影响几何操作

一个层的几何相关操作相对于该层的`锚点`发生，您可以使用该层的`anchorPoint`属性访问该锚点。在操作层的位置或转换属性时，锚点的影响是最明显的。位置属性总是相对于层的锚点指定的，并且您应用于层的任何转换也相对于锚点发生。

#### 层可以在三维空间中操作

每个层都有两个转换矩阵，您可以使用它们来操作层及其内容。CALayer的transform属性指定了您想要应用到层及其嵌入的子层的转换。通常，当您想要修改层本身时，会使用此属性。例如，您可以使用该属性来缩放或旋转该层或临时更改其位置。subblayertransform属性定义了仅应用于子层的其他转换，并且最常用来为场景的内容添加透视图视觉效果。

#### 层树反映了动画状态的不同方面

一个使用核心动画的应用程序有三组`layer`对象。每一组层对象有不同的作用，使你的应用程序的内容出现在屏幕上:
  
* 模型层树中的对象(或者简单地称为“层树”)是您的应用程序与之交互最多的对象。此树中的对象是存储任何动画的目标值的模型对象。当您更改一个层的属性时，您将使用这些对象之一。

* 表示树中的对象包含任何正在运行的动画的动态值的对象。虽然层树对象包含动画的目标值，但是表示树中的对象在屏幕上显示时反映当前值。永远不要修改这个树中的对象。相反，您可以使用这些对象来读取当前的动画值，也许可以从这些值开始创建一个新的动画。

* 渲染树中的对象执行实际的动画的对象，并且是核心动画的私有对象。

#### layer和view之间的区别

> Layers are not a replacement for your app’s views—that is, you cannot create a visual interface based solely on layer objects. Layers provide infrastructure for your views

层不能代替你的应用程序的视图——也就是说，你不能仅仅基于层对象来创建一个可视的界面。层为视图提供基础结构。具体来说，层可以更容易、更有效地绘制和动画视图的内容，并在此过程中保持较高的帧率。然而，有许多事情是层不能做的。层不处理事件、绘制内容、参与响应链或做许多其他事情。因此，每个应用程序都必须有一个或多个视图来处理这些交互

### 设置图层对象

#### 当我使用图片作为图层对象
> Because a layer is just a container for managing a bitmap image, you can assign an image directly to the layer’s contents property.

当我们添加图片昨天图层时，我们修改是`contents `属性

>The image you assign to a layer must be a CGImageRef type

我们传递给layer的图片对象必须是`CGImageRef `类型

### 图层内容设置动画

#### 使用关键帧动画来更改图层的属性

虽然基于属性的动画会将属性从起始值更改为结束值，但是CAKeyframeAnimation对象允许您以一种可能是线性也可能不是线性的方式对一组目标值进行动画处理。关键帧动画由一组目标数据值和达到每个值的时间组成。在最简单的配置中，使用数组指定值和时间。对于层位置的更改，您还可以让更改遵循一条路径。animation对象获取您指定的关键帧，并通过在给定时间段内从一个值插入到下一个值来构建动画。

创建一个关键帧的动画

```
// create a CGPath that implements two arcs (a bounce)
CGMutablePathRef thePath = CGPathCreateMutable();
CGPathMoveToPoint(thePath,NULL,74.0,74.0);
CGPathAddCurveToPoint(thePath,NULL,74.0,500.0,
                                   320.0,500.0,
                                   320.0,74.0);
CGPathAddCurveToPoint(thePath,NULL,320.0,500.0,
                                   566.0,500.0,
                                   566.0,74.0);
 
CAKeyframeAnimation * theAnimation;
 
// Create the animation object, specifying the position property as the key path.
theAnimation=[CAKeyframeAnimation animationWithKeyPath:@"position"];
theAnimation.path=thePath;
theAnimation.duration=5.0;
 
// Add the animation to the layer.
[theLayer addAnimation:theAnimation forKey:@"position"];
```

### 先进的动画技巧

#### 过渡动画支持改变层的可见性

顾名思义，一个transition animation对象为一个层创建一个动画视觉转换，转换对象最常见的用法是以一种协调的方式使一个层的出现和另一个层的消失具有动画效果。不像基于属性的动画，动画改变一个图层的一个属性，过渡动画操作一个图层缓存的图像来创建视觉效果，这是很难或不可能做到的，只改变属性。标准类型的转换允许您执行显示、推送、移动或交叉淡出动画。在OS X上，您还可以使用核心图像过滤器来创建使用其他类型效果的过渡，如擦除、页面卷曲、波纹或您设计的自定义效果

下面的代码演示两个视图之间的转换，在例子中`view1`和`view2`都在父视图的相同的位置，只不过只有`view1`可见，将`view1`从左边滑出且渐渐消失知道隐藏，此时`view2`从右边慢慢滑入直到可见，更改两个视图的属性，确保动画结束的时候都有正确的显示

```
CATransition* transition = [CATransition animation];
transition.startProgress = 0;
transition.endProgress = 1.0;
transition.type = kCATransitionPush;
transition.subtype = kCATransitionFromRight;
transition.duration = 1.0;
 
// Add the transition animation to both layers
[myView1.layer addAnimation:transition forKey:@"transition"];
[myView2.layer addAnimation:transition forKey:@"transition"];
 
// Finally, change the visibility of the layers.
myView1.hidden = YES;
myView2.hidden = NO;
```
其他 自定义动画的时间、动画暂停与恢复、改变动画的参数、改变动画的角度等 这里不做展开了 
可以查看[Advanced Animation Tricks](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/AdvancedAnimationTricks/AdvancedAnimationTricks.html#//apple_ref/doc/uid/TP40004514-CH8-SW1)


### 更改动画的默认行为

顾名思义 你可以自定义CAAction协议、更改响应对象`action object`、取消系统的临时动画等可以查看[Changing a Layer’s Default Behavior](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/ReactingtoLayerChanges/ReactingtoLayerChanges.html#//apple_ref/doc/uid/TP40004514-CH7-SW1)

## 应用

我们使用`CAKeyframeAnimation `来播放本地gif动画 待整理

>Note 动画只是给我们状态或者属性等增加一些过渡效果，本质是不影响数据本身的

参考  
[CoreAnimation](https://developer.apple.com/documentation/quartzcore)  
[Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW3)

