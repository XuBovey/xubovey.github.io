---
title: 堆栈bug引起的反思
toc: true
date: 2019-01-15 18:03:26
categories:
tags:
---

## 背景
项目开发中程序运行一段时间后逻辑异常。各种排查，各种加printf。最后定位是vsprintf函数被执行后引起了改变了代码中的某全局变量。胡乱配置，修改一通，无果。最后请教技术大牛，指点迷津`堆栈搞大`

## 搞大堆栈
使用的单片机是STM32，在`startup_stm32f10x_hd.s`文件中有如下配置：
```
Stack_Size      EQU     0x00000800

                AREA    STACK, NOINIT, READWRITE, ALIGN=3
Stack_Mem       SPACE   Stack_Size
__initial_sp
                                                  
; <h> Heap Configuration
;   <o>  Heap Size (in Bytes) <0x0-0xFFFFFFFF:8>
; </h>

Heap_Size       EQU     0x00000400

                AREA    HEAP, NOINIT, READWRITE, ALIGN=3
__heap_base
Heap_Mem        SPACE   Heap_Size
__heap_limit
```
可以发现stack空间是1K, Heap是512字节自然不大。分别进行修改：Heap->1K 仍然错误，之后又修改Stack->2K测试通过。

## 堆栈对数据破坏的隐蔽性
如何发现堆栈引起的错误，想到的有  
* if语句判断变量时应该列举每一种情况，不支持的进行打印或告警，基本发现变量被异常修改；
* 单片机代码进来避免深层调用，避免局部变量过大
* 适当加大堆栈空间

验证发现：后两种都可以解决遇到的问题




