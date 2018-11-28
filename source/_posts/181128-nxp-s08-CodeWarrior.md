---
title: CodeWarrior开发环境搭建
toc: true
date: 2018-11-27 22:59:06
categories:
tags:
---

# 下载
CodeWarriorV6.3版本为32位版本，需要安装在32位机器或虚拟机上，笔者是装在虚拟机上的。  
下载地址：[CodeWarriorV6.3]()
压缩包中包含4给文件分别是：  
* CW_MCU_V6.3--S08开发软件 - 软件安装包
* MC9S08AC60.pdf - 芯片手册
* USBDM_4_10_6_80_Win.msi - Flash编程工具
* USBDM_Drivers_1_2_0_WinXP_x32.msi - 编程器驱动

# 安装
需要依次安装软件安装包，编程工具和编程器驱动

# 打开工程
## 代码下载
从gitlab下载最小版本的代码
## 打开工程
直接运行上一步下载的代码文件中的"WSD_010",即可打开文件
## 编译与下载
1. 快捷键“F7”可直接编译或d安吉工具栏"make"按钮进行编译  
2. 如需将新的程序下载到单片机需要单击工具栏的"debug"按钮





