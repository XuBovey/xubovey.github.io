---
title: Contiki Rime协议分析-abc
toc: true
date: 2016-11-02 16:37:54
categories:
- Contiki
- Rime
tags:
- Contiki
---

# 0 背景
在研究rime过程中使用了example中的rime文件夹下的例程，中间遇到的问题进行记录。

<!--more-->

# 1 make出错
在rime例程文件夹下执行如下命令，以错误退出。
``` c
make TARGET=srf06-cc26xx BOARD=srf06/cc26xx
```
后来分析原因是里面有多个历程，复制rime到新的文件夹，将没用的c文件删除，哪些没用？可以看makefile文件中`all:`后面的名称对应的c文件，除了example-abc.c外的所有文件。

# 2 接收不到数据
接收方式：开发板+仿真器+smartrf radio 2.4.3
可以接收到数据但是有溢出警告。
警告没关系，看看数据对不对：
* 发送的内容：`Hello`  
* 对应的HEX：`48 65 65 60 6f`  

smartrf radio接收到的十六进制数据：  
```
16:58:33.737 | 16792 | 0e cd ab ff ff 03 3b 80 00 48 65 6c 6c 6f 00  |  -39
```
可以发现其中包含`48 65 65 60 6f`，看来数据正确，只是两种格式不一样而已，所以报错的。

# 3 example-abc最终发送了哪些东西
要说过程的话还是这个图片介绍的清楚：[转载](http://blog.sina.com.cn/s/blog_686ee2910102vvf9.html)  

![](http://oefaano2o.bkt.clouddn.com/blogimages/images/161102-Contiki-rime-abc/161102-Contiki-rime-abc-01.png-blog)
>如图2所示，描述了Contiki中整个无线传输链路基本结构，主要包含了：网络层（NETSTACK_NETWORK）、链路层安全驱动（link layer security driver, NETSTACK_LLSEC）、MAC层（Media Access Control, NETSTACK_MAC）,RDC层（Radio Duty Cycling，NETSTACK_RDC）以及物理层（NETSTACK_RADIO）。  
在图2的左边一列为对应不同层次，对应需要实现的驱动接口结构，右边为各个层次之间相互的调用情况（只提取了最重要的部分，相关处理没有细化）。  
其中网络层主要完成相关无线网络的连接、组网等情况，链路层安全驱动用于保证整个传输链路的数据安全，MAC层完成物理媒介的访问控制，RDC层用于控制无线电电路的开断，可以有效的控制无线电部分的耗电情况，RADIO层实现了不同无线媒介底层访问驱动，完成整个无线链路的访问与操作。  


