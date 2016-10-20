---
title: 从main函数分析contiki
toc: true
date: 2016-10-20 10:34:33
categories:
- contiki
- c
tags:
- contiki
---

使用平台为CC26xx，main函数基本上全部位于contiki下的platform文件夹下的各平台文件夹中，所以尝试从main函数入手分析。
main函数放在平台目录中，相同处理器不同的开发板提供的资源是不一样的，那么是需要新创建一个平台的，然后在里面进行各种定制，所以分析现有平台的设计方法是很有必要的。
main函数中主要包含：*_init(); process*(); while(1);
<!--mor-->

# init 部分
## 1. 配置CC26XX的VIMS工作模式
``` c
/* Enable flash cache and prefetch. */
  ti_lib_vims_mode_set(VIMS_BASE, VIMS_MODE_ENABLED);
  ti_lib_vims_configure(VIMS_BASE, true, true);
```
在vim.h文件中可以找到：
```
#define VIMS_MODE_ENABLED  (VIMS_CTL_MODE_CACHE) // Enabled mode, only USERCODE is cached.
```
在hw_vim.h文件中找到：
``` c
#define VIMS_CTL_MODE_CACHE                                         0x00000001
```
## 2. 关中断
```
ti_lib_int_master_disable();
```
在ti-lib.h文件中有：
#define ti_lib_int_master_disable(...)        IntMasterDisable(__VA_ARGS__)

## 3. 时钟选择
```
/* Set the LF XOSC as the LF system clock source */
  oscillators_select_lf_xosc();
```
## 4. lpm初始化
通过函数lpm_init()实现。
```
void
lpm_init()
{
  list_init(modules_list);

  /* Always wake up on any DIO edge detection */
  ti_lib_aon_event_mcu_wake_up_set(AON_EVENT_MCU_WU3, AON_EVENT_IO);
}
```
modules_list由LIST(modules_list)定义，展开后如下：
```
static void *modules_list_list = NULL;
static void ** modules_list = (**)&modules_list_list;
```
而list_init()是这样的：
```
void
list_init(list_t list)
{
  *list = NULL;
}
```
嗯，除非用过，否则这句执行不执行一个样。

## 5.  board_init
 board_init();实现，看看做了什么：
 ```
 board_init()
{
  uint8_t int_disabled = ti_lib_int_master_disable();//如果本来就disable则返回true，如果为enable，则disable并返回false。

  /* Turn on relevant PDs */
  wakeup_handler();

  /* Enable GPIO peripheral */
  ti_lib_prcm_peripheral_run_enable(PRCM_PERIPH_GPIO);//PRCM(power management and clock control)

  /* Apply settings and wait for them to take effect */
  ti_lib_prcm_load_set();
  while(!ti_lib_prcm_load_get());

  lpm_register_module(&srf_module); //将srf_module添加到lpm的模块链表中。srf_module是个lpm_registered_module_t类型的数据结构，

  configure_unused_pins();//这个本来不想看的，看到了就写到下面。

  /* Re-enable interrupt if initially enabled. */
  if(!int_disabled) {//如果原来为enable，这里进行还原。
    ti_lib_int_master_enable();
  }
}
 ```
configure_unused_pins：看来没用的全关了。为了低功耗，不过写出来是为了知道想要的时候这里需要修改宏。
```
static void
configure_unused_pins(void)
{
  /* Turn off 3.3-V domain (lcd/sdcard power, output low) */
  //#define BOARD_IOID_3V3_EN         IOID_13 //demo板上这个pin是LCD的reset。
  ti_lib_ioc_pin_type_gpio_output(BOARD_IOID_3V3_EN);
  ti_lib_gpio_clear_dio(BOARD_IOID_3V3_EN);

  /* Accelerometer (PWR output low, CSn output, high) */
  ti_lib_ioc_pin_type_gpio_output(BOARD_IOID_ACC_PWR);
  ti_lib_gpio_clear_dio(BOARD_IOID_ACC_PWR);
}
```

## 6. GPIO中断初始化
`gpio_interrupt_init();`函数完成，而这个函数实现的是对handles[]函数指针数组的初始化，全部初始化为NULL，这个需要的时候用`gpio_interrupt_register_handler`函数进行注册，然后遇到中断的时候会用`handles[]()`;来完成中断处理。

## 7. LED初始化
和GPIO一样需要设置相应端口为输出模式。其它事情不做。

## 8. 时钟初始化
时钟初始化通过三个函数完成：
```
  soc_rtc_init(); //初始化定时器，产生ticks
  clock_init(); //初始化GPT0定时器，配置用于定时的计时器
  rtimer_init();//什么也没做
```

## 9. 看门狗初始化
## 10. 产生随机数
## 11. 串口初始化
## 12. 串口输入初始化 //serial_line_init();
## 13. RF和网络协议相关配置初始化

# process 相关
在serial_line_init();函数中就使用了process_start函数。把其他相关代码整理如下：
```
  process_init(); //清空任务链表
  process_start(&serial_line_process, NULL);//serial_line_init();展开得到。

  process_start(&etimer_process, NULL);
  ctimer_init();

  process_start(&sensors_process, NULL);
  autostart_start(autostart_processes);
```
看了官方文档，跟了代码，总算搞明白了autostart_process，process_start, autostart_start, process_list之间的关系了。autostart_process是进程指针数组，包含需要直接启动的进程，process_list是进程链表所有启动的任务均在这个链表上。autostart_start最终调用了process_start，不同的是autostart_start的入口参数是个进程指针数组，可以同时启动多个进程，而process_start入口参数是进程指针，把某个进程添加到进程链表中。  
process_start函数不能使进程被执行。

## process start
* 将进程添加到链表process_list的最前端，并将该进程的地址赋值给process_list。
* 将进程状态设置为`PROCESS_STATE_RUNNING`;
* 执行`PT_INIT(&p->pt);`将进程的pt值设置为0。

# while(1)
```
  while(1) {
    uint8_t r;
    do {
      r = process_run();
      watchdog_periodic();
    } while(r > 0);

    /* Drop to some low power mode */
    lpm_drop();
  }
```
while(1)中执行1次`process_run`，然后执行`lpm_drop()`.

