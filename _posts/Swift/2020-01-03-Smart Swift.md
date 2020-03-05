---
layout: post
tag: Swift
author: aiQG_
---

Swift的特性, 有了它们或许能够更优雅地编写Swift程序

...持续更新...

---

`contains` 与 `allSatisfy` 用来判断数组中的元素是否满足某个条件, 返回Bool

`contains` 会在遍历到第一个满足条件的元素后停止并返回 `true` , 若全部元素遍历完都没有满足条件的元素则返回 `false`

`allSatisfy` 会完全遍历数组, 所有元素都满足条件才返回 `true` , 否则返回 `false`

---

`where` 关键字用来作为限定条件

如:

```swift
let temp:[Int] = Array(0...1000)
for i in temp where i % 117 == 0 || i % 347 == 0{
	print(i)
}
```

获取第一个满足条件的元素

```swift
let temp:[Int] = [1,2,3,4,5,6,7,8,9,10]
print(
	temp.first(where: { (x) -> Bool in
		x == 7
	})!
)
```

//好像也能简写成

```swift
let temp:[Int] = [1,2,3,4,5,6,7,8,9,10]
print(
	temp.first{ $0 == 7}!
)
```

---

`class` 与 `struct`

- `class` 

引用类型

```swift
class temp {
}
var temp0 = temp()
var temp1 = temp0
print(temp0 === temp1)
```

类可以继承, 但只有类可以继承 (可以用协议代替)

可以 `deinit` (析构函数)

用 ```===``` 判断是否指向同一实例

- `struct`

值类型(写时复制(COW)特性)

不能继承

不可以 `deinit`

用 ```==``` 判断数据是否相等

---

反射 和 `Mirror`

"反射"是一种在==运行时==检测、访问或者修改类型的行为的特性

通过 Mirror 初始化得到的结果中包含的元素的描述都被集合在 children 属性下

```swift
struct Point {
	let x: Int
	let y: Int
}
let p = Point(x: 21, y: 30)
for (label, value) in Mirror(reflecting: p).children {
	print("label: \(label)\tvalue: \(value)")
}
//>>> label: Optional("x")	value: 21
//>>> label: Optional("y")	value: 30

//也可以用dump

dump(p)
//>>>▿ __lldb_expr_3.Point
//>>>  - x: 21
//>>>  - y: 30
```

可以用来生成json, 但是反射是swift一直存在的非正式的功能, 所以可能将来会有很大变动

//目前只能访问属性, 不能修改

---

`defer` 延时调用

`defer` 所声明的 block 会在当前代码执行退出后被调用。

多个 `defer` 按照入栈顺序调用

一般用于资源的释放

在某个函数有多个返回出口(多个 `return` )的时候特别有用。

```swift
class Foo {
    var num = 0
    func foo() -> Int {
        defer { num += 1 }
        return num
    }
    
    // 没有 `defer` 的话我们可能要这么写
    // func foo() -> Int {
    //    num += 1
    //    return num - 1
    // }
}

let f = Foo()
f.foo() // 0
f.foo() // 1
f.num   // 2

// 类似的可以实现"后++"

```

> ⚠️This means that a defer statement can be used, for example, to perform manual resource management such as closing file descriptors, and to perform actions that need to happen **even if an error is thrown**.

---

`enum` 满足协议以作为隐式 `rawValue`

一般为 `String` or `Int`

`enum` 可以递归( `indirect` 关键字表明是一个递归的 `enum` / `case` )

```swift
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// 创建表达式:(5+4)*2
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))

//计算
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}

print(evaluate(product))
```

---

`@escaping` & `@autoclosure`

闭包为引用类型.

`@escaping` 关键字可以让闭包避免在编译时计算(而是使其引用次数+1). //否则闭包将不会被保存

`@autoclosure` 关键字可以将传入的表达式变成闭包

```swift
UIView.animate(withDuration: 0.25) {
    view.frame.origin.y = 100
}
//也可以写成
func animate(_ animation: @autoclosure @escaping () -> Void,
             duration: TimeInterval = 0.25) {
    UIView.animate(withDuration: duration, animations: animation)
}
animate(view.frame.origin.y = 100)
```

---
