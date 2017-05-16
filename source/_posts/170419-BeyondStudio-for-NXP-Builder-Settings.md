---
title: BeyondStudio for NXP Builder Settings
toc: true
date: 2017-04-19 13:50:34
categories:
tags: 
- BeyondStudio
- Builder
---

# 摘要
在使用`BeyondStudio`过程中，发现修改Makefile中的部分配置信息无效，不能生产相应的文件，例如配置了`JENNIC_CHIP=JN5168`,但是实际生成的文件却是`JENNIC_CHIP=JN5169`。
<!---more--->

# 解决办法
* 打开`Project`->`Properties`->`C/C++ Build`->`Builder Settings`
* 勾选`Use default build command`

