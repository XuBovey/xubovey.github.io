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
PRCM: Power and clock management 
PW: Power Down

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
函数体挺简单的：
``` c
void
lpm_sleep(void)
{
  ENERGEST_SWITCH(ENERGEST_TYPE_CPU, ENERGEST_TYPE_LPM);

  /* We are only interested in IRQ energest while idle or in LPM */
  ENERGEST_IRQ_RESTORE(irq_energest);

  /* Just to be on the safe side, explicitly disable Deep Sleep */
  HWREG(NVIC_SYS_CTRL) &= ~(NVIC_SYS_CTRL_SLEEPDEEP);

  ti_lib_prcm_sleep();

  /* Remember IRQ energest for next pass */
  ENERGEST_IRQ_SAVE(irq_energest);

  ENERGEST_SWITCH(ENERGEST_TYPE_LPM, ENERGEST_TYPE_CPU);
}
```
## 4.1 sleep之前之后
这里又出现了ENERGEST_SWITCH，先来看看执行sleep之前和之后都做了什么吧。展开宏得到：
``` c
//ERGEST_SWITCH(ENERGEST_TYPE_CPU, ENERGEST_TYPE_LPM); 宏展开
//#define ENERGEST_SWITCH(type_off, type_on) 
do { 
  rtimer_clock_t energest_local_variable_now = RTIMER_NOW();

  if(energest_current_mode[ENERGEST_TYPE_CPU] != 0) { 
    if (energest_local_variable_now < energest_current_time[ENERGEST_TYPE_CPU]) { 
      energest_total_time[ENERGEST_TYPE_CPU].current += RTIMER_ARCH_SECOND; 
    }
    energest_total_time[ENERGEST_TYPE_CPU].current += (rtimer_clock_t)(
                          energest_local_variable_now 
                        - energest_current_time[ENERGEST_TYPE_CPU]); 
    energest_current_mode[ENERGEST_TYPE_CPU] = 0; 
  } 
  energest_current_time[ENERGEST_TYPE_LPM] = energest_local_variable_now; 
  energest_current_mode[ENERGEST_TYPE_LPM] = 1; 
  } while(0)
```
该代码段处理的是energest_total_time[].current,energest_current_mode,energest_current_time之间的事情。首先将当前时间保持在*_now变量中；然后看需要关闭的*_mode[type_off]是否为0，如果不为零对其total_time处理后进行置零；最后赋值需要打开的[type_on]的*_time[type_on]和*_mode[type_on]变量。  
此处的时间处理，是将CPU启动到关闭运行的时间保存到其total_time变量中。那么这个ENERGEST_SWITCH做的事情就是切换正在执行的类型，记录被关闭的类型的允许时间，并将当前时间保持到要切换的新类型中。

ENERGEST_IRQ_*宏是在lpm.c函数中定义的，RESTORE调用的是`energest_type_set(ENERGEST_TYPE_IRQ, a);`函数，SAVE调用的是`a = energest_type_time(ENERGEST_TYPE_IRQ);`函数。总之就是进行休眠前的变量存储和唤醒后的变量还原。

## 4.2 sleep
``` c
ti_lib_prcm_sleep();
```
``` c
#define ti_lib_prcm_sleep(...)        PRCMSleep(__VA_ARGS__)
```
这里的PRCM: Power and clock management，而这条语句执行了什么操作呢？继续看
``` c
__STATIC_INLINE void
PRCMSleep(void)
{
    // Wait for an interrupt.
    CPUwfi();
}
```
``` c
__asm __STATIC_INLINE void
CPUwfi(void)
{
  wfi;
  bx      lr
}
```
wfi是Wait for interrupt的缩写，这是个ARM的汇编指令，用于让CPU进入idle状态。可参考链接：[ARM WFI和WFE指令](http://www.wowotech.net/armv8a_arch/wfe_wfi.html)了解更多内容。  
bx lr同样是汇编语句，作用类似于c函数的return。  
这下就明白了，执行lpm_sleep函数后，最终停在wfi等待中断事件的发生了。当中断发生时退出休眠模式，返回原来的函数继续执行。

# 5. deep_sleep
需要说明的是deep_sleep或shutdown中被唤醒之后是需要执行wakeup函数的。wakeup做的事情与deep_sleep和shutdown刚好相反。

### 5.1 domains
在deelp_sleep中首先执行的是各模块的shutdown函数:
``` c
  //来自lpm.c
  #define LOCKABLE_DOMAINS ((uint32_t)(PRCM_DOMAIN_SERIAL | PRCM_DOMAIN_PERIPH))
  //来自deep_sleep
  uint32_t domains = LOCKABLE_DOMAINS; //变量用于统计深度休眠下需要保持供电的单元
  for(module = list_head(modules_list); module != NULL;
      module = module->next) {
    if(module->shutdown) {
      module->shutdown(LPM_MODE_DEEP_SLEEP);
    }
    domains &= ~module->domain_lock;
  }
  domains &= LOCKABLE_DOMAINS; //保险起见，变量初始化之后再次进行排查LOCKABLE_DOMAINS，保证这里只控制SERIAL和PERIPH。
  //最后domains中
```
这里处理的只有PRCM_DOMAIN_SERIAL和PRCM_DOMAIN_PERIPH，根据实际情况选择是否深度休眠状态下保持供电。  
函数中的后面有语句：
``` c
ti_lib_prcm_power_domain_off(PRCM_DOMAIN_RFCORE | PRCM_DOMAIN_CPU |
                               PRCM_DOMAIN_VIMS | PRCM_DOMAIN_SYSBUS);
```
使其它部分掉电。                               

### 5.2 喂狗
`watchdog_periodic();`防止唤醒后事情没处理完就发生看门狗中断。

### 5.3 时钟 VIMS等其它部分关闭
`` c
  oscillators_switch_to_hf_rc();
  /* Shut Down the AUX if the user application is not using it */
  aux_ctrl_power_down(false);
  /* Configure clock sources for MCU: No clock */
  ti_lib_aon_wuc_mcu_power_down_config(AONWUC_NO_CLOCK);
  ti_lib_aon_wuc_jtag_power_off();
  ti_lib_aon_wuc_domain_power_down_enable();
  ti_lib_sys_ctrl_set_recharge_before_power_down(XOSC_IN_HIGH_POWER_MODE);
  ti_lib_prcm_cache_retention_disable();
  ti_lib_vims_mode_set(VIMS_BASE, VIMS_MODE_OFF);
```
代码整理后就是上面这些完成了相关部分的powerdown.

### 5.4 进入深度休眠
`ti_lib_prcm_deep_sleep();`函数完成。
最终函数体如下：
``` c
void
PRCMDeepSleep(void)
{
    // Enable deep-sleep.
    HWREG(NVIC_SYS_CTRL) |= NVIC_SYS_CTRL_SLEEPDEEP;

    // Wait for an interrupt.
    CPUwfi();

    // Disable deep-sleep so that a future sleep will work correctly.
    HWREG(NVIC_SYS_CTRL) &= ~(NVIC_SYS_CTRL_SLEEPDEEP);
}
```
同样是执行CPUwfi函数进入休眠状态。
# 6. 小结
与lmp_sleep不同的是，deep_sleep函数进入休眠之前把许多单元的电源时钟都关闭了，同时退出休眠的时候需要执行wakeup函数进行还原。而lmp_sleep不需要，它从休眠模式退出时直接可以执行原来正在执行的代码，并且各个单元与进入休眠时的状态一致。

