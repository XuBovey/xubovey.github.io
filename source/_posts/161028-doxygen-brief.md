---
title: Doxygen 简介
toc: true
date: 2016-10-28 09:40:31
categories:
- Doxygen
- Doc
tags:
- Doxygen
---

再也不用担心软件不会写文档了，再也不用担心软件文档写着麻烦了。之前见到过doxygen这个东西，只是没留意，昨天研究Contiki的时候发现了这个[contiki说明](http://dak664.github.io/contiki-doxygen/)，就搜了下原来正式自动文档生成工具。那还不迅速研究下....

<!--more-->
首先是这个工具是做什么用的：
> Doxygen是一种开源跨平台的，以类似JavaDoc风格描述的文档系统，完全支持C、C++、Java、Objective-C和IDL语言，部分支持PHP、C#。注释的语法与Qt-Doc、KDoc和JavaDoc兼容。Doxygen可以从一套归档源文件开始，生成HTML格式的在线类浏览器，或离线的LATEX、RTF参考手册。  

这里说的RTF文件是可以用world打开的哦。

# 不同平台的安装
这里没有，请移步[Downloads](http://www.stack.nl/~dimitri/doxygen/download.html)  
[linux doxygen 的安装和使用](http://blog.csdn.net/blood008/article/details/6567169)

# C语言注释规范
[参考官方说明](http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html#cppblock)  
首先需要想清楚代码中需要哪些注释，软件说明文档要对软件进行哪些介绍。
* 包含哪些函数，这些函数的功能介绍，入口参数，出口参数介绍；
* 包含哪些数据结构，数据结构的介绍，以及详细说明；
* 特殊变量的说明；
* 函数调用关系说明
* ...

理想是很丰满的，现实也很赞的。
这些都能实现：
## c代码注释规则
其实这里不该说是c代码的规则，应当说是c类代码：包含C/C++/C#/Objective-C/PHP/Java。注释中包含`brief`做简单介绍，可以针对函数、数据结构、变量；注释中包含`detailed`做详细介绍，可以针对函数、数据结构、变量。
* brief 简单介绍，建议简短，要惜字如金；
* detailed 详细介绍，要对描述对象做足够充分的介绍。

对brief和detailed的描述格式有很多种，这里只说明并采用JavaDoc格式的，可以参考[Javadoc 的书写格式和javadoc命令的使用](http://blog.csdn.net/yongping8204/article/details/1796667)：
* brief
``` c
/**
 * \brief It is a brief
 */
```
注意这里一开始是2个`*`，如果是1个就是普通注释，不会出现在文档里。

* detailed
``` c
/**
 * ... text ...
 */
```

* brief、param、detailed
如果是一个包含参数的函数呢，有简介，参数说明，详细说明，举了例子吧：
``` c
/**
 * \brief Set the device name to use with the BLE advertisement/beacon daemon
 * \param interval The interval (ticks) between two consecutive beacon bursts
 * \param name The device name to advertise
 *
 * If name is NULL it will be ignored. If interval==0 it will be ignored. Thus,
 * this function can be used to configure a single parameter at a time if so
 * desired.
 */
void rf_ble_beacond_config(clock_time_t interval, const char *name);
```
这里If name...是详细说明部分，它上面有个空行。这个是必须的。

* 变量后的注释
在struct, union, class, or enum的注释中，有可能注释出现在代码后面。怎么做呢：
``` c
int var; /**< Detailed description after the member */
```
`*< `注意这里有个空格的。

## 标记示例
[转载](https://www.ibm.com/developerworks/cn/aix/library/au-learningdoxygen/)

> 体内描述：类、结构、联合体和名称空间等 C++ 元素都有自己的标记。比如<\class>、<\struct>、<\union> 和 <\namespace>。
> 示例包含用于四种元素的标记：函数标记（<\fn>）、函数参数标记（<\param>）、变量名标记（<\var>）、用于 #define 的标记（<\def>）以及用来表示与一个代码片段相关的问题的标记（<\warning>）。
``` c
/** \file globaldecls.h
      \brief Place to look for global variables, enums, functions
           and macro definitions
 */

/** \var const int fileSize
      \brief Default size of the file on disk
 */
const int fileSize = 1048576;

/** \def SHIFT(value, length)
      \brief Left shift value by length in bits
 */
#define SHIFT(value, length) ((value) << (length))

/** \fn bool check_for_io_errors(FILE* fp)
      \brief Checks if a file is corrupted or not
      \param fp Pointer to an already opened file
      \warning Not thread safe!
 */
bool check_for_io_errors(FILE* fp);
```
doxygen的输出：

---
**Macros**  
#define SHIFT(value, length)   ((value) << (length))  
&emsp;&emsp;&emsp;&emsp; Left shift value by length in bits.

**Functions**  
bool check_for_io_errors (FILE *fp)  
&emsp;&emsp;&emsp;&emsp; Checks if a file is corrupted or not.

**Variables**  
const int fileSize = 1048576;  
&emsp;&emsp;&emsp;&emsp; Default size of the file on disk.  

**Parameters**  
&emsp;&emsp;&emsp;&emsp;fp: Pointer to an already opened file

**Detailed Description**  
Place to look for global variables, enums, functions and macro definitions.

---
这里其实是doxygen对注释部分进行了解析，得到以上内容。

## 函数调用关系图、UML等  
doxygen +  Graphviz  
参考文件： 
[用doxygen+graphviz自动化生成代码文档（附详细教程）](http://www.cnblogs.com/tianzhijiexian/p/4392924.html)
