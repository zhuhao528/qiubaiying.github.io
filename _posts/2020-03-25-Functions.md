---
layout:     post
title:      Swift 函数
subtitle:	 
date:       2020-03-27
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - Swift  Functions
---
总结了一下swift的函数部分

1 Multiple Return Values （多参数）
多参数可以返回元组比如 (min: Int, max: Int) 

2 Functions With an Implicit Return （隐式返回）

```
func greeting(for person: String) -> String {
   "Hello, " + person + "!"
}
print(greeting(for: "Dave")) 
// Prints "Hello, Dave!"
```
像这样简短的函数可以省掉retrun

3 Function Argument Labels and Parameter Names （参数标签和参数名称）

```
func greet(person: String, from hometown: String) -> String {
	return "Hello \(person)!  Glad you could visit from \(hometown)."
}
print(greet(person: "Bill", from: "Cupertino"))
// Prints "Hello Bill!  Glad you could visit from Cupertino."
```
参数有由Argument和Parameter两部分组成，其中Argument使得函数读起来更像一个句子，表达的更加清楚 
4 Omitting Argument Labels （参数标签）

```
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(1, secondParameterName: 2)
```
我这里暂且叫Argument为标签吧
如果不想给参数加一个标签，可以Argument设置为_ ，这样函数调用的时候标签和参数名都不需要写了

5 Default Parameter Values （参数的默认值）

```
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
	    // If you omit the second argument when calling this function, then
	    // the value of parameterWithDefault is 12 inside the function body.
}
someFunction(parameterWithoutDefault: 3, parameterWithDefault: 6) // parameterWithDefault is 6
someFunction(parameterWithoutDefault: 4) //
```

6 Variadic Parameters （多参数）

```
func arithmeticMean(_ numbers: Double...) -> Double {
	var total: Double = 0
	for number in numbers {
		total += number
	  }
	 return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// returns 3.0, which is the arithmetic mean of these five numbers
arithmeticMean(3, 8.25, 18.75)
```
这个 不说废话了，看代码估计就能看明白

7 In-Out Parameters （函数想改变函数以外的值）

```
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
	let temporaryA = a
	a = b
	b = temporaryA
}
```
对不OC的就像相对传递的是对象地址

8 Function Types

```
func addTwoInts(_ a: Int, _ b: Int) -> Int {
	return a + b
}
```
函数类型，函数式编程中函数相当于一个对象  
那么上面代码中的函数类型便是(Int, Int) -> Int  
这样方便我们Using Function Types  
比如 var mathFunction: (Int, Int) -> Int = addTwoInts 定义一个函数类型的变量

```  
print("Result: \(mathFunction(2, 3))")  
// Prints "Result: 5"
```

除此之外你还可以将函数变量赋值给其他的变量或者常量. 
比如 let anotherMathFunction = addTwoInts
这个在OC 有点想函数指针，不过好像没没有这么用，swift是怎么做到的呢

9 Function Types as Parameter Types （函数中函数函数类型的参数）

```
func printMathResult(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
	print("Result: \(mathFunction(a, b))")
}
printMathResult(addTwoInts, 3, 5)
```
你把函数理解成变量或者对象，这里就不难理解，函数式编程的思想

10 Function Types as Return Types 

```
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
	return backward ? stepBackward : stepForward
}
```
这里类似这个写法，原理和9一样这里就不奢述了

11 Nested Functions （嵌套函数）
还是函数式的思想

```
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
	func stepForward(input: Int) -> Int { return input + 1 }
	func stepBackward(input: Int) -> Int { return input - 1 }
	return backward ? stepBackward : stepForward
}
```
这里一样函数是有类型的，stepForward是函数变量而不仅仅是和函数名称了 有点类似OC中的block 
Block作为一函数代码块，用法有时候也可以是一个属性，然后给其去赋值、返回等

参考  
[Functions](https://docs.swift.org/swift-book/LanguageGuide/Functions.html)
