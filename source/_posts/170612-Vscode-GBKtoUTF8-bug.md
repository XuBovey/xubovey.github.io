---
title: 170612-Vscode-GBKtoUTF8-bug
toc: true
date: 2017-06-12 10:54:52
categories:
tags:
- Vscode
- GBKtoUTF8
---

# 问题
使用Vscode过程中，会有一些文件是GBK编码格式，打开后是乱码，所以就安装了插件GBKtoUTF8，自动进行转换。  
问题来了，如果一个`*.h`文件是GBK编码格式的，同时这个文件又被一个`*.c`文件给包含了，那么当按住ctrl键，同时移动鼠标到这个头文件的名称上时，会在当前位置打开`*.h`文件进行格式转换，并将转换后的代码覆盖`*.c`文件中的原有代码。

# 解决
需要时再打开GBKtoUTF8功能。^_^，简单粗暴。

