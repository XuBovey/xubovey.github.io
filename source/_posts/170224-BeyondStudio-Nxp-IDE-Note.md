---
title: 170224-BeyondStudio Nxp IDE Note
toc: true
date: 2017-02-24 21:31:37
categories: Nxp
tags: BeyondStudio
---


# 简述
最近研究JN5168，需要用到NXP官方提供的开发环境，使用中遇到的小东小西就纪录在这里。

# 问题
## Type 'uint8' could not be resolved
参考官方文档JN-UG-3098，page 25：  
Step 4 Configure the workspace preferences as follows:  
a) Open the Preferences dialogue box by following the menu path
Window>Preferences.
b) In the left tree of the Preferences dialogue box, open the C/C++ entry and click
on the Indexer sub-entry. The right side of the dialogue box is now populated with
the Indexer options.
c) In the section Build configuration for the indexer, select the radio-button
Use active build configuration.
d) Click Apply and then OK.

问题解决。每次新建workspace都需要配置。

## workspace
自定义workspace，不能编译问题。  
解决：  
修改biuld文件夹下的makefile
```
SDK_BASE_DIR       ?= $(abspath ../../../../sdk/$(JENNIC_SDK)/)
```
为
```
SDK_BASE_DIR       ?= $(abspath ../../../$(JENNIC_SDK)/)
```

## import工程后文件夹为空
问题：  
导入`JN-AN-1229.zip`工程时，导入后项目文件夹下只有两个文件`.cproject``.project`。当然也无法编译了，怎么办呢？  
解决：  
* 将`JN-AN-1229.zip`解压到`当前`文件夹;
* 从文件夹中导入工程时，把options->Copy projects into workspace 选项取消；
* 导入后工程的目录里面显示的链接标志，不过可以编译了。

原因：  
不知道。

