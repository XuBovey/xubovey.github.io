---
title: Contiki中cc26xx之LPM
toc: true
date: 2016-10-26 14:58:04
categories:
- Contiki
- cc26xx
tags:
- cc26xx
- LPM
---

LPM: Low-Power management  
在Contiki中只有CC26XX和CC2538有该特性。  

WUC: Wake-Up Control；
AON: Alway ON;

lpm的使用分析同样按main函数中的主线进行，不过有了前面的分析，会有一些表面上看不到的思路，例如main函数中没有直接调用，而是在其调用的子函数中有相关代码。实际分析是ieee-mode.c的init函数被main函数经netstack_init函数调用。关系如下：  
main->netstack_init->init->lpm_register_module(&cc26xx_rf_lpm_module);

<!--more-->
先包含main函数中的其它lpm相关代码和上面这个进行一个整合：
``` c
int
main(void)
{
    ...
    lpm_init();
    ...
    lpm_register_module(&cc26xx_rf_lpm_module);
    ...
    while(1) {
    ...
    /* Drop to some low power mode */
    lpm_drop();
  }
}
```
# 0. lpm API函数
* void lpm_drop(void); //释放CPU，调用lpm_sleep或deep_selp函数
* void lpm_sleep(void); //进入休眠状态
* void lpm_shutdown(uint32_t wakeup_pin, uint32_t io_pull, uint32_t wake_on); //关闭所有外设，时钟；配置唤醒IO口为wakeup_pin，然后进入deep_sleep。类似于手机的关机。
* void lpm_register_module(lpm_registered_module_t *module);   //注册，添加到链表
* void lpm_unregister_module(lpm_registered_module_t *module); //删除，从链表移除
* void lpm_init(void); //初始化链表，配置IO口沿信号事件触发唤醒
* void lpm_pin_set_default_state(uint32_t ioid);

数据结构：
lpm_registered_module_t：
``` c
typedef struct lpm_registered_module {
  struct lpm_registered_module *next; //下一个
  uint8_t (*request_max_pm)(void);    //返回休眠类型
  void (*shutdown)(uint8_t mode);     //对应模块、单元执行shutdown前需要执行的函数
  void (*wakeup)(void);               //执行wake_up时模块、单元需要执行的函数
  uint32_t domain_lock;
} lpm_registered_module_t;
```
几个宏定义，说明了不同的休眠等级：
``` c
#define LPM_MODE_AWAKE         0
#define LPM_MODE_SLEEP         1
#define LPM_MODE_DEEP_SLEEP    2
#define LPM_MODE_SHUTDOWN      3

#define LPM_MODE_MAX_SUPPORTED LPM_MODE_DEEP_SLEEP
```

# 0.1 lpm_shutdown
该函数在platform下的button-sensor.c文件中，在按键处理函数中当检测到右键被按下时执行如下代码段：
``` c
lpm_shutdown(BOARD_IOID_KEY_RIGHT, IOC_IOPULL_UP, IOC_WAKE_ON_LOW);
```

# 1. lpm_init
```c
void
lpm_init()
{
  list_init(modules_list); //设置modules_list = NULL

  /* Always wake up on any DIO edge detection */
  ti_lib_aon_event_mcu_wake_up_set(AON_EVENT_MCU_WU3, AON_EVENT_IO);
}
```
这里需要简单介绍WUC(Wake-Up Control).

## 1.1 AON_EVENT_MCU_WU3
表示配置的是MCU唤醒事件的事件3，共有4个事件分别为事件0~事件3。AON_EVENT_IO表示任何IO口上的边沿事件，对应手册中的PAD事件。

## 1.2 WUC-唤醒控制器
CC26XX的唤醒控制器根据JTAG，AUX，MCU的输入事件控制power-on的触发。主要用于低功耗状态下的唤醒。唤醒控制器的输入是事件形式的，分三类：JTAG，AUX，MCU。
* JTAG: 有1个唤醒事件;
* AUX : 有3个可编程的唤醒事件;
* MCU : 有4个可编程的唤醒事件. 
事件的编程通过AON_EVENT:AUXWUSEL和AON_EVENT:MCUWUSEL完成。

## 1.3 AON_EVENT寄存器
包含四组：
* MCUWUSEL  : MCU唤醒事件选择配置 
* AUXWUSEL  : AUX唤醒事件选择配置
* EVTOMCUSEL: MCU事件来源选择（AON 相关）
* RTCSEL    : AON_RTC的事件捕捉选择

## 1.4 MCUWUSEL寄存器
该寄存器包含4个字节，每个字节对应一组AON_EVENT事件选择.  
AON_EVENT事件包含IO口的边沿脉冲，RTC，JTAG，AUX_ADC,AUX_TIMER, BAT..其它事件。具体参考TI手册[SWCU117D.pdf](http://www.ti.com/lit/ug/swcu117f/swcu117f.pdf)第246页。

# 2. lpm_register_module
lpm_register_module(&cc26xx_rf_lpm_module);

## 2.1 lpm_register_module函数
lpm_register_module调用list_add(modules_list, module)函数，将新模块添加到modules_list链表上，完成注册。

## 2.2 cc26xx_rf_lpm_module定义
语句中的cc26xx_rf_lpm_module在ieee-mode.c文件中定义。
``` c
LPM_MODULE(cc26xx_rf_lpm_module, request, NULL, NULL, LPM_DOMAIN_NONE);
```
宏展开
``` c
static lpm_registered_module_t cc26xx_rf_lpm_module ={NULL, request, NULL, NULL, LPM_DOMAIN_NONE};
```
其中request为函数，函数体：
``` c
static uint8_t
request(void)
{
  if(rf_is_on()) {
    return LPM_MODE_SLEEP;
  }
  return LPM_MODE_MAX_SUPPORTED; //LPM_MODE_DEEP_SLEEP
}
```
# 3. lpm_drop
函数体如下：
``` c
void
lpm_drop()
{
  uint8_t max_pm;

  /* Critical. Don't get interrupted! */
  ti_lib_int_master_disable();

  max_pm = setup_sleep_mode(); //获取可执行的最高休眠等级

  /* Drop */
  if(max_pm == LPM_MODE_SLEEP) {
    lpm_sleep();
  } else if(max_pm == LPM_MODE_DEEP_SLEEP) {
    deep_sleep();
  }

  ti_lib_int_master_enable();
}
```
因为这个函数是在while(1)中被调用的，所有没有shutdown函数的调用，根据上文分析，shutdown是在按键处理函数中进行的。
那么lpm_sleep和deep_sleep的区别呢：

# 4. lmp_sleep

# 5. deep_sleep

