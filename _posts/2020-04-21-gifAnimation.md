---
layout:     post
title:      gif动画
subtitle:	 
date:       2020-04-21
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - gif动画
---

动画是开发过程中经常遇到的，这次开发需求中遇到了音频播放和录音的需求，音频播放伴随着语音波形，当然这里暂时不是使用语音频谱波形图，毕竟是动漫类的效果，根据设计效果主要是图片的切换的动画，所以考虑到使用`UIImageView`的图片动画

```
let imageView = UIImageView()
imageView.image = UIImage(named: "icon_follow_soundbyte_right")
imageView.animationImages = [
	UIImage(named: "icon_follow_soundbyte_left")!,
	UIImage(named: "icon_follow_soundbyte_right")!,
]
imageView.animationDuration = 0.5
imageView.isUserInteractionEnabled = false
return imageView
```
譬如上面这个例子，使用图片动画，可以满足简单图片的动画，当然设计小姐姐当图片较多时，比如人的说话、转脸带着诸多表情变化的时候，这是由于图片较多就给出了gif的资源，在使用gif的时候有同样的的思路，将读取gif资源图片使用`UIImageView `的图片动画

```
/// 给UIImageView设置gif图
/// - Parameters:
///   - gifFileName: gif图名称
///   - duration: 动画时间
public func ks_setAnimationImage(_ gifFileName: String, _ duration: TimeInterval = 0.05, _ repeatCount : Int = 0) {
    if let gifFileUrl = Bundle.main.url(forResource: gifFileName, withExtension: "gif") {
        if let gifSource = CGImageSourceCreateWithURL(gifFileUrl as CFURL, nil) {
            let frameCount = CGImageSourceGetCount(gifSource)
            var framesArray: [UIImage] = []
            for index in 0 ..< frameCount {
                if let imageRef = CGImageSourceCreateImageAtIndex(gifSource, index, nil) {
                    let image = UIImage(cgImage: imageRef)
                    framesArray.append(image)
                } else {
                    continue
                }
            }
            animationRepeatCount = repeatCount
            animationImages = framesArray
            animationDuration = duration
            image = framesArray.last
        }
    }
}
```

可是`UIImageView`的图片动画可是展示gif动画，但是在当存在一种情况就是播放完一个gif动画然后再播放另外一组gif的动画的时候，由于`UIImageView `的图片动画是没有`completion`回调的，所以这里有出现了不能满足我们需求的场景，这个时候我们想到了核心动画，使用`CAKeyframeAnimation`的`content`路径来播放图片动画，然后赋值给`UIImageView`的的`layer`

```
func anmation(_ toView:UIView, _ gifFileName:String, _ repeatCount:Float = MAXFLOAT) {
    let animation = CAKeyframeAnimation(keyPath: "contents")
    animation.delegate = self
    if let gifFileUrl = Bundle.main.url(forResource: gifFileName, withExtension: "gif") {
        if let gifSource = CGImageSourceCreateWithURL(gifFileUrl as CFURL, nil) {
            let frameCount = CGImageSourceGetCount(gifSource)
            var framesArray: [CGImage] = []
            for index in 0 ..< frameCount {
                if let imageRef = CGImageSourceCreateImageAtIndex(gifSource, index, nil) {
                    framesArray.append(imageRef)
                } else {
                    continue
                }
            }
            animation.values = framesArray
            animation.duration = CFTimeInterval(framesArray.count) * 0.3
        }
    }
    animation.repeatCount = repeatCount
    animation.isRemovedOnCompletion = false
    animation.setValue(gifFileName, forKey: "ani")
    animation.fillMode = CAMediaTimingFillMode.forwards
    toView.layer.add(animation, forKey: gifFileName)
}
```

起初`CAKeyframeAnimation`的动画的完成调如下代码

```
func animationDidStop(_ anim: CAAnimation, finished flag: Bool){
    let key = anim.value(forKey: "ani") as! String
    if key ==  gif_follow_look_front_to_down {
        kidAnimationImageView.layer.removeAnimation(forKey: gif_follow_look_front_to_down)
        lookdown()
    }
}
```

如果封装使用的话，需要在封装一层`protocol`来供别人使用，使用起来也不是很方便，有没有可能直接使用闭包来完成呢,于是使用了`CATransaction`动画可以用与图层之前转换动画（转场动画）且有动画`completion`回调，详情可以参考我之前的文章[核心动画](https://zhuhao.co/2020/04/08/Animation/)，于是gif的动画我修改为了如下代码：

```
func anmation(_ toView:UIView, _ gifFileName:String, _ repeatCount:Float = MAXFLOAT, completion: @escaping ()->Void) {
    CATransaction.begin()
    CATransaction.setCompletionBlock({ // 动画结束回调
        completion()
    })
    let animation = CAKeyframeAnimation(keyPath: "contents")
    if let gifFileUrl = Bundle.main.url(forResource: gifFileName, withExtension: "gif") {
        if let gifSource = CGImageSourceCreateWithURL(gifFileUrl as CFURL, nil) {
            let frameCount = CGImageSourceGetCount(gifSource)
            var framesArray: [CGImage] = []
            for index in 0 ..< frameCount {
                if let imageRef = CGImageSourceCreateImageAtIndex(gifSource, index, nil) {
                    framesArray.append(imageRef)
                } else {
                    continue
                }
            }
            animation.values = framesArray
            animation.duration = CFTimeInterval(framesArray.count) * 0.3
        }
    }
    animation.repeatCount = repeatCount
    animation.isRemovedOnCompletion = false
    animation.setValue(gifFileName, forKey: "ani")
    animation.fillMode = CAMediaTimingFillMode.forwards
    toView.layer.add(animation, forKey: gifFileName)
    CATransaction.commit()
}
```

这里添加一个`CATransaction`动画，使用gif动画有的可以使用闭包作为回调，使用起来也比较方便

> 注意：这里`animation`的`values`使用的是`CGImage`类型，当我们添加图片昨天图层时，我们修改是contents 属性, 我们传递给layer的图片对象必须是CGImageRef 类型，详见[核心动画](https://zhuhao.co/2020/04/08/Animation/)  
> 细心的可能还注意到了`@escaping `这个关键字，这个关键字意识就是逃逸闭包，顾名思义就是函数`return`以后并为执行这个闭包，而是等到要执行的再去执行，详见[闭包](https://zhuhao.co/2020/03/27/Closures/)，显然代码中的闭包是在尾部，调用的使用可以使用尾部闭包，[闭包](https://zhuhao.co/2020/03/27/Closures/)也提到如何使用尾部闭包来简化代码，当然如果你习惯约定俗成，通熟易懂也可以不使用尾部闭包

作者  
zhuhao@ksjgs.com

