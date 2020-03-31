---
layout:     post
title:      Swift Closures
subtitle:	 
date:       2020-03-27
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - Swift Closures
---

## Closeures

>Closures are self-contained blocks of functionality that can be passed around and used in your code. Closures in Swift are similar to blocks in C and Objective-C and to lambdas in other programming languages.

闭包和c与oc中`block`和`lambdas`特别的像

> Global and nested functions, as introduced in Functions, are actually special cases of closures. Closures take one of three forms:

全局函数和嵌套函数是闭包的特殊形式，闭包有三种形式：  

* 全局函数是具有名称且不捕获任何值的闭包
* 嵌套函数是具有名称的闭包，可以从其封闭函数中捕获值
* 闭包表达式是用轻量级语法编写的未命名闭包，可以从它们周围的上下文捕获值

## Closure Expressions （闭包表达式）

```
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
// reversedNames is equal to ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```

方法`sort(by:)`方法接受一个闭包，该闭包接受与数组内容类型相同的两个参数，并返回一个Bool值，以说明排序后第一个值应该出现在第二个值之前还是之后。如果第一个值出现在第二个值之前，排序闭包需要返回true，否则返回false。

这个例子是对一个字符串值数组进行排序，因此排序闭包需要是类型(String, String) -> Bool的函数。

提供排序闭包的一种方法是编写一个正确类型的普通函数，并将其作为参数传递给sort (by:)方法

### Closure Expression Syntax (闭包表达式的语法)

```
{ (parameters) -> return type in
    statements
}
```

>The parameters in closure expression syntax can be in-out parameters, but they can’t have a default value

**注意的是闭包的参数是不能设置默认值的**

```
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```
这样的话就和上文中将一个函数作为参数传入，直接写成了闭包的方式更加方便简洁


>The start of the closure’s body is introduced by the `in` keyword. This keyword indicates that the definition of the closure’s parameters and return type has finished, and the body of the closure is about to begin.

闭包主体的开始由in关键字引入。这个关键字表示闭包的参数和返回类型的定义已经完成，闭包的主体即将开始。

### Inferring Type From Context (从上下文推断类型)

因为所有类型都可以推断，所以返回箭头(->)和参数名称周围的括号也可以省略:

`reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )`

**尽管如此，如果您愿意，您仍然可以使类型显式化，如果这样做可以避免代码读者产生歧义，则鼓励您这样做。**

### Implicit Returns from Single-Expression Closures（隐式返回）

单表达式闭包明确表示返回单行表达式的结果的时候可以省略`return`关键字 

`reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )`

因为闭包的内容只有一条表达式返回一个`Bool`类型，是没有歧义的，所以`return`关键字是可以删除的

### Shorthand Argument Names（省略参数名称）

Swift自动为内联闭包提供简写参数名，可以使用$0、$1、$2等名称引用闭包参数的值。

当你在这写闭包表达式中使用简短的参数的时候，你可以省略闭包的参数的数量和类型，因为这些可以从的特定的函数类型中可以推断出来，`in`关键字也可以省略，因为闭包是由函数主题构成

`reversedNames = names.sorted(by: { $0 > $1 } ) `


### Operator Methods （操作符方法）

因为字符串定义了一个特殊的操作符`>`方法，类型也是(string,string)->bool，所以这里可以简写成 

`reversedNames = names.sorted(by: >) `

更多的内容请查看[Operator Methods](https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html#ID42)


## Trailing Closures(尾部闭包)

如果需要将闭包表达式作为函数的最后一个参数传递给函数，并且闭包表达式很长，那么将它写成结尾闭包是很有用的。`尽管闭包是函数的一个参数，但是尾部闭包是写在函数的括号的后面。`当你使用尾部闭包的时候，你不需要在给闭包添加参数标签`argument label `

```
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}

// Here's how you call this function without using a trailing closure:

someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})

// Here's how you call this function with a trailing closure instead:

someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}

```

最后一个就是尾部闭包的写法了 所以字符串排序就可以写成如下的写法了

`reversedNames = names.sorted() { $0 > $1 } `

如果一个闭包表达式是作为函数或方法的`唯一`参数提供的，并且您将该表达式作为一个`尾闭包`提供，那么在调用函数或方法时，您不需要在函数或方法的名称后面编写一对圆括号():

`reversedNames = names.sorted { $0 > $1 } `

尾部闭包最有用的时候是尾部的比较长，无法使用一个单句时,这个时候就显得非常有用了

比如map(_:)方法

```
let strings = numbers.map { (number) -> String in
    var number = number
    var output = ""
    repeat {
        output = digitNames[number % 10]! + output
        number /= 10
    } while number > 0
    return output
}
// strings is inferred to be of type [String]
// its value is ["OneSix", "FiveEight", "FiveOneZero"]

```

上述的例子要注意的地方：通常会使用闭包的参数来初始化一个变量来供闭包使用，这样变量的值才可以修改（因为函数参数和闭包参数通常是一个常量），最后闭包有一个返回值说明返回的数据类型会被保存并输出到新的数组

> NOTE
> 这里注意的是digitNames字典后面跟的是!，如果key不存在，字典里会查找失败，最后返回一个可选值，在上面的示例中，可以保证数字% 10始终是digitNames字典的有效下标键，所以这里使用感叹号来强解一个可选类型的字符串。

在上面的例子中，尾闭包语法的使用巧妙地将闭包的功能封装在闭包支持的函数之后，而不需要将整个闭包封装在map(_:)方法的外括号中。

## Capturing Values(捕获值)

最简洁的捕获值的形式就是嵌套函数，嵌套函数可以捕获定义其外的常量和变量，他可以改变其值，尽管定义其值的变量已经不不存在

```
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}

```
想看函数如何作为返回类型请查看[Function Types as Return Types](https://docs.swift.org/swift-book/LanguageGuide/Functions.html#ID177)

> NOTE 
> 有一个优化，如果闭包没有改变这个这个值，swift便取消捕获而去保存这个值
> Swift 还处理不在需要的变量的内存管理

举个例子

```
let incrementByTen = makeIncrementer(forIncrement: 10)
```

```
incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

runningTotal这个变量每次都被调用，但是它的值会被保存，所以看到累加的结果

当你创建一个新的incrementer的时候，他会重新保存一个runningTotal

```
let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven()
// returns a value of 7
```

重新调用之前的incrementByTen，还是原来的runningTotal变量，不受incrementBySeven的影响

```
incrementByTen()
// returns a value of 40
```

> NOTE
> 如果你将闭包作为你个类的属性，同时闭包通过捕获类或者成员变量而捕获了类，这样你会在类和闭包之间构成循环引用，Swift使用`捕获列表`来打破这种闭包和类之间的循环引用，更多信息请查看[see Strong Reference Cycles for Closures](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID56)

## Closures Are Reference Types（闭包是引用类型）

未完待续

## Escaping Closures（逃离闭包）

未完待续

## Autoclosures（自动闭包）

未完待续

参考  
[Closures](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)
