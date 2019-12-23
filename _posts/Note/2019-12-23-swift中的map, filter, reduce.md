---
layout: post
tag: Note
author: aiQG_
---

- map

map 理解为对数组中每个元素进行一个相同的操作, 然后返回到原来的位置(元素之间不互相影响)

map 的标准格式:

```swift
var arr: [Int] = [1,2,3,4,5,6,7]
arr.map { (<#Int#>) -> T in
	<#code#>
}
```

如

```swift
arr.map { (aValue: Int) -> Int in
	aValue + 1
} //返回一个[Int], 元素分别是原[Int]的每一个元素加一
//>>>[2, 3, 4, 5, 6, 7, 8]

//优秀写法
arr.map {aValue in aValue + 1}
//更优秀写法
arr.map {$0 + 1}
//$0 代表map的第0个参数

```

- filter

filter 理解为对数组中每个元素进行一个相同的判断, 判断为真则返回当前元素

==(filter的返回值必须为Bool)==

filter 的标准格式:

```swift
var arr: [Int] = [1,2,3,4,5,6,7]
arr.filter({ (<#Int#>) -> Bool in
	<#code#>
})
```

如

```swift
arr.filter({ (aValue: Int) -> Bool in
	aValue > 4
}) //返回一个[Int], 元素分别是判断"大于4"为真的元素
//>>>[5, 6, 7]

//优秀写法
arr.filter({aValue in aValue > 4})
//更优秀写法
arr.filter({$0 > 4})
//$0 是filter第0个参数
//尾随闭包写法
arr.filter(){$0 > 4}
//or
arr.filter {$0 > 4}

```

- reduce

理解为把每一个元素 经过某个规则 合并成一个东西

reduce 有两种实现

1.  ```reduce(initialResult: , nextPartialResult: )``` 的标准格式:

```swift
var arr: [Int] = [1,2,3,4,5,6,7]
arr.reduce(Result, { (<#Result#>, <#Int#>) -> Result in
		<#code#>
	})
```

如

```swift
arr.reduce(0, { (resu, aValue) -> Int in
	resu + aValue
}) //参数resu是reduce的第一个参数0, 参数aValue是arr中的每一个元素, 每一次调用闭包, 返回值都会被保存到resu里, 作为下一次调用闭包的参数. 最后返回resu.
//>>>28

//优秀写法
arr.reduce(0){ resu, aValue in resu + aValue }
//更优秀写法
arr.reduce(0){ $0 + $1 }
//对于个别运算
arr.reduce(0, +) //在这里, 左参数为上面的resu, 右参数为上面的aValue

```

1.  ```reduce(into: , updateAccumulatingResult: )``` 的标准格式:

```swift
var arr: [Int] = [1,2,3,4,5,6,7]
arr.reduce(into: Result, { (<#inout Result#>, <#Int#>) in
	<#code#>
})
```

如

```swift
arr.reduce(into: [], { (resu, aValue) in
	resu.append(aValue)
})

//优秀写法
arr.reduce(into: []) { (resu, aValue) in resu.append(aValue) }
//更优秀写法
arr.reduce(into: []) { $0.append($1) }
```

这个方法适用于有 `copy-on-write` 特性的类型(如 字典, 数组,  集合等)

>  This method is preferred over  `reduce(_:_:)`  for efficiency when the result is a copy-on-write type, for example an Array or a Dictionary.
>  ——*Apple文档*

(如Int等类型会直接复制)


计算字符串中各个字符个数

```swift
let s = "Hello world!"
s.reduce(into: [:]){ $0[$1, default: 0] += 1}
//>>>["w": 1, "H": 1, "l": 3, " ": 1, "e": 1, "d": 1, "o": 2, "!": 1, "r": 1]
```

- 链式语法

如

```swift
//求arr数组中每个元素乘3后的偶数数组, 并将得到的数组每个元素相加减100
var arr: [Int] = [1,2,3,4,5,6,7]
arr.map{$0 * 3}.filter{$0 % 2 == 0}.reduce(-100){$0 + $1}
//-64
```



---

> *Premature optimisation is the root of all evil.*