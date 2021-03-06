---
layout: post
tag: CTF
author: aiQG_
---



//这是一个macOS下用swift5 + SpriteKit框架写出来的touchbar上的游戏
考虑到有mac的师傅并不多，所以这里提供一个静态分析的解法...

//[动态分析链接](/2020/01/08/macOS逆向-之-你大概需要一个带bar的mac-动态.html)

得到的是一个后缀为.app的文件夹(macOS下的应用)。Contents\MacOS下有一个```touchbarGame```这个就是游戏本体


> 关于.app里各个文件：
> \_CodeSignature文件夹 是各个文件的数字签名防止被篡改
> Resources文件夹 是各种资源文件
> MacOS文件夹 是此App的真正可执行文件
> Info.plist文件 是App的基本信息(比如最低系统版本要求, 版本号, Copyright等等标识)
> PkgInfo文件 是一个可选的8个字节长度的文件, 可保存程序类型和创建者签名(当然这些可以写在 Info.plist 中), 这个文件通常包含四字节的程序类型信息(通常为 APPL)和四个字节的签名信息(比如 System Preferences.app 的 PkgInfo 就是 APPLsprf)
> Assets.car中保存着资源图片


[//关于资源文件Assets.car的分析链接](https://blog.timac.org/2018/1018-reverse-engineering-the-car-file-format/)
Assets.car文件可以提取出资源图片(似乎有在线提取的, 但是并不能提取出全部的东西) (关于提取资源, 似乎没有找到非macOS上的现成工具)
![mac3](https://i.imgur.com/jWTjHmZ.png)
![mac4](https://i.imgur.com/RDfIzWQ.png)
可以看到有"shot","rock"和"enemy"的图片, 猜测是"打飞机的游戏"(跑不起来只能猜了呗...)
//这个游戏打起来是这样的......![gaming](https://i.imgur.com/mD1jVkC.gif)

ida分析touchbarGame可以看到函数里有Objective-C的方法和swift的函数, 大多数的函数的名字ida都给分析出来了//所以猜测个别函数的作用还是比较简单的
![mac1](https://i.imgur.com/qkAM9mL.png)

> OC调用函数的机制是"send message"(OC的Runtime), 相当于在运行时才确定函数调用, 所以ctrl+x查看函数的交叉引用几乎是找不到啥信息的了

因为无法运行(无法知道程序输入输出), 我们可以找找程序中的字符串, 提取关键信息




+ 发现有个"/114514) R(estart)"
//114514???
我们交叉引用跟过去到```sub_100002190```, 发现这个函数里OC发了很多带"set"的消息, 可以推断这里是初始化整个游戏的位置
那这个字符串应该就是一个(提示重新开始的)Label


+ 可以找到头的信息
![mac2](https://i.imgur.com/IV35qBC.png)
看到使用了几个框架, google一下发现SpriteKit这个框架有"Physics Simulation"的class, 其中有一个[SKPhysicsContact](https://developer.apple.com/documentation/spritekit/skphysicscontact)看Apple文档的描述可以发现这个东西和物体的接触(碰撞)有关(因为这是个游戏, 所以检测碰撞的功能是一个关键的位置)
继续往下看文档, 有两个变量```bodyA```和```bodyB```
由于OC执行函数的机制(发消息), 我们可以在字符串中搜到这两个变量的名字, 并且可以交叉引用找到给Runtime System 发消息的函数```sub_1000034C0```
![mac5](https://i.imgur.com/0ReT786.png)

//也可以尝试搜索一下包含"print"的函数, 找到```_$ss5print_9separator10terminatoryypd_S2StF```, 发现有两处调用了这个函数, 而且也都在```sub_1000034C0```里

可以仔细分析一下```sub_1000034C0```函数
发现有几个立即数, 很明显是字符串
![mac6](https://i.imgur.com/Ze6WHkE.png)
"hit player"
显然这是一个swift 调用print 的结构, 再往下看看能找到类似的结构
![mac7](https://i.imgur.com/shDXDtb.png)
"hit enemy"

显然这个函数判断了触发碰撞的是哪两个物体

再仔细看一下"hit player"下面的部分, 又发现了一个字符串
![mac8](https://i.imgur.com/bVXCoNn.png)
"Game Over"
是个Label, 并且这个Label是和"hit player"处于同层(同一个if), 可以推断游戏如果触发了"hit player"必定触发"Game Over"

//```_sSS5write2toyxz_ts16TextOutputStreamRzlF```函数其实在给Label初始化字符串

//其实在"hit player"下面很近的地方就能找到"KilledPlayer", 在"hit enemy"下面很近的地方能找到"killEnemy". 这两个位置其实是调用了Resources文件夹里两个.sks的粒子文件
![mac9](https://i.imgur.com/alT47K4.png)

接下来主要分析一下"hit enemy"部分的代码
很容易看见有个if判断了一个变量的数值是否为114514, 之前的分析也发现了这个奇怪的值(而且是在字符串里), 很可疑,  重点关注
![macA](https://i.imgur.com/45cHEUJ.png)
v53[v115]这个变量可以往回追溯, 发现
![macB](https://i.imgur.com/f2E9Hem.png)
v53 来自 v124,  ```OBJC_IVAR____TtC12touchbarGame17TouchbarGameScene_score```这个常量的值是0x28, v115的值同样也是  ```OBJC_IVAR____TtC12touchbarGame17TouchbarGameScene_score```, 并且这个名称一看就和分数有关, 并且  ```v28 = v26 + 1```这个位置很明显是分数加一
因此那个if其实是在判断分数是否为114514

继续分析判断分数为114514后程序执行了什么
能看到这个if最后有个goto
![macC](https://i.imgur.com/wghTuLs.png)
跳过了对一个变量的赋值(赋0), 并且这个变量最后会被放到一个Label里//并且可以看到"setPosition"消息的参数和"Game Over"Label的"setPosition"参数一样, 说明这两个东西出现的位置相同, 但内容必定不同///而且v65是经过了一个while运算(其实是一个数值往字符串转换的操作)
//这里需要注意一下由于Swift是类型安全的. 所以, 数值并不能直接转成字符(其实程序在判断分数为114514后, 执行的是一个数值转字符串的操作)

那么我们主要分析一下从得分("hit enemy")到判断分数为114514(打印flag)之间的内容(因为可以推断这里面很可能有对flag数组加解密的操作)

首先能看到好几个常量字符串
![macD](https://i.imgur.com/ADmlKLP.png)
"remove"了两个"body", 然后播放了一个动画(去查了查Apple开发者文档, 发现SKAction是一个用来播放动画的class)

接着能看见```OBJC_IVAR____TtC12touchbarGame17TouchbarGameScene_flagArr```这个变量, 它和前面的```OBJC_IVAR____TtC12touchbarGame17TouchbarGameScene_score```有相同的前缀```OBJC_IVAR____```, 所以我们可知这里也是一个变量(其实是一个数组), 这个变量被传递到后面进行了一个计算
![macE](https://i.imgur.com/R4yoyno.png)
我们可以稍微整理一下得到 ```*(v34 + v116%v31 + 0x20) ^= v32``` 
其中```v32```和```v116```是得到的总分数, ```v31(=v30+0x10)```是数组的长度, ```v34+0x20```处是数组第0个元素的位置
再整理一下可以得到, ```(Arr + score%Arr.length) ^= score```
(注意这里有个数据类型的转换, ida下按'\\'键显示)
//中间的两个if是swift内存安全的检查, 防止数组越界造成内存泄露

//关于Swift数组的结构:
![swiftArr](https://i.imgur.com/X2Andab.png)

按照这个结构可以在```__data:```段里找到多个像这样相同的数组
![flagArr](https://i.imgur.com/xjhToPf.png)

---

到这里这个函数的功能就整理清楚了
判断是"player"发生了碰撞, 还是"enemy"发生了碰撞. 

- 如果是"player"发生碰撞, 则"Game Over";
- 如果"enemy"发生碰撞, 则: 
1. 初始化killEnemy.sks的粒子
2. 分数+1
3. **对flagArr数组进行运算**
4. 移除bodyA
5. 移除bodyB(移除两个发生碰撞的物体)
6. 按照"killEnemy.sks"文件生成粒子
7. 检查分数是否为114514, 如果分数为114514, 则生成一个字符串; 否则不生成字符串
8. 如果生成了字符串, 则生成一个Label

---

那我们主要关注对flagArr计算的位置
由于只在```__data:```段里找到了一个数组(多个相同的), 那我们可以尝试一下解这个数组

```swift
var flagArr:[UInt8] = [55, 32, 78, 37, 55, 98, 241, 242, 147, 177, 160, 31, 70, 34, 15, 60, 231, 178, 146, 144, 239, 20, 98, 114, 78, 30, 141, 151, 136, 185, 197, 51, 124, 61, 75, 111, 157, 205, 239, 232, 237]

var flag = ""

for i in 1...114514 {
	flagArr[i%flagArr.count] = flagArr[i%flagArr.count] ^ UInt8(i & 0xFF)	
}

for j in flagArr {
	flag += String(UnicodeScalar(j))
}

print(flag)

```

可解出flag

---

//关于指针的问题
可以看到程序中很多的指针, 这是swift(OC)de Runtime性质, 在分析的时候不必过度地纠结指针到底指向什么东西(除非想要深入了解OC的Runtime机制)


//ida分析出的函数名给我们提供了很大的帮助







