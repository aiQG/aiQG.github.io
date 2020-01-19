---
layout: post
tag: Note
author: aiQG_
---

> 本文记录UE4学习笔记
> 本文使用第三人称模版
> 本文参考油管上kitty unreal和諶嘉誠CJC的教程

*
Hello Unreal Engine 4.2
目标：UE4源码
~~UE4比CTF有趣多了~~
*
*
在百度搜UE4的教程少的可怜
//墙外的世界真美好
*
*
//听说下一个版本要出摧毁效果了，简直激动人心
*


从制作出生点开始讲起
//虽然ue4自带玩家出生点的控件

一、建立出生点
1. 建立一个BluePrint继承Actor，双击打开
![1](https://i.imgur.com/Jn19KND.png)
2. 添加Box collision（用玩家是否与其重叠判断出生点是否可用）（为了方便选中可用加上一个Billboard），将Box collision作为蓝图自身，加上个箭头，标记正向。
![2](https://i.imgur.com/HfX4Tf6.png)
3. 创建一个函数，用于返回当前出生点是否被占用（actor数组长度大于0则被占用）(可以标注为纯虚函数)
![3](https://i.imgur.com/OxnzL1S.png)
4. actor和出生点开始重叠/结束重叠，对actor数组进行增/减操作
![4](https://i.imgur.com/p1yxWUX.png)

二、建立GameMode（这个东西只有主机存在）
1. 创建蓝图，继承引擎自动生成的GameMode(项目名+GameMode)，在项目设置里设置默认GameMode（和Map）
![5](https://i.imgur.com/wzIhaiJ.png)
2. 创建一个自定义事件，用于设置玩家的方向和位置。在BeginPlay事件里找出所有的出生点保存起来，然后调用刚才创建的自定义事件（传入player 0的Pawn）。
![6](https://i.imgur.com/yq9bVUx.png)
![7](https://i.imgur.com/Qncqx9f.png)
3. 在函数标签里选择重写一个函数：Spawn Default Pawn Transform 右键调用其父类
![8](https://i.imgur.com/AzqFeuP.png)


OK 出生点的基本功能就是这样了



