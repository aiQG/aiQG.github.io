---
layout: post
tag: Note
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