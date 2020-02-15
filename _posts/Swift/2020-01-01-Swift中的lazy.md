---
layout: post
tag: Swift
author: aiQG_
---


`lazy` 主要是减少初始化所花费的资源和时间

只能用在 `class` 或 `strutc` 里

`lazy` 声明对变量会在使用到的时候才对其进行赋值(如果这之间没有重新赋值)


比较下面两段程序, 可以明显感觉到运行时间的不同
```swift
class ClasA {
	lazy var value = Array(1...1000000000).map{$0+1}
}
var temp = ClasA()
print(temp.value) //访问了value成员
```

```swift
class ClasA {
	lazy var value = Array(1...1000000000).map{$0+1}
}
var temp = ClasA()
print(temp) //未访问value成员
```

主要是避免计算初值引起的资源浪费

如

```swift
class ClasA {
	lazy var value = Array(1...1000000000).map{$0+1}
}
var temp = ClasA()
temp.value = Array(1...10).map{$0-1} //不要初始值了, 重新赋值!
print(temp.value)
```

*⚠️lazy成员在重新赋值的时候, 会立即计算出要赋予的值. 如即使没有访问 `temp.value` 在 `temp.value = Array(1...10).map{$0-1}` 处仍会计算 `Array(1...10).map{$0-1}`*

---

补充:

那么如果仅仅是访问一个数组中的几一个元素呢, 如

```swift
func addFunction(x: Int) -> Int {
	print("input \(x)")
	return x + 1
}
let temp = Array(1...10).map(addFunction)
print(temp[0])
```

看一下playground显示的信息

![p1](https://i.imgur.com/uXJwjtw.png)

可以看到在初始化的时候map里的函数就被执行了

> 在 `Swift` 标准库中，`SequenceType` 和 `CollectionType` 协议都有个叫 `lazy` 的计算属性，它能给我们返回一个特殊的 `LazySequence` 或者 `LazyCollection` 。这些类型只能被用在 `map` ，`flatMap` ，`filter` 这样的高阶函数中，而且是以一种惰性的方式。

改进:

```swift
func addFunction(x: Int) -> Int {
	print("input \(x)")
	return x + 1
}
let temp = Array(1...10).lazy.map(addFunction)
print(temp[0])
```

再看一下playground显示的信息

![p2](https://i.imgur.com/9d6grUw.png)

*⚠️这个例子中 `Array(1...10)` 还是生成了的, 但是没有进行 `map` 的计算*












