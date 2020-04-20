---
layout: post
tag: Developing & Reversing
author: aiQG_
---

[*Using Combine*](https://heckj.github.io/swiftui-notes)

操作符类符合Subscriber协议和Publisher协议

可以拆分/合并数据流


A simple Combine pipeline written in swift might look like:
```swift

let _ = Just(5)    // 输出类型为<Integer>, 失败类型为<Never>
    .map { value -> String in
        // do something with the incoming value here
        // and return a string
        return "a string"
    }              // 输出类型为<String>, 失败类型为<Never>
    .sink { receivedValue in
        // sink is the subscriber and terminates the pipeline
        print("The end result was \(receivedValue)")
    }              // sink用来结束

```

SwiftUI使用Combine提供的@Published和@ObservedObject属性包装器隐式创建发布者

一个简单的例子

```swift
import UIKit
import Combine

class ViewController: UIViewController {
	
	@IBOutlet weak var textLabel: UILabel!
	@IBOutlet weak var actionButton: UIButton!
	
	@Published var labelValue: String? = "Click the button!"
	
	var cancellable: AnyCancellable?
	
	override func viewDidLoad() {
		super.viewDidLoad()
		
		self.cancellable = self.$labelValue.receive(on: DispatchQueue.main)
			.assign(to: \.text, on: self.textLabel)
		
	}
	
	@IBAction func actionButtonTouched(_ sender: UIButton) {
		self.labelValue = "Hello World!"
	}
}

```

通过使用 `$` 和  `assign` 函数, 可以创建绑定并订阅值的修改.

如果 `labelValue` 被改变, 则 `self.textLabel` 的 `text` 也会被修改. (UI会被更新)

// 因为这是更新UI的功能, 所以在主队列上进行. ( `receive` 指定)

`$` 可以访问被(@Published)包装的值


[Introducing Combine](https://developer.apple.com/videos/play/wwdc2019/722/)

[Combine in Practice](https://developer.apple.com/videos/play/wwdc2019/721/)

|        | 同步  |   异步    |
| :----: | :---: | :-------: |
| 单个值 |  Int  |  Future   |
| 多个值 | Array | Publisher |


上游到下游的传递, 会在遇到错误/结束时终止

Operators⬇️: 介于Subscribers与Publishers之间(充当数据处理的角色)

Zip: 可以将多个上游的值转换为单个元组, 发至下游

CombineLatest: 若上游其中一个值改变, 则重新组合成一个元组发至下游

debounce: 间隔一段时间向下游传递值

removeDuplicates: 删除重复的值(这次从上游传下来的值与上次的相同, 则不再向下游传递)

assign: 将发布者传递的值应用于由keypath定义的对象上
```swift
.assign(to: \.isEnabled, on: aButton)
```

sink: 接收一个任何类型的闭包, 并终止到下游的传输(debug很有用)

catch: 可以提供一个闭包. 在收到上游的错误后: 1). **停止**上游的连接. 2). 将闭包作为上游(Publisher), 开始接收闭包的值 (可以替换原来的Publisher)(闭包return一个Publisher(Just))

可以利用flatMap防止与上游的连结断开:
```swift

//******
.map{****}
.flatMap{ data in
    return Just(data)
    .decode(***.self, JSONDecoder())//这里可能会产生错误
    .catch{
        return Just(***.NoError)//这里一定不产生错误
    }
}

// 放在flatMap内部, 不影响外部与上游的连接
```

// 还有各种高阶函数

tryMap: 相比map(不改变并继续传递失败类型,)可以抛出错误. 可以提供一个出错时调用的闭包.
```swift
Just(5)
.tryMap {
    if inValue < 5 { 
        throw MyFailure.notBigEnough 
    }
    return inValue 
}
```



