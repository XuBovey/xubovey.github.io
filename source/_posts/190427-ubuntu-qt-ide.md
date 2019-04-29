---
title: 190427-ubuntu-qt-ide
toc: true
date: 2019-04-27 13:16:28
categories:
tags:
---

## 参考

[Ubuntu--搭建QT开发环境](https://blog.csdn.net/meteor_s/article/details/79307496)

## 各种装
```
sudo apt-get install qt4-dev-tools		//QT资源包
sudo apt-get install qt4-qtconfig		//配置工具
sudo apt-get install qt-demos			//官方案例源代码
sudo apt-get install qtcreator			//IDE
```
安装过程出现依赖项问题，解决办法：
```
sudo aptitude install qtcreator
```
安装过程遇到`libclang-3.6 libobjc-5-dev libobjc4 `没有安装，输入`n`,切换解决方案。部分库版本过高，选择降版本处理。

安装完成后执行
```
qmake -v
# QMake version 2.0.1a
Using Qt version 4.8.7 in /usr/lib/x86_64-linux-gnu
```

## 编译工具
1. 下载poky
下载路径：https://pan.baidu.com/s/11WdTVIuiZNdrW1f5QjYAQA
提取码：9zhv
2. 安装： sh xx.sh
3. 配置环境变量，在/opt/poky/1.4.3路径下执行 source enviroment-setup-arm5te-poky-linux-gnuabi


