---
title: Contiki helloworld例程函数体宏展开
toc: true
date: 2016-10-17 15:46:17
categories:
- Contiki
tags: 
- Contiki
- helloworld
---

看看helloworld.c文件：
```
/*---------------------------------------------------------------------------*/
PROCESS(hello_world_process, "Hello world process");
AUTOSTART_PROCESSES(&hello_world_process);
/*---------------------------------------------------------------------------*/
PROCESS_THREAD(hello_world_process, ev, data)
{
  PROCESS_BEGIN();

  printf("Hello, world\n");
  
  PROCESS_END();
}
/*---------------------------------------------------------------------------*/
```
宏展开后：
```
static char process_thread_hello_world_process(struct pt *process_pt, process_event_t ev, process_data_t data);  //process函数声明

//定义并初始化helloworld的进程结构体。
struct process hello_world_process = {NULL,process_thread_hello_world_process};

//定义自启动process，创建并添加到autostart_processes数组中。
struct process * const autostart_processes[] = { &hello_world_process, NULL};

//helloworld进程函数体定义
static char process_thread_hello_world_process(struct pt *process_pt, process_event_t ev, process_data_t data)
{
    char PT_YIELD_FLAG = 1;
    if (PT_YIELD_FLAG) {;}
    switch((process_pt) -> lc) {
        case 0:
        printf("Hello world!\n");
    };
    PT_YIELD_FLAG = 0;
    process_pt->lc = 0;
    return PT_ENDED;
}
```


