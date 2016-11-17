---
title: Contiki Energest
toc: true
date: 2016-10-26 14:34:38
categories:
- Contiki
tags:
- Contiki
- Energest
---

Contiki 的设计目的是在极端低功耗的系统中运行，这些系统甚至可能需要只用一对AA电池能够工作许多年。Contiki 为辅助这些低功耗系统的开发提供了功耗估计和功耗分析机制Energest。  
代码分析中多次遇到这个活力评估相关的函数，现在来做简单分析。  
遇到的代码中基本上是这样的：
``` c
  energest_init();
  ENERGEST_ON(ENERGEST_TYPE_CPU);

  ENERGEST_ON(ENERGEST_TYPE_IRQ);

  ENERGEST_OFF(ENERGEST_TYPE_IRQ);
  ...
```
<!--more-->
# 1. energest_init
这个比较简单，代码如下：
``` c
void
energest_init(void)
{
  int i;
  for(i = 0; i < ENERGEST_TYPE_MAX; ++i) {
    energest_total_time[i].current = energest_current_time[i] = 0;
    energest_current_mode[i] = 0;
  }
#ifdef ENERGEST_CONF_LEVELDEVICE_LEVELS
  for(i = 0; i < ENERGEST_CONF_LEVELDEVICE_LEVELS; ++i) {
    energest_leveldevice_current_leveltime[i].current = 0;
  }
#endif
}
```
就是将相关数组中的变量初始化为0。简化分析写做time数组和mode数据。

# 2. ENERGEST_ON
这个宏展是这样定义的：
``` c
#define ENERGEST_ON(type)  do { \
                           /*++energest_total_count;*/ \
                           energest_current_time[type] = RTIMER_NOW(); \
			   energest_current_mode[type] = 1; \
                           } while(0)
```                           
以ENERGEST_ON(ENERGEST_TYPE_CPU);为例进行展开如下；
``` c
do { 
     /*++energest_total_count;*/ 
     energest_current_time[ENERGEST_TYPE_CPU] = RTIMER_NOW(); 
	 energest_current_mode[ENERGEST_TYPE_CPU] = 1; 
} while(0)
```
这里将time数组中存入当前的RTIMER时间，并将mode置1。而ENERGEST_TYPE_CPU是个枚举变量成员，定义如下：
``` c
enum energest_type {
  ENERGEST_TYPE_CPU,
  ENERGEST_TYPE_LPM,
  ENERGEST_TYPE_IRQ,
  ENERGEST_TYPE_LED_GREEN,
  ENERGEST_TYPE_LED_YELLOW,
  ENERGEST_TYPE_LED_RED,
  ENERGEST_TYPE_TRANSMIT,
  ENERGEST_TYPE_LISTEN,

  ENERGEST_TYPE_FLASH_READ,
  ENERGEST_TYPE_FLASH_WRITE,

  ENERGEST_TYPE_SENSORS,

  ENERGEST_TYPE_SERIAL,

  ENERGEST_TYPE_MAX
};
```

# 3. ENERGEST_OFF
宏定义如下；
``` c
#define ENERGEST_OFF(type) if(energest_current_mode[type] != 0) do {	\
                           energest_total_time[type].current += (rtimer_clock_t)(RTIMER_NOW() - \
                           energest_current_time[type]); \
			               energest_current_mode[type] = 0; \
                           } while(0)
```
ENERGEST_OFF(ENERGEST_TYPE_IRQ);展开后：
``` c
if(energest_current_mode[ENERGEST_TYPE_IRQ] != 0) 
  do {
    energest_total_time[ENERGEST_TYPE_IRQ].current += 
                       (rtimer_clock_t)(RTIMER_NOW() 
                     - energest_current_time[ENERGEST_TYPE_IRQ]); 
	energest_current_mode[ENERGEST_TYPE_IRQ] = 0; 
  } while(0)

```
这里做的是将RTIMER_NOW得到的时间减去执行ON时存入current_time的时间，并将差值存入total_time中。也就是说这个total_time中存的是某类代码的活跃时间。  
