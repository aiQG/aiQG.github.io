---
layout: post
tag: CTF
author: aiQG_
---

macOS上对于第三方软件，一般不能直接动态调试(权限)

//run的时候lldb会报 `error: process exited with status -1 (Error 1)`

需要进行一下步骤

先查看权限

`codesign -d --entitlements :- [yourAPP]`

将当前权限配置导出

`codesign -d --entitlements :- [yourAPP] >> temp.plist`

在temp.plist中添加

```
<key>com.apple.security.get-task-allow</key>
<true/>
```

保存

找到系统上安装的所有签名身份的列表(需要开发者账号)

`security find-identity -v -p codesigning` 

重新签名

`codesign --sign "Apple Development: ***@***.com (***)" -f --timestamp --options=runtime --entitlements temp.plist [yourAPP] `

签名完成，可以调试

---

静态分析的过程在[这里](/2019/11/27/macOS逆向-之-你大概需要一个带bar的mac.html)

这里用动态分析的方法验证一下静态分析


首先按照上面给出的步骤加上权限

然后在 `sub_1000034C0` 开始处下断点, 可以发现每次子弹与敌人接触或者玩家与敌人接触就会触发断点，确定这是一个判断碰撞的函数

将断点下在 `0x100003974` (cmp     qword ptr [rbx+rax], 114514)处，可以发现每次击中敌人会被断下，并且rbx+rax处的值比之前增加1

从而很快速地就能找到保存分数的地方

将断点下在 `0x1000037c8` (xor    r12b, r13b)处，运行，当击中一个敌人后被断了下来
之后mov    byte ptr [rax + rdx + 0x20], r12b 将结果保存在了一个数组中

![flagArr](https://i.imgur.com/e18ICXp.png)

继续调试发现每次断在xor    r12b, r13b时，r12b是数组中的下一个元素，r13b是分数，当最后一个元素xor之后，程序重新回到第0个元素进行xor操作

---

动态能够分析出静态时不是很清楚的一些变量，当然完全动态分析很容易就会"晕"

总之，动态分析还是要结合静态分析，



---

>"class-dump提供了可以落脚的小屋，但要走出这片森林，还需要一张地图和一个指南针——它们就是IDA和LLDB。这两款工具就像两座挡在我们面前的大山，绝大多数逆向工程初学者都没能成功翻越它们，爬到半山腰就打道回府了，而翻越大山的人们顺利跨过逆向工程的门槛，欣赏到了别样的风景。梦想还是要有的，万一实现了呢？我们鼓起勇气，试试看能不能征服它们。"
摘录来自: 沙梓社 吴航 著. "iOS应用逆向工程(第2版)pdf"



