---
title: Contiki进程分析-struct process
toc: true
date: 2016-10-20 16:47:34
categories:
- Contiki
tags:
- Contiki
- 进程
---

Contiki定义了一个链表数据结构`struct process{struct process *next; ...;};` 用来管理进程，所有需要执行的进程都被注册（通过process_starta函数添加）在这个链表上。这个数据结构中包含进程的状态。  
链表上的进程根据需要以轮询的方式逐一执行。以printf("hello,world.\n")为例,函数体；
<!--more-->

# 1. helloworld进程宏展开
``` c
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
```
宏展开：
``` c
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
从宏展开后的代码可以看出，printf语句被执行是发生在`(process_pt) -> lc`被置1之后的。并且printf语句执行完后return值是PT_ENDED，进程结束。所以只会打印1次。

# 2. process数据结构
觉得还是先看下进程数据结构比较好：
``` c
struct process {
  struct process *next;
  const char *name;
  int (* thread) (struct pt *, process_event_t, process_data_t);
  struct pt pt;
  unsigned char state, needspoll;
};
```
* next 用来构成链表的将指向其他的process数据结构；
* name 字符串，用来存放进程名称的；
* thread 函数指针，指向进程的函数体；
* pt struct pt数据结构被展开是unsigned short lc; 这个lc就是上面helloworld函数体重语句switch((process_pt) -> lc)中的lc，此值为0时执行printf语句；
* state 进程的状态变量，执行process_start函数后该变量被置为PROCESS_STATE_RUNNING；
* needspoll 是否需要轮询变量，只有当该变量值为1时，才会轮询到改进程。
要知道每一个进程都会有这样的一个数据结构，其中theread是函数指针，需要指向进程的函数体，这里就是process_thread_hello_world_process函数。  

# 3. 对process数据结构进行操作的函数
## 3.1 process数据结构类型变量定义

``` c
PROCESS(hello_world_process, "Hello world process");
```
宏展开：
``` c
#define PROCESS(name, strname)				\
  PROCESS_THREAD(name, ev, data);			\
  struct process name = { NULL, strname,		\
                          process_thread_##name }
```
上面语句完成了进程函数的声明，同时也定义了process类型变量,process的内容为：
* next = NULL;
* name = "Hello world process";
* thread = process_thread_hello_world_process;//这是函数地址

## 3.2 数据结构成员变量的操作
对进程相关变量的操作是通过process.c中的一些函数和一些进程原函数实现的。
* process_start: 启动进程函数，将进程数据结构添加到process_list中，同时state变量置为PROCESS_STATE_RUNNING;再通过PT_INIT()进程原函数将pt置0（这里其实是1个unsigned short型变量）；
* process_poll: 如果state=PROCESS_STATE_RUNNING/PROCESS_STATE_CALLE就会执行:
  ``` c
  p->needspoll = 1;
  poll_requested = 1;
  ```
* process_run: 执行进程,如果poll_requested=1就执行do_poll函数，并且总会执行do_event函数。
* do_poll: 进程轮询函数，该函数会轮询process_list中的所有进程，poll_requested清零，同时轮询方式执行所有needspoll=1的进程函数,执行完后needspoll清零,state=PROCESS_STATE_RUNNING, 进程函数的执行用call_process函数进行,ev=PROCESS_EVENT_POLL；
* call_process: 进程调用函数，该函数会把当前被调用的函数设置为当前进程（process_current）,进程调用前将state=PROCESS_STATE_CALLED，然后通过ret = p->thread(&p->pt, ev, data);语句执行进程，然后根据ret返回值改变进程的state值，并根据需要执行exet_process函数，函数体主要部分如下：
  ``` c
  if((p->state & PROCESS_STATE_RUNNING) &&
     p->thread != NULL) {
    PRINTF("process: calling process '%s' with event %d\n", PROCESS_NAME_STRING(p), ev);
    process_current = p;
    p->state = PROCESS_STATE_CALLED;
    ret = p->thread(&p->pt, ev, data);
    if(ret == PT_EXITED ||
       ret == PT_ENDED ||
       ev == PROCESS_EVENT_EXIT) {
      exit_process(p, p);
    } else {
      p->state = PROCESS_STATE_RUNNING;
    }
  }
  ```
  回想process_thread_hello_world_process进程的返回值就是PT_ENDED，所以helloworld只打印了1次；
* exit_process: 进程退出，将进程数据结构从process_list中移除；
* 
