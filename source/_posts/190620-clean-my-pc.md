---
title: 150G的C盘又满了
toc: true
date: 2019-06-20 07:54:11
categories:
tags:
---

## 背景
C盘100G满的时候扩容到了150G，如今又提示不到13G了。。。

## 不能忍
眼睁睁看着C盘变大的时候，尝试过很多种廋身办法都没啥效果。

## 绝地反击
后来发现在C盘取消所有文件隐藏的条件下，选中所有文件，查看大小是100G，这里消失的几十G哪里去了。。。
然后就想着如何查看文件夹大小，接着就找到了工具wiztree，这个是好东西，软件很小，但是能几秒内统计100G的文件大小，并用色块表示文件大小。这样很容易就定位到了一些好几G的大文件夹，找到没用的删除之。
其中有1个文件是N卡驱动缓存文件，5-6个G，使用工具DriverStoreExplorer.v0.10.39完成扫码驱动备份，并删除老版本的驱动

## C:\windows\softwaredistribution过大处理
需要以管理员身份启动cmd
```
    Start>Run
    type cmd and press enter
    type net stop wuauserv and press enter
    type rename c:\windows\SoftwareDistribution softwaredistribution.old and press enter
    type net start wuauserv and press enter
    type exit and press enter 
```

## Hiberfil.sys很大
Hiberfil.sys 是 Windows 休眠功能（Windows Hibernation）将内存数据与会话保存至硬盘、
以便计算机断电重新启动后可以快速恢复会话所需的内存镜像文件。在早期版本的 Windows 中，Hiberfil.sys 
文件的大小等同于物理内存大小；而在 Windows 7 中，Hiberfil.sys 可以在物理内存大小的 50%－100% 的范围自行调整。
因此， Windows 7 的 Hiberfil.sys 大小不一定等同于物理内存大小。
需要以管理员身份启动cmd
```
    cmd
    powercfg -h off //关系休眠，该文件可删除
    powercfg -h on  //如果需要可执行该命令，再次启动休眠
```


## 惨痛的教训
C:\Program Files\目录下有uninstall.exe文件，还有一些数据库文件，数据库相关的文件日期是14年的，可能是什么时候装软件选择的不对没有放到文件夹下。想用uninstall删除，结果悲催了。这个文件居然开始删除adobe和autocad，赶紧任务管理杀掉进程。后来分析uninstall是用来删除文件夹下其他文件的，所以...如果放任不管，我可能就真的悲催了。
就这样重装了adobe，autocaad，7zip，git等问题解决。

