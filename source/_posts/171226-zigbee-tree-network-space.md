---
title: zigbee-tree-network-space
toc: true
date: 2017-12-26 13:27:55
categories:
- zigbee
tags:
- zigbee
- tree network
---

参考：[Zigbee Tree Routing - How It Works and Why It Sucks](https://archive.freaklabs.org/index.php/blog/zigbee/zigbee-tree-routing-how-it-works-and-why-it-sucks.html)

研究发现博文中Cm!=Rm情况下的公式存在问题。

描述树形网络可用3个重要参数：
Dm：最大网络深度
Cm：最大子节点数量
Rm：子节点中最大路由数量, Cm>=Rm

先看简单例子：
Dm = 2, Cm = 3, Rm = 3
Dm   Cm=Rm
0    3^0=1
1    3^1=3
2    3^2=9
可以看出网络规模等于：  
```sum(Rm^n); n={0..Dm}```

但是如果Cm>Rm，例如：
```
Dm = 2, Cm = 7, Rm = 3
0
1 2 3 4 5                 13                    21
        6 7 8 9 10 11 12  14 15 16 17 18 19 20  22 23 24 25 26 27 28

Dm    Rm     Cm-Rm=4
0     3^0=1  0
1     3^1=3  4*3^0=4
2     3^2=9  4*3^1=12
```
可以看出网络规模为1+3+4+9+12=29等于: 

```sum(Rm^n); n={0..Dm} + sum((Cm-Rm)*Rm^n); n={0..Dm-1}```  
整理后：

```1+sum(Cm*Rm^n); n={0..Dm-1}```  

Dm = 2, Cm = 7, Rm = 3带入公式；
1 + 7*1 + 7*3 = 29


