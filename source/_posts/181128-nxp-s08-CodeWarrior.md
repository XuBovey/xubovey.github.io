---
title: CodeWarrior开发环境搭建与程序下载
toc: true
date: 2018-11-27 22:59:06
categories:
tags:
---

## 下载
CodeWarriorV6.3版本为32位版本，需要安装在32位机器或虚拟机上，自己是装在虚拟机上的。  
下载地址：[CodeWarriorV6.3](https://pan.baidu.com/s/1KxLyj4M1UdU38vvc_VyKog), 提取码：7um4
压缩包中包含4给文件分别是：  
1. CW_MCU_V6.3--S08开发软件 - 软件安装包  
2. MC9S08AC60.pdf - 芯片手册  
3. USBDM_4_10_6_80_Win.msi - Flash编程工具  
4. USBDM_Drivers_1_2_0_WinXP_x32.msi - 编程器驱动  

注意：
安装包是32位的，建议虚拟机安装XP，不要折腾去找64位版本。应该是仿真器驱动只能是xp-32位

## 安装
需要依次安装软件安装包，编程工具和编程器驱动

## 打开工程

### 代码下载
从gitlab下载cloud_coffee_S9S08DZ60分支代码

### 打开工程
直接运行上一步下载的代码文件中的"WSD_010",即可打开文件

## 编译与下载
1. 快捷键“F7”可直接编译或单击工具栏"make"按钮进行编译  
2. 如需将新的程序下载到单片机需要单击工具栏的"debug"按钮

## 仿真器与板相连
仿真器与板之间只需要4根线即可烧录程序，分别是`VCC,GND,RST,BKG`  
板子上的IO描述在背面，仿真器上的IO描述在正面，对应连接即可



