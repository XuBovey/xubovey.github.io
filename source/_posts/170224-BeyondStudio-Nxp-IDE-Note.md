---
title: BeyondStudio Nxp IDE Note
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
* Open the Preferences dialogue box by following the menu path Window>Preferences.
* In the left tree of the Preferences dialogue box, open the C/C++ entry and click on the Indexer sub-entry. The right side of the dialogue box is now populated with the Indexer options.
* In the section Build configuration for the indexer, select the radio-button Use active build configuration.
* Click Apply and then OK.

问题解决。每次新建workspace都需要配置。

### 有时候执行完上面的操作后仍然有问题
[LittleBoy's Blog](http://blog.163.com/rainsmell_/blog/static/212827113201431605936633/)找到如下解决办法：  
``` 
也不知道这算不算是一个bug，即便是添加了所有object所依赖的head files，include path也完整，依然会出现这个问题。refresh工程，重启Eclipse也无济于事。
Google了一番,终于在stackoverflow里找到了解决办法：  
http://stackoverflow.com/questions/10041453/eclipse-c-type-could-not-be-resolved-error-even-though-build-is-successful  
记录一下，Project -> C/C++ index -> Freshen All Files，问题解决。
```

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
暂时不清楚。

## include文件不够
问题：  
缺少`C:\NXP\bstudio_nxp\sdk\JN-SW-4170\Components\ZigbeeCommon\Include`

解决： 
在`项目右键->properties -> general -> paths and symbols -> include` 点击add，添加如上路径即可。
