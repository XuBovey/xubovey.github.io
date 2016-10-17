---
title: Contiki_Makefile-prj
toc: true
date: 2016-10-14 13:18:26
categories: 
- Contiki
tags: 
- Contiki
- Makefile
---

参考：[鱼竿的传说](http://www.cnblogs.com/chineseboy/archive/2014/07.html)
鱼竿的文档里写的思路很清晰，理解Makefile包含关系，点到为止，代码阅读中遇到宏进行搜索替换。逐层拨开云团迷雾，看清代码的本质，很是受用。

本篇同样从Makefile入手，了解示例工程中的Makefile的区别和平台上Makefile文件的移植自定义方法。文中有误之处迭代更新吧。

<!--more-->

# Makefile
> 其实Makefile种类一说，这个是我自己创的—-其实它们都是makefile文件了。Contiki OS一共分为5类makefile文件：
> * Makefile 位于工程目录下，文件中包含Makefile.include
> * Makefile.include 执行make命令的时候会展开Makefile.include,其中根据make命令的参数判断TARGET是什么，不指定则采用默认值，选择完后会展开Makefile.\$(TARGET)。
> * Makefile.\$(TARGET) 这个target其实就是指定用的开发平台是什么，再简单点说就是哪个开发板，其中会包含有哪些外设资源的。同时也会设定使用的处理器是什么Makefile.\$(CPU)，相同处理器的可以有不同的开发板，所以target与CPU就分开了。
> * Makefile.\$(CPU) CPU会指定采用什么样的编译器，会包含一些底层的东西。
> * Makefile.\$(APP) APP是Makefile.include中指定的，这些app由contiki提供，当然了，也可以用户自定定义的。就像操作系统上的应用程序一样。  
其中第一个“Makefile”，我称之为Makefile-prj，因为它与实际工程有关，并且位于项目工程目录下。更详细的介绍可参见./contiki/doc/build-system.txt。

# Makefile-prj
以helloworld和CC26XX两个例程为例，比较Makefile的区别。  
helloworld目录下：
```
./examples/heloworld/Makefile

CONTIKI_PROJECT = hello-world
all: $(CONTIKI_PROJECT)

CONTIKI = ../..
include $(CONTIKI)/Makefile.include
```

CC26XX目录下：
```
./examples/CC26XX/Makefile

DEFINES+=PROJECT_CONF_H=\"project-conf.h\"

CONTIKI_PROJECT = cc26xx-demo
all: $(CONTIKI_PROJECT)

CONTIKI_WITH_IPV6 = 1

CONTIKI = ../..
include $(CONTIKI)/Makefile.include
```
比较发现CC26XX目录下的Makefile文件中多了两行
```
DEFINES+=PROJECT_CONF_H=\"project-conf.h\"
CONTIKI_WITH_IPV6 = 1
```
CONTIKI_WITH_IPV6应该是用于指定包含IPv6协议栈的。不做展开。  
那么project-conf.h文件中是什么呢：
```
/*---------------------------------------------------------------------------*/
#ifndef PROJECT_CONF_H_
#define PROJECT_CONF_H_
/*---------------------------------------------------------------------------*/
/* Disable button shutdown functionality */
#define BUTTON_SENSOR_CONF_ENABLE_SHUTDOWN    0
/*---------------------------------------------------------------------------*/
/* Enable the ROM bootloader */
#define ROM_BOOTLOADER_ENABLE                 1
/*---------------------------------------------------------------------------*/
/* Change to match your configuration */
#define IEEE802154_CONF_PANID            0xABCD
#define RF_CORE_CONF_CHANNEL                 25
#define RF_BLE_CONF_ENABLED                   1
/*---------------------------------------------------------------------------*/
#endif /* PROJECT_CONF_H_ */
/*---------------------------------------------------------------------------*/
```
看样子是配置了一些用户自定义的信息，主要与外设相关。

觉得不够充分，那么就再看看2530和2538的吧：
```
/examples/CC2530dk/Makefile
CONTIKI_PROJECT = hello-world blink-hello timer-test sensors-demo

all: $(CONTIKI_PROJECT) 

CONTIKI = ../..
CONTIKI_WITH_RIME = 1
include $(CONTIKI)/Makefile.include
```
```
/examples/CC2538dk/Makefile
DEFINES+=PROJECT_CONF_H=\"project-conf.h\"
CONTIKI_PROJECT = cc2538-demo

all: $(CONTIKI_PROJECT)

CONTIKI = ../..
CONTIKI_WITH_RIME = 1
include $(CONTIKI)/Makefile.include
```
```
/examples/CC2538dk/project_conf.h
#ifndef PROJECT_CONF_H_
#define PROJECT_CONF_H_

#define NETSTACK_CONF_RDC     nullrdc_driver

#endif /* PROJECT_CONF_H_ */
```

嗯，看样子就是一些配置了。至于怎么配置暂时还不能很自如的说明。  
但是除了project_conf.h文件，就是CONTIKI_PROJECT变量的配置了。
这个变量很简单，就是Makefile所在文件夹，当前文件夹中有几个*.c文件全部写在后面就可以了。  
貌似没什么干货，不过算是简单的整理了。

# 参考
[contiki makefile框架分析 < contiki学习之一 >](http://www.cnblogs.com/chineseboy/p/3844981.html)  
[build-system.txt](https://github.com/contiki-os/contiki/blob/master/doc/build-system.txt) 

***
