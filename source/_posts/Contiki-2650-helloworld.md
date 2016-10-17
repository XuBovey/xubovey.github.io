---
title: Contiki CC2650 HelloWorld
toc: true
date: 2016-10-12 16:33:14
categories: Contiki, 技术
tags: Contiki, CC2650
---


# 使用开发板完成HelloWorld的串口输出
## [环境搭建](http://www.contiki-os.org/start.html)
* 基于win10开发，所以要安装虚拟机VMware；
* 官网下载ubuntu虚拟机镜像；
* 启动虚拟机, 输入密码`user`完成登陆；
* 下载最新版contiki系统源文件；
* 下载CC26xx系列固件。

<!--more-->

### VMware安装
自行百度完成。

### 下载ubuntu
[下载地址](https://sourceforge.net/projects/contiki/files/Instant%20Contiki/)
一共2.2G左右。 用户名：；密码。 

### 下载Contiki3.0
3.0版本是当前最新版。在虚拟机中打开终端，进入`home`目录，执行如下操作。
```
git clone https://github.com/contiki-os/contiki.git​git
```
下载完成后，虚拟机自带的contiki可以删除。

### 下载CC26XX固件
进入contiki->cpu->cc26xx-cc13xx>lib->cc26xxware目录，
执行如下操作：
```
git clone https://github.com/contiki-os/cc26xxware.git
```
等待完成固件的下载。

至此环境搭建完成。

## 编译
ubuntu系统中，终端界面下打开contiki->example->hello-world。  
分布执行以下命令：
```
make clean
make TARGET=srf06-cc26xx BOARD=srf06/cc26xx
```
在hello-world文件加下可以得到编译处的hex，bin文件。
## 下载
将二进制文件复制到win10下，
启动`FlashProgrammer 2`(1不支持26xx)。  
电脑连接开发板串口、仿真器后可在`FlashProgrammer 2`下看到连接2650设备。
在`FlashImages`选项中选择signal,并通过Browse按钮选择复制出的二进制文件。 
Actions选型中保留默认配置，单击`Play`按钮,即可完成下载。

打开串口终端，选择与开发板串口对应的串口-打开。然后按下开发板复位键，即可在终端上看到打印信息。
```
【2016-10-12 16:31:41:028】Starting Contiki-3.x-2881-g3f4436b
With DriverLib v0.46593
TI SmartRF06EB + CC26xx EM
 Net: sicslowpan
 MAC: CSMA
 RDC: ContikiMAC, Channel Check Interval: 16 ticks
 RF: Channel 25
 Node ID: 15107
```

***
