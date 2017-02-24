---
title: 17224-bstudio_nxp-User Workspace config
toc: true
date: 2017-02-24 20:46:44
categories: NXP
tags: bstudio_nxp
---

# 背景
最近要搞JN5168的开发，下载了官网的历程，搭建了官方提供的bstudio_nxp环境，默认配置下，导入官方demo，编译成功，但是导入到自定义`workspace`路径时就不能成功。

# 原因
简单了解发现，工程中的C文件包涵的头文件都没有。研究build文件夹下的makefile发现有如下语句：
``` makfile
SDK_BASE_DIR       ?= $(abspath ../../../../sdk/$(JENNIC_SDK)/)
APP_BASE            = $(abspath ../..)
APP_BLD_DIR         = $(APP_BASE)/$(TARGET)/Build
APP_SRC_DIR         = $(APP_BASE)/$(TARGET)/Source
APP_COMMON_SRC_DIR  = $(APP_BASE)/Common/Source
```
可以发现`SDK_BASE_DIR`经过路径设定指向了`sdk/JN-SW-4163`,而`JN-SW-4163`中放的是板级支持包，正是工程中c文件内缺少的头文件存放的位置。  
分析默认配置发现，这些路径是在安装路径下可用的，而makefile用的是当前路径。所以呢最简单的办法是修改`SDK_BASE_DIR`变量指向`JN-SW-4163`所在路径。参考
```
#SDK_BASE_DIR       ?= $(abspath /C/NXP/bstudio_nxp/sdk/$(JENNIC_SDK)/)
```

如果只想指向自定义的路径呢？？  
有点小麻烦...不过还是有办法的！这么修改变量`SDK_BASE_DIR`  
```
SDK_BASE_DIR       ?= $(abspath ../../../$(JENNIC_SDK)/)
```
之后把对应的板级支持包拷贝到用户设定的路径`workspace`下。但是还没完...  
因为板级支持包下有`config.mk`文件，内部制定了编译相关的参数，而指向的路径还是在安装环境下，也是通过SDK_BASE_DIR变量访问的，所以还需要把板级支持包同路径的`tools`文件复制到用户`workspace`下。想想都好麻烦.

# 结论
所以还是改Makefile比较简单。
```
#SDK_BASE_DIR       ?= $(abspath /C/NXP/bstudio_nxp/sdk/$(JENNIC_SDK)/)
```
不过就用官方默认配置还是最方便的选择。

