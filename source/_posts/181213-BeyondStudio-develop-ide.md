---
title: BeyondStudio develop ide
toc: true
date: 2018-12-13 09:00:45
categories:
tags:
---

## 介绍
BeyondStudio for NXP IDE（JN-SW-4141）是NXP的JN516X系列无线微控制器的开发平台。本工具是支持JN516X系列芯片的软件开发包和开发工具链合集，需要先安装，占用约700MB空间。  
JN516x SDK需要第二步安装，用于支持不同的网络协议。

## 安装
* JN-UG-3098[下载](https://pan.baidu.com/s/1ZTZLjXxYRGp_glxZYPUCsQ)  
* JN-SW-4141[下载](https://pan.baidu.com/s/1FrquoxXaSGNlOL6bN1TEvw)  
* JN-SW-4170[下载](https://pan.baidu.com/s/1jlJWhEkCf0LwhXojVjvERA)  
* JN-SW-4170 V1840[下载](https://pan.baidu.com/s/1smR270bZ9KHsLPTG2wqgyA)i7nq  
* JN-SW-4163[下载](https://pan.baidu.com/s/1Mim6SftD87ZuOs38ArzxAg)  
* JN-SW-4168[下载](https://pan.baidu.com/s/1cfz_LnFkfeKG7DK7JcnwGA) 4suq   

根据手册介绍：  
1. BeyondStudio for NXP需要先安装: JN-SW-4141  
2. SDK需要后续安装: JN-SW-4170/4168  

## app.zpscfg
app.zpscfg如何打开呢？其实`JN-UG-3098 Beyond Studio for NXP`文件中已经进行了详细描述。
文件的`1.2.3Installing the ZigBee Plug-ins`章节有着详细的描述。
需要在eclipse环境下安装2个插件分别是`ZPS Configuration Editor`和`JenOS Configuration Editor`,
具体请参考文档[JN-UG-3098](https://www.nxp.com/docs/en/user-guide/JN-UG-3098.pdf)


## 工程导入
[异常处理](https://community.nxp.com/docs/DOC-340028)中的2.Project Directory


## 其他问题参考
[BeyondStudio Nxp IDE Note](https://xubovey.github.io/2017/02/24/170224-BeyondStudio-Nxp-IDE-Note/)
