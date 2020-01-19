---
layout: post
tag: Other
author: aiQG_
---


接下来做一个“传送点”，用来在两张地图上转移角色。
//这里用流关卡(Stream Level)实现

创建一个Actor，的蓝图，添加一个Box Collision。
/*
也可以再加上一个Billboard（当然也可以加入一个模型）
Billboard 有个属性"Hidden in Game"
对于整个蓝图自身有个属性"Net Load on Client"
适当调整可以控制Billboard在客户端和主机的可见性
*/


创建一个空的关卡，将关卡添加进去
![1](https://i.imgur.com/4rgOOeh.png)

设置起始关卡载入方式为“总是加载”（这里影响的是地图的可见性，一般之后转移的地图设置会为“蓝图”）。然后把我们的“传送点”放置好，然后开始写逻辑。

首先我们要获取目的地关卡的关卡名称（才知道要传送到哪），所以需要一个变量来保存一个Name，然后从Name找到关卡，加载它，如果被加载了就转移角色。
![2](https://i.imgur.com/oDNcG5M.png)
//这里Load Stream Level函数主要就是把相关的关卡设置为可见，接着经过Timer一秒的延迟后触发检查关卡是否被加载的自定义事件。
Onto是一个接口，用来设置角色的位置：
右键新建一个Blueprints->蓝图接口
我们只需要一个函数和一个输入值作为关卡名。
![3](https://i.imgur.com/dNS4pQk.png)
在人物角色的蓝图里实现这个接口（相当于直接给单个人物设置位置...这很合理）
但是要知道位置的值，所以先在我们自己的Game Mode（服务器单独拥有）里取得出生点的位置。
在Game Mode里创建一个函数用来：

1. 找到所有的出生点
2. 循环判断 出生点是否被占用&&这个出生点的标签是否是关卡的名称。如果都满足，就取得这个出生点的位置和角度，保存到一个临时变量，然后退出循环。（这一步之前我将每个关卡中出生点的标签改成了和关卡同样的名称）
![4](https://i.imgur.com/lcTif7O.png)

ok, 有了位置信息就可以给角色设置了。
先取得我们的Game Mode。最后用Teleport来改变角色位置（这里要实现接口，需要在“类设置里”选择这个接口的蓝图）
![5](https://i.imgur.com/XSdkcKi.png)
传送点就完成了

我们可以加一个人数判断的功能。超过人数，新的客户端无法加入。
在Game Mode里重写SpawnDefaultPawnAtTransForm，可以加一个HUD进行提示
![6](https://i.imgur.com/mSefxH5.png)






