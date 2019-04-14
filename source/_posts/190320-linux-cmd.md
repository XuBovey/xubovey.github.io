---
title: 190320 linux cmd
toc: true
date: 2019-03-20 10:02:08
categories:
tags:
---

## 查看存储空间使用情况
```
//查看当前系统磁盘使用空间
df -h
//查看当前目录文件占用空间大小
du -sh *
```

运行效果如下
```
root@tianRong:/usr# du -sh *
5.7M    bin
112.0K  etc
128.5M  lib
0       local
60.0K   regdefault
1.4M    sbin
6.3M    share

root@tianRong:/usr# df -h
Filesystem                Size      Used Available Use% Mounted on
ubi0:rootfs             104.6M     98.6M      1.3M  99% /
devtmpfs                 60.4M      4.0K     60.4M   0% /dev
tmpfs                    60.6M     80.0K     60.5M   0% /var/volatile
tmpfs                    60.6M         0     60.6M   0% /media/ram
tmpfs                    60.6M    244.0K     60.4M   0% /tmp
ubi0_1                  107.9M     24.0K    103.1M   0% /mnt/user
```

## 重新加载文件系统
```
mount -o remount -rw /
```
Linux 文件权限 变成制度 readonly 解决方法
文件系统加载为只读
```/erc/rcS.d/S01checkout.sh```文件中修改mount语句为```mount -n -o remount,ro /```
修改为```mount -n -o remount, rw /```即为读写模式加载


## su,sudo,退出
终端使用sudo可实现,使用su可进入root把$变成# 
退出：ctrl+D,exit,logout

## ls -Sl或-lS 按大小排序

## 遍历当前文件夹中所有文件夹内文件，输出md5到指定文件夹
```
find ./ ! -name 'checksum' -prune -type f -print0 | xargs -0 md5sum > ./checksum
```
## 打包
```
tar -zcvf rootfs.tar.gz ./rootfs
```
打包过程：
1. 把当前可用的最新文件系统复制到Linux下
2. 把调试中修改过的usr,opt,etc文件夹打包，并复制到Linux下
3. 把修改后的3个文件夹替换，替换后遍历所有文件并做MD5校验
4. 把新的文件系统打包

## 删除文件夹
rm -rf * //删除文件下所有文件，包含子文件夹

## 环境变量删除
export -n

## 查看文件被哪个进程占用
lsof

## QT命令行编译
执行qmake xxx.pro生产makefile，执行make完成编译


## 声音播放，音量调节
aplay ***.mav
alsamixer
aplay -l 可查看声卡信息
aplay -D plughw:0,0 xxx.wav




