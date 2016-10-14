---
title: Contiki_Makefile-prj
toc: true
date: 2016-10-14 13:18:26
categories:
tags: Contiki, Makefile
---

参考：[鱼竿的传说](http://www.cnblogs.com/chineseboy/archive/2014/07.html)
鱼竿的文档里写的思路很清晰，理解Makefile包含关系，点到为止，代码阅读中遇到宏进行搜索替换。逐层拨开云团迷雾，看清代码的本质，很是受用。

| 版本   | 信息描述                       | 更新日期  |
| :---- | :----------------------------- |:-------- |
| V0.1  | 初稿                           | 16-10-14 |

本篇同样从Makefile入手，了解示例工程中的Makefile的区别和平台上Makefile文件的移植自定义方法。文中有误之处迭代更新吧。

# Makefile
> 其实Makefile种类一说，这个是我自己创的—-其实它们都是makefile文件了。Contiki OS一共分为5类makefile文件：
> * Makefile
> * Makefile.include
> * Makefile.\$(TARGET)
> * Makefile.\$(CPU)
> * Makefile.\$(APP)
其中第一个“Makefile”，我称之为Makefile-prj，因为它与实际工程有关，并且位于项目工厂目录。

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
貌似没什么干货，不过算是简单的真理了。

