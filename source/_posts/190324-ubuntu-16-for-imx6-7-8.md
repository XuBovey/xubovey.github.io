---
title: '190324-ubuntu 16 for imx6,7,8'
toc: true
date: 2019-03-24 08:24:01
categories:
tags:
---

## boot from sd
[参考](https://blog.csdn.net/kongzhongloucsdn/article/details/54016248)
SD 卡启动是开发板系统启动方式的一种。 SD 系统启动卡共有 FAT32、 EXT3 两个格式分区，还包含 RAW 格式的无名分区。其中 FAT32 格式分区在 Windows 系统下可见，EXT3 格式分区在 Windows 系统下不可见，两分区在 Linux 系统下均可见。无名分区在Windows 和 Linux 操作系统下均不可见。 无名分区存放 u-boot.ais， FAT32 格式分区存放内核文件 uImage、系统启动脚本等文件， EXT3 格式分区存放文件系统。 
## 如何做个SD卡启动卡
### step1 安装mkimage,搭建环境
```
sudo apt-get install uboot-mkimage
```
提示
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package uboot-image
root@ubuntu:/# apt-get install uboot-mkimage
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package uboot-mkimage is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  u-boot-tools:i386 u-boot-tools

```
重新输入命令
```
sudo apt-get install u-boot-tools 
```
安装好后命令行输入`mkimage`
```
Usage: mkimage -l image
          -l ==> list image header information
       mkimage [-x] -A arch -O os -T type -C comp -a addr -e ep -n name -d data_file[:data_file...] image
          -A ==> set architecture to 'arch'
          -O ==> set operating system to 'os'
          -T ==> set image type to 'type'
          -C ==> set compression type 'comp'
          -a ==> set load address to 'addr' (hex)
          -e ==> set entry point to 'ep' (hex)
          -n ==> set image name to 'name'
          -d ==> use image data from 'datafile'
          -x ==> set XIP (execute in place)
       mkimage [-D dtc_options] [-f fit-image.its|-F] fit-image
          -D => set all options for device tree compiler
          -f => input filename for FIT source
Signing / verified boot not supported (CONFIG_FIT_SIGNATURE undefined)
       mkimage -V ==> print version information and exit
Use -T to see a list of available image types
```

### step2 SD卡启动脚本制作
* 将 SD 卡格式化成无名分区（ RAW 格式）、 boot 分区（ FAT32 格式）和 rootfs分区（ EXT3 格式）。
* 拷贝镜像所在目录相关文件到 SD 卡对应分区。
* 在 boot 分区生成 SD 卡启动脚本源文件和 SD 卡启动脚本镜像。

### step3 把镜像等文件复制到sd卡上





